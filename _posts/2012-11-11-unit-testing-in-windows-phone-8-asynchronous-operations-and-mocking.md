---
id: 280
title: 'Unit testing in Windows Phone 8 &#8211; Asynchronous operations and mocking'
date: 2012-11-11T10:00:48+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=280
permalink: /unit-testing-in-windows-phone-8-asynchronous-operations-and-mocking/
categories:
  - Windows Phone
tags:
  - Unit testing
  - Windows Phone
---
One of the most common scenarios to test are asynchronous operations: WinRT is heavily based on async methods, both in Windows 8 and in Windows Phone 8. Plus, thanks to the new async and await keywords, it’s now easier for developers to manage correctly this pattern.

Which is the correct way to test async methods? If you try to write a test in the same way we did in the previous post, you’ll see that it doesn’t work: the test will always pass, regardless of the result. The reason is that the unit test runner doesn’t know that you are testing an asynchronous operation, so it doesn’t wait for it to be finished before evaluating the assert.

In this post we’ll see which is the correct way to manage an asynchronous test. But, first, we’ll have to create some code to use in our tests. Let’s start!

### The asynchronous operation

To do our test we’re going to use the **WebClient** class, which is the simplest class available in Windows Phone to perform network operations, like downloading or uploading a file. Unlike in Windows 8 (that doesn’t have the **WebClient** class, but a similar one called **HttpClient** ), the **WebClient** only supports asynchronous operations in the old fashion way: a method to launch the operation and an event that is raised when the operation is finished and we are ready to parse the result. This approach is still supported by the Unit Test Framework but we prefer to use and test the new one, based on the async and await keywords.

To accomplish this task, we’re going to install the **Microsoft.Bcl.Async** library from NuGet, that adds async methods to standard Windows Phone classes. To see this package you have to choose the option **Include prerelease** in the dropdown placed at the top of the windows. The package, in fact, is still in beta and you won’t be able to see it with the default configuration. This way, when we use the **WebClient**, we have some new methods that contain the keyword **Task** that return a **Task,** so that they can be used with the **await** keyword. In our example we’re going to use the **DownloadStringTaskAsync** method, which returns a string with the content of the response. We&#8217;re going to dig deeper about this library in another post.

Here is a simple � test that executes this method and verifies that the returned content’s length is equal to 0.

<pre class="brush: csharp;">[TestMethod]
public async void AsyncMethod()
{
    WebClient client = new WebClient();
    string s = await client.DownloadStringTaskAsync("http://www.qmatteoq.com");
    Assert.IsTrue(s.Length == 0);
}</pre>

I’ve intentionally created this test to fail: in fact, since the **DownloadStringTaskAsync** method returns the HTML of my blog’s home page, the result’s length will always be greater than 0.

Run the application and.. the test will pass! If you have the debugger attached to the process, you’ll notice that first the test will pass and then, later, you’ll get an **AssertFailedException** on the **Assert.IsTrue** method. This happens because the unit test runner application doesn’t wait that the **DownloadStringTaskAsync** operation is completed: since the method ends without exceptions, the unit test runner marks it as passed. When the operation is really completed, the assert is executed and it fails: but it’s too late, the test has already been marked as valid.

We need to make three changes to allow the unit test runner to properly run our asynchronous test. The first one is to change our test class, so that it inherits from **WorkItemTest** class, that is part of the **Microsoft.Phone.Testing** namespace. The second step is to mark the test as asynchronous, by using the **[Asynchronous]** attribute. The third and last one is to tell to the unit test runner when the test is really finished: to do this we call the **EnqueueTestComplete()** method (which is inherited from the **WorkItemTest** class) when the async operation is ended. Here is how will look our final test method:

<pre class="brush: csharp;">[TestClass]
public class SimpleUnitTest: WorkItemTest
{
    [TestMethod]
    [Asynchronous]
    public async void AsyncMethod()
    {
        WebClient client = new WebClient();
        string s = await client.DownloadStringTaskAsync("http://www.qmatteoq.com");
        Assert.IsTrue(s.Length == 0);
        EnqueueTestComplete();
    }
}</pre>

Launch again the application: this time you’ll see that the test correctly fails, because the unit test runner will wait until the **EnqueueTestComplete** method is called before considering the test method finished.

The couple **[Asynchronous]** attribute and **EnqueueTestComplete** method can be used with any asynchronous pattern, not just the new one. Here is, for example, how can we use the same approach to test the callback based method of the **WebClient** class.

<pre class="brush: csharp;">[TestMethod]
[Asynchronous]
public void AsyncMethod()
{
    WebClient client = new WebClient();
    client.DownloadStringCompleted += (obj, args) =&gt;
                                          {
                                              Assert.IsTrue(args.Result.Length == 0);
                                              EnqueueTestComplete();
                                          };
    client.DownloadStringAsync(new Uri("http://www.qmatteoq.com"));
}</pre>

The result will be exactly the same: the test will correctly fail, since the length of the downloaded content is greater than 0.

**UPDATE:** As Anders. a blog&#8217;s reader, correctly pointed out, the test we have just written it&#8217;s not a unit test, but an integration test. The reason is that, to execute on the test, we rely on external dependencies that are available just in a� real enviroment. In this example, we&#8217;re talking about the network connection; another example of integration test would involve, for example, writing a file in the local storage or accessing to the GPS to get the current position. In the next chapter we&#8217;ll see how, with the help of mocking, we can turn an integration test into a unit test.

### Mocking: what is it?

