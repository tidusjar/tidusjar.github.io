---
title: "Using SignalR in an real Angular Application"
published: true
---

I wrote this after finding a bunch of articles on how to integrate Angular with SignalR, but it didn't really explain how really it should be used in a real-world scenario.

I am using a .Net Core backend in this example so certain parts may not be applicable depending on your requirements, but you should get the picture.
In this example we are going to have a generic way of sending a user notification down to the UI that is triggered by a background job.

###### Backend

Let's start at the backend, so we are going to need to import the Nuget package [`Microsoft.AspNetCore.SignalR`](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR/) into your main project.

Next is to register SignalR as part of your application, so in your `Startup.cs` we are going to want the following:

``` csharp
public IServiceProvider ConfigureServices(IServiceCollection services)
{
    // Your other stuff
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
    services.AddSignalR();
}

public void Configure(IApplicationBuilder app)
{
    // Other stuff
    app.UseSignalR(routes =>
    {
        routes.MapHub<NotificationHub>("/hubs/notification");
    });
    app.UseMvcWithDefaultRoute();
}
```

We have setup a SignalR endpoint to `/hubs/notification/` and registered the `NotficationHub` which does not yet exist, so let's create that class.

``` csharp
public class NotificationHub : Hub
{
    public static ConcurrentDictionary<string, string> UsersOnline = new ConcurrentDictionary<string, string>();

    public const string NotificationEvent = "Notification";

    public Task Notification(string data)
    {
        return Clients.All.SendAsync(NotificationEvent, data);
    }
}
```

This is a basic sample of a hub, and does what it needs to do, it has a single method that will notify ALL the clients of a message.
If you want to only notify certain clients, then you will need to override the `OnConnectedAsync` method and store the `ConnectionId` in some sort of `ConcurrentDictionary`. There are plenty of examples on the internet for this.

This example was a background job that is triggering the notification message, I am not going to show you how to implement a background job as that is not in scope for this blog, but I'll create a basic controller to show you the basics.

``` csharp
public class HubController : ControllerBase
{
    public HubController(IHubContext<NotificationHub> hub)
    {
        _hub = hub;
    }
    private readonly IHubContext<NotificationHub> _hub;

    [HttpGet("{message}")]
    public async Task Notify(string message)
    {
        await _hub.Clients.All.SendAsync("Notification", searchTerm);
    }
}
```

As you can see you are able to use the inbuilt DI to inject an instance of the `HubContext` and then use that.

I personally ran into some complications with being able to get the Users Identity as part of the Hub. I am using JWT token for authentication and the way SignalR passes it's JWT token is through a query parameter and not a header.
So as part of my setup code for the JWT authentication you need to set the `OnMessageRecieved` event to something like the following:

``` csharp
services.AddJwtBearer(x =>
{
    x.Audience = "Audience";
    x.TokenValidationParameters = tokenValidationParameters;
    x.Events = new JwtBearerEvents
    {
        OnMessageReceived = context =>
        {
            var accessToken = context.Request.Query["access_token"
            // If the request is for our hub...
            var path = context.HttpContext.Request.Path;
            if (!string.IsNullOrEmpty(accessToken) &&
                (path.StartsWithSegments("/hubs")))
            {
                // Read the token out of the query string
                context.Token = accessToken;
            }
            return Task.CompletedTask;
        }
    };
});
```

Now that's pretty much the backend code needed to initially setup SignalR.

###### Frontend

First we need the [`@aspnet/signalr`](https://www.npmjs.com/package/@aspnet/signalr) package installed via NPM (or Yarn 😊).
Now this next bit is a personal preference, but this is how I use SignalR in my frontend, I need to create a new service and we are going to call it the `SignalRNotificationService`

``` typescript
import { Injectable, EventEmitter } from '@angular/core';
import { AuthService } from '../auth/auth.service';

import { HubConnection } from '@aspnet/signalr';
import * as signalR from '@aspnet/signalr';

@Injectable()
export class SignalRNotificationService {

    private hubConnection: HubConnection | undefined;
    public Notification: EventEmitter<string> = new EventEmitter<string>();

    constructor(private authService: AuthService) {
    }

    public initialize(): void {
        this.stopConnection();

        this.hubConnection = new signalR.HubConnectionBuilder().withUrl("/hubs/notification", {
            accessTokenFactory: () => {
                return this.authService.getToken();
            }
        }).configureLogging(signalR.LogLevel.Information).build();

        this.hubConnection.on("Notification", (data: any) => {
            this.Notification.emit(data);
        });

        this.hubConnection.start().then((data: any) => {
            console.log('Now connected');
        }).catch((error: any) => {
            console.log('Could not connect ' + error);
            setTimeout(() => this.initialize(), 3000);
        });
    }

    stopConnection() {
        if (this.hubConnection) {
            this.hubConnection.stop();
            this.hubConnection = null;
        }
    };
}
```

Now the key parts here is that we are using the `accessTokenFactory` and then passing it our JWT token so SignalR can put it as part of the query string.
We are subscribing on the `Notification` event and then outputting an event with the result, this is what we are going to bind to.

Here is an example of us using the above class

``` typescript
@Component({
    selector: "my-app",
    templateUrl: "./app.component.html",
    styleUrls: ["./app.component.scss"],
})
export class AppComponent implements OnInit {
    constructor(private signalRNotification: SignalRNotificationService) { }

    public ngOnInit() {
        this.signalrNotification.initialize();
        this.signalrNotification.Notification.subscribe(data => {
            // do some sort of notification!
            console.log(data); // <-- This will be the message that has come from your NotificationHub!
        });
    }
}
```

To test this all you now need to do is call the `HubController` API and the message that is passed into the route will be logged into the browsers console!

To conclude, I'd personally create an Angular Service for a Hub and then subscribe to the event that is being emitted, ensuring a constant stream of data coming through.
