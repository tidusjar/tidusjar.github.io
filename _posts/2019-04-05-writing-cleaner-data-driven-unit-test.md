---
title: "Writing Cleaner Data Driven Unit Tests with NUnit"
published: true
---

Something that I see all the time is people copy and pasting unit tests and just changing one or two variables to test a different scenario to get full coverage.

Now this obviously works but it's not the most efficient way of doing things, if that code under test changes slightly it might mean that you have to amend several tests instead of one.

What I do in this scenario is take advantage of the NUnit framework and it's implementation of the `[TestCaseSource()]` attribute.

For example, if we have the following method that we want to test:

``` csharp
public static class HtmlHelper
{
    public static string RemoveHtml(this string value)
    {
        if (string.IsNullOrEmpty(value))
        {
            return string.Empty;
        }
        var step1 = Regex.Replace(value, @"<[^>]+>|&nbsp;", "").Trim();
        var step2 = Regex.Replace(step1, @"\s{2,}", " ");
        return step2;
    }
}

```

From this method name and signature, it is clear that this is going to remove any HTML from a string. 
We could write 100's of different individual tests to actually test this method but this is how you would write one test method to test lots of differences.

``` csharp
[TestFixture]
public class HtmlHelperTests
{
    [TestCaseSource(nameof(HtmlData))]
    public string RemoveHtmlTests(string input)
    {
        return HtmlHelper.RemoveHtml(input);
    }
    public static IEnumerable<TestCaseData> HtmlData
    {
        get
        {
            yield return new TestCaseData("<h1>hi</h1>").Returns("hi").SetName("Simple Html");
        }
    }
}
```

As you can see we are feeding the test method with inputs, and then returning the result. Now NUnit is going to actually verify that the return value matches the `.Returns(string)` specified in the `TestCaseData`.
With this we can perform multiple tests with different data on one test method. 

Here is an example with more data

``` csharp
get
{
    yield return new TestCaseData("<h1>hi</h1>").Returns("hi").SetName("Simple Html");
    yield return new TestCaseData("<html><body><head></head><h1>hi</h1></body></html>").Returns("hi").SetName("Nested text inside Html");
    yield return new TestCaseData("there is no html here").Returns("there is no html here").SetName("No Html");
    yield return new TestCaseData("there is <b>some</b> html here").Returns("there is some html here").SetName("Html in middle");
    yield return new TestCaseData("<a>there</a> <u>is</u> <b>lots</b> <i>html</i> <span>here</span>").Returns("there is lots html here").SetName("Html in everywhere");
    yield return new TestCaseData("there is <span class=\"abc\">some</span> html here").Returns("there is some html here").SetName("Html in with classes");
    yield return new TestCaseData("there is <span id=\"sometag\">some</span> html here").Returns("there is some html here").SetName("Html in with attribute");
    yield return new TestCaseData("there is <span data-tag=\"sometag\" class=\"abc\">some</span> html here").Returns("there is some html here").SetName("Html in with attribute and class");
}
```

So now we have an extra 8 tests over one method instead of 8 different test methods!

The good thing about this is you would also be able to re-use other TestCaseData inside other test methods, for example:

``` csharp
public static IEnumerable<TestCaseData> OtherData
{
    get
    {
        foreach (var data in HtmlData)
        {
            yield return data;
        }
        yield return new TestCaseData("xyz").Returns("xyz").SetName("More Tests");
    }
}
```