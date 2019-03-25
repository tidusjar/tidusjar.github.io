---
title: ".Net Core JWT Authentication with an API Key middleware"
published: false
---

My question on [Stackoverflow](https://stackoverflow.com/questions/45798325/net-core-web-api-key) and subsequent answer inspired me to write this post as there seems to be quite a lot of frequent visits to that post.

The requirement was to have a .Net Core application which its primary authentication method is using JWT plugged into the Asp.Net Core Identity membership system.

Once identity has been plugged into your application you should now be able to use the ```[Authorization]`` attributes in your controllers and verify that the user is now logged into the system. But now, if you have some public API's that you want to secure, but not behind a user with a JWT token how do you achieve this?

Well the answer is a custom middleware in the asp.net pipeline.

The code for this is below
``` csharp
public class ApiKeyMiddlewear
{
    public ApiKeyMiddlewear(RequestDelegate next)
    {
        _next = next;
    }
    private readonly RequestDelegate _next;
    public async Task Invoke(HttpContext context)
    {
        if (context.Request.Path.StartsWithSegments(new PathString("/api")))
        {
            //Let's check if this is an API Call
            if (context.Request.Headers.Keys.Contains("ApiKey", StringComparer.InvariantCultureIgnoreCase))
            {
                // validate the supplied API key
                // Validate it
                var headerKey = context.Request.Headers["ApiKey"].FirstOrDefault();
                await ValidateApiKey(context, _next, headerKey);
            }
            else
            {
                await _next.Invoke(context);
            }
        }
        else
        {
            await _next.Invoke(context);
        }
    }
    private async Task ValidateApiKey(HttpContext context, RequestDelegate next, string key)
    {
        // validate it here
        var valid = false;
        if (!valid)
        {
            context.Response.StatusCode = (int)HttpStatusCode.Unauthorized;
            await context.Response.WriteAsync("Invalid API Key");
        }
        else
        {
            var identity = new GenericIdentity("API");
            var principal = new GenericPrincipal(identity, new[] { "Admin", "ApiUser" });
            context.User = principal;
            await next();
        }
    }
}
```

What we are doing here is first, checking if the requested path contains `api`, you might need to change this if you are using some other paths for your Api endpoints, in my situation it was `~/api/v{version}/{controller}`. 
I'd personally expect a Api Key to be passed as a header, but you can easily check any query params if needed, and then you will need to validate the api, by looking it up in the database for example.
Luckily we have a `HttpContext` here so you have access to `IServiceProver` here by `context.RequestServices`, but you do need to be careful as this might impact the ability to unit test this middleware.

The last part of the puzzle is to have a 'pretend' user for this api call to impersonate

``` csharp
var identity = new GenericIdentity("API");
var principal = new GenericPrincipal(identity, new[] { "Admin", "ApiUser" });
context.User = principal;
```

As you can see above I am creating a new identity with the roles of "Admin" and "ApiUser", now you will need to change this to match your needs. But this part is the key, you will need to do this to allow you to pass through the `[Authorization]` attributes and you will then have access to the `User.Identity` properties on the `BaseController`.

