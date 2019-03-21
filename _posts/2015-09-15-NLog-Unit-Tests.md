---
title: "Using NLog in your Unit Test Framework"
published: true
---

So I was writing some automation tests and I wanted to have some in-depth logging to know what is going on when the tests fail for better diagnostics (same as you would in any normal application).

So it occurred to me, why re-invent the wheel creating a logging framework when there are some really good ones out there? I chose [NLog](http://nlog-project.org/), I had a small amount of experience with NLog before but not that much (I primarily used [Elmah](https://code.google.com/p/elmah/))

I did a bit of research and found that people wanted logging in their Unit Tests but there were not that many solutions so I just went with it and implemented NLog into my Unit Test Project. Here is what I did:

Installed NLog into my Unit Test Project via [Nuget](https://www.nuget.org/) `Install-Package NLog` Great!

So since a unit test project doesn't really need any configuration files I needed to create an `App.config` for NLog to read it's configuration.
I added the following configuration elements between `<configuration>`:
```xml
<configSections>
    <section name="nlog" type="NLog.Config.ConfigSectionHandler, NLog" />
</configSections>

<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <targets>
        <target name="file"
                xsi:type="File"
                layout="${date} ${logger} ${level}: ${message}"
                fileName="C:\Logs\${shortdate}.log" />
        </targets>
    <rules>
        <logger name="*" minlevel="Trace" writeTo="file"/>
    </rules>
</nlog>
```
So since we need this `app.config` with the actual unit tests we need to make sure in out `TestSettings` we add this as a deployment item. (If you do not have a `TestSettings` file under your solution you will need to create it: Right click your solution in Solution Explorer, Add > New Item, Under Test Settings menu there is the `TestSettings` file.) So open up the `TestSettings` file and navigate to the `Deployment` menu and ensure that the 'Enable Deployment' checkbox is checked. Then Press 'Add File' and navigate to your `app.config` and then press Apply!

Now when we run the unit tests the `app.config` will be deployed with the relevant `.dll`. Now you can just use NLog like you would usually!
```csharp
var logger = LogManager.GetCurrentClassLogger();
logger.Debug("This is a debug message!");
```
---
You could just finish there and be done with it, but usually any logging implementations I write I like to implement an interface so I can easily swap out the Logging framework for another and also Mock out the Logger (not this [type](http://www.squareeyed.tv/wp-content/uploads/2015/05/MOCK-THE-WEEK-panellists-010.jpg)).

**Interface**

```csharp
public interface ILogger
{
    void Trace(string message);
    void Info(string message);
    void Warn(string message);
    void Debug(string message);
    void Error(string message);
    void Error(string message, Exception x);
    void Error(Exception x);
    void Fatal(string message);
    void Fatal(Exception x);
}
```

**Implementation**
```csharp
public class NLogLogger : ILogger
    {
        private readonly Logger _logger;
        public NLogLogger(Type classType)
        {
            _logger = LogManager.GetLogger(classType.Name);
        }
        public void Trace(string message)
        {
            _logger.Trace(message);
        }
        public void Info(string message)
        {
            _logger.Info(message);
        }
        public void Warn(string message)
        {
            _logger.Warn(message);
        }
        public void Debug(string message)
        {
            _logger.Debug(message);
        }
        public void Error(string message)
        {
            _logger.Error(message);
        }
        public void Error(Exception x)
        {
            _logger.Error(x);
        }
        public void Error(string message, Exception x)
        {
            _logger.Error(x, message);
        }
        public void Fatal(string message)
        {
            _logger.Fatal(message);
        }
        public void Fatal(Exception x)
        {
            _logger.Fatal(x);
        }
    }
```

So since we are using a wrapper with the NLog implementation we are going to need to do something slightly different as you can see in the `NLogLogger` constructor. We cannot call `LogManager.GetCurrentClassLogger()` since it will always be `NLogLogger` so we need to pass it the class type.
```csharp
 public class LoggerTestClass
 {
     public LoggerTestClass()
     {
         Logger = new NLogLogger(typeof(LoggerTestClass));
     }
     private ILogger Logger { get; set; }
     [TestMethod]
     public void LogUnitTest()
     {
         Logger.Fatal("Fatal Exception!");
     }
   }
```

Then you are all set! You will get some nice formatted logs from your Unit tests.