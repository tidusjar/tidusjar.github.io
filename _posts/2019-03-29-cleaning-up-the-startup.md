---
title: "Cleaning up the Startup.cs"
published: false
---

One thing I always encounter with .Net Core projects now is that the `Startup.cs` class ends up growing 'organically'. Now there is a decent amount of functionality within there from the template.

Personally I want to keep the startup of the application as light as possible, but sometimes we do need to do things at the startup e.g. initalise thirdparty middlewears for example Swagger.

I think the solution to this is extension methods, that's how the Asp.Net team are doing it, just take a look at `services.AddMvc()`, it seems that the build pattern they are using is all based around the `IServiceCollection`. 
To tidy up our `Startup.cs` why don't we take the same approach as the Asp.Net Team then!

Usually I'd create a new static class within the web project called `StartupExtensions.cs` and has it something like the following:


``` csharp
public static class StartupExtensions
{
    public static IServiceCollection AddDefaultServices(this IServiceCollection services)
    {
        services.AddCors();
        services.AddMvc();
    }

    public static IApplicationBuilder AddDefaultConfiguration(this IApplicationBuilder app)
    {
        app.UseExceptionMiddleware();
        app.UseCors("YourPolicy");
        app.UseAuthentication();
        app.UseMvc();
    }

    public static IServiceCollection AddDependancies(this IServiceCollection services)
    {
        
    }
}

```