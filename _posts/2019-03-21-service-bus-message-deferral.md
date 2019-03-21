---
title: "Azure Service Bus Message Deferral Implementation"
published: true
---
I was recently tasked to create an API that needs to be resilient and decoupled from other systems. So, we used Azure's Service Bus for this. This API is used in a way where it receives instructions in an order, and that order matters. This part is key.

What we want is:

`Entry API > Service Bus > Azure Function > Save API`

For example if the first message fails, we can't process the second message, because again they need to be processed in an order. The solution to this was to implement message deferral! You can read Microsoft's article on this [here](https://docs.microsoft.com/en-us/azure/service-bus-messaging/message-deferral). This is a summary taken from that article

> When a queue or subscription client receives a message that it is willing to process, but for which processing is not currently possible due to special circumstances inside of the application, it has the option of "deferring" retrieval of the message to a later point. The message remains in the queue or subscription, but it is set aside.

> Deferral is a feature specifically created for workflow processing scenarios. Workflow frameworks may require certain operations to be processed in an order and may have to postpone processing of some received messages until prescribed prior work that is informed by other messages has been completed.

Perfect! This sounds exactly like what I want. 

I take the message in my Azure Function, process it, if it fails, defer it until later!
So as easy as it sounds, looking into the documentation it's not as straight forward as that. 
To defer a message, you need to call `.Defer()` on the `BrokeredMessage` object. Since the `.Defer()` method does not take any parameters as part of its signature means that you can’t do something like `.Defer(DateTime.UtcNow.AddMinutes(-5);` because that would be too easy right? 

My solution to this was to defer the message and place a new message onto the Service Bus with the sequence number of that `BrokeredMessage` so we can take the message out of the deferred state. 
If you look at the [docs](https://docs.microsoft.com/en-us/azure/service-bus-messaging/message-deferral) the way you retrieve a deferred message is to call the `Receive(sequenceNumber)` method on your `SubscriptionClient` (if you are using Service Bus with a Topic) or `QueueClient`. 
This took me a while to work out as the docs at the time didn't have any information on where the `Receive(sequenceNumber)` method was until [recently](https://github.com/MicrosoftDocs/azure-docs/pull/27040#event-2214873391).

`.Defer()` the message and place another message onto the Service Bus with the sequence number.

Here is an example of deferring and placing another message.

``` csharp
private static async Task<bool> DeferMessage(BrokeredMessage message)
{
    await message.DeferAsync().ConfigureAwait(false);

    var client = TopicClient.CreateFromConnectionString(ConnectionString, TopicName);
    var obj = new DeferredMessageModel { SequenceNumber = message.SequenceNumber };

    using (var ms = new MemoryStream())
    using (var writer = new StreamWriter(ms))
    using (var jsonWriter = new JsonTextWriter(writer))
    {                      
        var ser = new JsonSerializer();
        ser.Serialize(jsonWriter, obj);
        jsonWriter.Flush();

        ms.Position = 0;

        await client.SendAsync(new BrokeredMessage(ms)
        {
            Properties = { new KeyValuePair<string, object>("deferred", true) },
            ScheduledEnqueueTimeUtc = DateTime.UtcNow.AddMinutes(-5)
        }).ConfigureAwait(false);
    }

    return true;
}
```


The `DeferredMessageModel` class just has one property of `SequenceNumber` that is 	a `long`.

As you can see the above implementation is pretty simple. Now the key part here is that we are adding a property to the new message stating the it's be deferred.

When we process the message we can check if this is a message to retrieve a deferral or a regular message.
``` csharp
private static bool IsDeferredMessage(BrokeredMessage message)
{
    if (message.Properties.TryGetValue("deferred", out var value))
    {
        if (value is bool deferredResult)
        {
            return deferredResult;
        }
    }
    return false;
}
```
    
We are just going to check to see if the message has the `deferred` property set, if it has this is a message to retrieve a deferred message.
To retrieve a deferred message we need to do something like the following:

    if (IsDeferredMessage(message))
    {
        var deferralObject = JsonConvert.DeserializeObject<DeferredMessageModel>(json);
        await message.CompleteAsync(); // Complete this message
        message = await GetDeferredMessage(storedMessage.SequenceNumber); // get the deferred message
    }
    private static async Task<BrokeredMessage> GetDeferredMessage(long sequenceNumber)
    {
        var client = 
            SubscriptionClient.CreateFromConnectionString(Environment.GetEnvironmentVariable("AzureServiceBusPrimary"),
            TopicName, SubscriptionName);
        return await client.ReceiveAsync(sequenceNumber).ConfigureAwait(false);
    }
    
As you can see we need to `Complete` the current message otherwise we will end up getting the dreaded Message Locked Exception and it will be placed back onto the queue to be re-processed (depending on your service bus topic strategy).

Now you have your deferred message that has been delayed for 5 minutes!

There is one more thing, because calling `.Defer()` on a message removes it from the queue you are now managing the lifecycle of the messages. So in your `host.json` configuration you need to tell the Azure Function that you are going to continue managing the lifecycle of the messages, and not to complete the messages automatically when the function completes.  

    {
      "serviceBus": {
        "autoComplete": false
      }
    }
    
This also means when you have finished processing your message you are going to need to call the `Complete()` method on the `BrokeredMessage` object.
