---
title: "Multiple Browsers in Selenium"
published: true
---

You can easily run [Selenium](http://www.seleniumhq.org/) tests in multiple browsers without having to write duplicate tests<small>*</small>:

***The Problem***
```csharp
[TestFixture]
public class ExampleSeleniumTest
{
    [Test]
    public void TestLogin_InternetExplorer()
    {
        LoginPage.LoginAs("User").WithPassword("Pass");
        // Asserts
    }
    
    [Test]
    public void TestLogin_FireFox()
    {
        LoginPage.LoginAs("User").WithPassword("Pass");
        // Asserts
    }
}
```

That code is not DRY. 
We can make use of NUnit's `[TestFixtures]` to solve this problem.
We will have to make a generic WebDriver class, something like the below
```csharp
public static class WebDriverSetup<TWebDriver> where TWebDriver : IWebDriver, new()
{
    private static IWebDriver _webDriver;
    public static IWebDriver SetUp()
    {
      // The reason for setting up an EventFiringWebDriver is we could be
      // able to take a screenshot or log something when we hit an exception
      // e.g. If we throw an AssertException then we can take a screenshot
      // and save it to a location.
      var firingDriver = new
          EventFiringWebDriver(Activator.CreateInstance(typeof(TWebDriver),
          new object[] { Constants.DriverLocation }) as IWebDriver);
          _webDriver = firingDriver;
          _webDriver = NavigateToWebsite(_webDriver);

      return _webDriver;
     }
     private static IWebDriver NavigateToWebsite(IWebDriver webDriver)
     {
         webDriver.Navigate().GoToUrl(new Uri("https://example.com/"));
         return webDriver;
        }
    }
```

So now all we need to do on our Test class is to inherit the `WebDriverSetup` class:
```csharp
[TestFixture(typeof(ChromeDriver))]
[TestFixture(typeof(InternetExplorerDriver))]
[TestFixture(typeof(FirefoxDriver))]
public class ExampleSeleniumTest<TWebDriver> where TWebDriver : IWebDriver, new()
{
    [SetUp]
    public void Init()
    {
       Driver.SetUp<TWebDriver>();
    }
    [Test]
    public void TestLogin()
    {
        LoginPage.LoginAs("User").WithPassword("Pass");
        // Asserts
    }
}
```

This will now run each `[TestFixture]` and will run the tests for FireFox, Chrome and Internet Explorer!
<small>
**note I am using [NUnit](http://www.nunit.org/)*
</small>