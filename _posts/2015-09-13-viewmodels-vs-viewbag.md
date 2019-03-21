---
title: "MVC: View Models or ViewData?"
published: true
---

I have seen a lot of questions on StackOverflow recently regarding the View Model vs ViewData/ViewBag argument.

In my own opinion using a `ViewModel` is the way forward. The `ViewModel` is the class that defines your requirements for the view.

My reason behind this is that in the long run, it will be a lot more maintainable. When using `ViewBag` it's a dynamic class so in your views you should be checking if the `ViewBag` property exists (And can lead to silly mistakes like typos) e.g.:
```csharp
if(ViewBag.PropertyName != null)
{
// ViewBag.PropertyName is a different property to ViewBag.propertyName
}   
```

    
    
This type of code can make your Views quite messy.
What I mean by this is in your Controller if you want to send an `IEnumerable<T>` to the view you would have to do this:
```csharp
ViewBag.Collection = new IEnumerable<string>(); // For example
```

Then in your view you would need to manually cast the `ViewBag` item to `IEnumerable<string>` so you can actually use it for its intended purpose.

```csharp
@{
var collection = (IEnumerable<string>)ViewBag.Collection;
}
```
Again, using `ViewModels` your view already knows what type your properties are, it would be this simple:
```csharp
@model MyViewModel
@Model.Collection // This property is declared as an IEnumerable<string> in MyViewModel
```


If you use a strongly typed model, you should be able to put most of the logic in your controllers and keep the View as clean as possible which is a massive plus in my books.

You also will also end up (if you use `ViewBag`) attempting to maintain it at some point and struggle. You are removing one great thing about C#, it's a strongly typed language! `ViewBag` is not strongly typed, you may think you are passing in a `List<T>` but you could just be passing a string.

One last point, you also will lose out on any intellisense features in Visual Studio.

>Some people say that it’s a lot of effort or very makes their solution 'messy' having `View Models` per view.

Wouldn't it just be as messy in your `controllers` assigning everything to a `ViewBag`? If it was a `ViewModel` you could send it off to a 'Mapping' class to map your DTO to your View.
That is another benefit  benefit of using a `ViewModel` would be that you can just send the model off to a mapping class for easier interaction with any DTO's you have, this makes life a lot easier when you have different layers e.g. DAL, Business Layer, Presentation Layer. You can easily map any Business DTO's to your View Model. There are some good frameworks that do this like [AutoMapper](http://automapper.org/) or [ValueInjector](https://github.com/omuleanu/ValueInjecter) (thanks to Val Antonini in the comments)
So you could end up with something like this:
```csharp
var mapper = new Map();
var viewModel = mapper.Map(businessDto);

return View(viewModel);
```
So to conclude, you won’t catch me at any point using `ViewBag/ViewData` in any of my code.