Sometimes you need to test logic that depends on other classes and other methods, that perform various operations and that can affect the final result of test. But, what you need to test, in the end, is that the logic of your method is correct, without bothering about all the dependencies and external factors. Let’s see an example to better explain the concept.

Let’s say you have a class (with the corresponding interface) that simply exposes a method that uses the WebClient class to retrieve the content of a page and returns it:

<pre class="brush: csharp;">public interface INetworkService
{
    Task&lt;string&gt; GetContent(string url);
}

public class NetworkService: INetworkService
{
    public async Task&lt;string&gt; GetContent(string url)
    {
        WebClient client = new WebClient();
        string content = await client.DownloadStringTaskAsync(url);
        return content;
    } 
}</pre>

Your real application has another class that uses the **NetworkService** to get the length of a page’s content:

<pre class="brush: csharp;">public class DataProvider
{
    private readonly INetworkService networkService;

    public DataProvider(INetworkService networkService)
    {
        this.networkService = networkService;
    }

    public async Task&lt;int&gt; GetLength(string url)
    {
        string content = await networkService.GetContent(url);
        return content.Length;
    }
}</pre>

This means, that, for example, in the **MainPage** of your application you have some code that uses the **DataProvider** to issue a network request and get the content’s length, like in the following example:

<pre class="brush: csharp;">private async void Button_Click_1(object sender, RoutedEventArgs e)
{
    INetworkService networkService = new NetworkService();
    DataProvider provider = new DataProvider(networkService);

    int length = await provider.GetLength("http://www.qmatteoq.com");

    MessageBox.Show(length.ToString());
}</pre>

In a real application you would have used the dependency injection approach, but since we’re working on an example I won’t make the scenario too complex. Our goal is to test the **GetLength** method of the **DataProvider** class: this method relies on another class, **NetworkService**, to perform the network request. We could simply write a test that performs a real web request and check if the length of the response is greater than zero, but in this case the test would be out of scope: the fact that the network request can fail (for example, because the Internet connection is missing) doesn’t mean that the **GetLength** method is wrong. Here comes the concept of **mock:** mocking means creating a fake class, that exposes the same properties and methods of the original class, but that we can instruct to specify what a method should do and should return without effetely executing it.

In our example, our goal is to create a mock of the **NetworkService** class, so that we can simulate a real execution of the **GetContent** method and return a fake content, that will be processed by the **GetLength** method of the **DataProvider** class. For this purpose we’re going to use a library called **Moq**, that is available on NuGet but that, for the moment, we’re going to download it from <a href="http://code.google.com/p/moq/" target="_blank">the official website</a>. The reason is that Moq doesn’t officially support Windows Phone, so NuGet would abort the installation process because it doesn’t find a supported platform. Instead, if you download the library from the website, you’ll find a Silverlight version that works also on Windows Phone: simply extract the file downloaded from the website, extract the file called **Moq.Silverlight.dll** in a folder and add it to your project using the **Add reference** option. You’ll get a warning that the Silverlight library can be not fully compatible with the project: simply click **Yes** and the library will be added.

Let’s take a look at the code of the unit test method that checks if the **GetLength** method of the **DataProvider** class works fine:

<pre class="brush: csharp;">[TestMethod]
public async void ComplexTest()
{
    TaskCompletionSource&lt;string&gt; tcs = new TaskCompletionSource&lt;string&gt;();
    tcs.SetResult("done");

    Mock&lt;INetworkService&gt; mock = new Mock&lt;INetworkService&gt;();
    mock.Setup(x =&gt; x.GetContent(It.IsAny&lt;string&gt;())).Returns(tcs.Task);

    DataProvider provider = new DataProvider(mock.Object);
    int length = await provider.GetLength("http://www.qmatteoq.com");

    Assert.IsTrue(length &gt; 0);
}</pre>

Using **Moq** is very simple: first we create a mock version of the class which behavior we need to fake, that is the **NetworkService** class. Then, using the **Setup** method, we tell to the library which is the method we want to simulate and, using the **Returns** statement, which is the result that the method should return. The parameter of the **Setup** method is passed using a lambda expression: in our example, we set up a fake call of the **GetContent** method. Notice the use of the method **It.IsAny<T>**, that is used to generate a fake parameter of the request. **T** is the parameter’s type: in our example this type is equal to string, since the parameter is the URL of the website.

To manage the **Returns** statement we need a little workaround, since **GetContent** is an asynchronous method. To avoid that the **Assert** is evaluated before the method is actually finished, we need to prepare first the result of the **GetContent** method using the **TaskCompletionSource** class. In this example, we want that the **GetContent** method returns the string **“done”**, so we create a **TaskCompletionSource<string>** object and we pass the string to the **SetResult** method.

Now we are ready to simulate the **GetContent** method: we pass to the **Returns** statement the **Task** object that is returned by the **TaskCompletionSource** object.

In the end we can use the **DataProvider** class like we would do in the real application: the only difference is that, instead of passing to the constructor a real **NetworkService** instance, we pass the mocked object, that can be accessed using the **Object** property of the **Mock** object.

If you execute the test you’ll see that it will successfully pass. If you place a breakpoint and you debug the test step by step you’ll see that our mocked object has worked correctly: in our case, the length of the content will be 4, since the string returned by the **GetContent** method will be **“done”**, like we specified with the **Returns** statement.

Just a note! You can notice that, this time, we didn’t mark the test with the **Asynchronous** attribute and we didn’t call the **EnqueueTestComplete** method: the reason is that, since we have created a mock of the **GetContent** operation, the method is not asynchronous anymore, since the result is immediately returned.

### In the end

I hope you had fun playing with unit tests in Windows Phone 8!