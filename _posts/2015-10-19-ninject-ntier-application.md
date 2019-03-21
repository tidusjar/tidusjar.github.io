---
title: "Using Ninject in a N-Tier MVC Application"
published: true
---
Let's say the application structure looks like this (to keep things simple):

* MyApplication.UI (MVC)
* MyApplication.BLL (Class Library)
* MyApplication.DLL (Class Library)

Ideally, we want to have our IoC (we are using Ninject in this example) container at the composition root of our application (entry point), this is the UI project.
Time to install Ninject into our MVC project:

    Install-Package Ninject.MVC5
    
 Now we should have a new file under `App_Data` called `NinjectWebCommon.cs`.
 If you take a look at this file it gives us the basic setup to start binding our Interfaces to concrete implementations, but since we are going to want to Inject Interfaces and Implementations that the UI layer cannot see (Separation of concerns) then we should really encapsulate this functionality away.
 
 Let's create a new project (Class Library) called `MyApplication.DependencyResolver` and also a project called `MyApplication.DependencyResolver.Modules`. Once we have both projects add a new reference from the `UI` project to the `DependancyResolver` project.
 And the `DependencyResolver` project should only reference the `DependancyResolver.Modules` project.
 
 <small>Note: the two new projects will need to add the regular Ninject references `Install-Package Ninject`</small>
 
 Now we have a nice structure in place I have added the following
 
###### DependancyResolver
CustomDependencyResolver.cs
```csharp
public class CustomDependencyResolver : IDependencyResolver
{
    public INinjectModule[] GetModules()
    {
        var modules = new INinjectModule[]
        {
            new ServiceModule(),
        };
        return modules;
    }
 }
```

IDependencyResolver.cs <small>(Allows us to be able to provide a different implementation or mock it out for unit testing).</small>
```csharp
public interface IDependencyResolver
{
    INinjectModule[] GetModules();
}
```
###### DependancyResolver.Modules
ServiceModule.cs
```csharp
public class ServiceModule : NinjectModule
{
    public override void Load()
    {
        Bind<IService>().To<ApplicationService>();
    }
}
```

That's basically all the code we require, I personally prefer to split out all my dependencies into different modules, so I would also have a `LoggingModule` for my logging framework, maybe a `RepositoryModule` for any repository’s I have etc.

Now going back to the `NinjectWebCommon` file in the composition root of our application, there is a method called `CreateKernel()`, inside this method is where we are going to want to get all of our Modules.

So I have modified the code to be like this:
```csharp
 private static IKernel CreateKernel()
 {
     var dependencyResolver = new CustomDependencyResolver();
     var modules = dependencyResolver.GetModules();

     var kernel = new StandardKernel(modules);
     ...Rest of the method that we are not interested about.
}
```

That is basically it, we now have passed all of our modules into the Kernel. 
This also means that all the references that you do not want in the UI project are over in the `MyApplication.DependencyResolver.Modules` project and the only reference you have to add to the UI is the `MyApplication.DependencyResolver` assembly.
