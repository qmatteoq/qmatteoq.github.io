---
id: 276
title: Unit testing in Windows Phone 8 – The basics
date: 2012-11-09T10:00:31+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=276
permalink: /unit-testing-in-windows-phone-8-the-basics/
categories:
  - Windows Phone
tags:
  - Unit testing
  - Windows Phone
---
Unit testing in Windows Phone has never been really supported by Microsoft: the only semi-official way to do unit tests in Windows Phone 7 is using an adaptation of the Silverlight Unit Test Framework made by Jeff Wilcox. This library is able to render a Windows Phone application, that is capable of running the unit tests included in the project. The downside of this approach is that you have to use a dedicated Windows Phone app to run the tests, instead of using a built-in tool like Resharper. The biggest advantage, instead, is that unit tests are executed into a real environment, so it’s easy to write integration tests that makes use, for example, of the storage or of a real Internet connection.

Since unit tests are an important part of every project, especially the most complex ones, Microsoft has now adapted the Silverlight Unit Test Framework to Windows Phone 8, dropping the Silverlight part of the name and adding it as a part of the new Windows Phone 8 toolkit. Let’s see how to use it, how to run simple tests and, in the next post, how to face some more complicated scenarios (like mocking and dealing with async methods).

### Preparing the environment

As I anticipated you, the tests are executed in a separate Windows Phone application: for this reason, you’ll have to add another Windows Phone project to your solution. Once you’ve done it, you can install the <a href="https://nuget.org/packages/WPtoolkit" target="_blank">Windows Phone toolkit</a> from NuGet and <a href="https://nuget.org/packages/WPToolkitTestFx" target="_blank">the unit testing framework</a>: now we are ready to set up the application.

The first thing is to open the code behind file of the MainPage, that is **MainPage.xaml.cs** and add in the constructor, right below the **InitializeComponent()** method, the following code:

<pre class="brush: csharp;">public MainPage()
{
    InitializeComponent();
    this.Content = UnitTestSystem.CreateTestPage();
}</pre>

This way when the application is launched the unit test runner is displayed: you can see it by yourself by launching the application in the emulator. The first screen will ask you if you want to use tags or not (we’ll see later which is their purpose): if you press the **Play** button in the application bar you’ll be redirected to the main window of the unit test runner, even if you won’t see no tests at the moment since we haven’t added one yet.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="unit1" src="https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/unit1_thumb.png?resize=228%2C380" alt="unit1" width="228" height="380" border="0" data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/unit1.png)

The next step is to create one or more classes that will hold our tests: in a real scenario we would create a unit test class for every real class that we need to test. For this example, we’re simply going to create a single class: we can place it everywhere we want but, to keep the structure of the project organized, we will add a new folder ****called **UnitTests** and we will add the new class there: in my example, I’ve called it **SimpleUnitTest**. To add it, simply right click on the new folder, choose **Add – New item** and add a **Class file**.

The first thing to do is to add the namespace **Microsoft.VisualStudio.TestTools.UnitTesting** at the top of your class: this way we can access to the attributes and classes needed to test our code. The second step is to mark the entire class with the attribute **[TestClass]**: this way we’ll tell to the application that this class contains unit tests that should be processed and executed.

Now let’s add a simple unit test (to be honest, it’s more stupid than simple <img class="wlEmoticon wlEmoticon-smile" style="border-style: none;" src="https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/wlEmoticon-smile2.png?w=640" alt="Smile" data-recalc-dims="1" />): the purpose is just to make you comfortable with the basics.

<pre class="brush: csharp;">[TestClass]
public class SimpleUnitTest
{
    [TestMethod]
    public void SimpleTest()
    {
        int a = 1;
        int b = 2;

        int c = a + b;

        Assert.IsTrue(c == 3);
    }
}</pre>

Notice the **[TestMethod]** attribute that we used to mark the **SimpleTest** method: with this attribute we’re telling to the application that this method actually contains a unit test that should be executed. The test is really stupid: we sum two numbers and we check that the sum is correct, using the **Assert** class, which contains different methods that can be used to check the results of our test. In this case, we need to test a boolean condition, so we can use the **IsTrue** method. Other examples of methods are **IsNotNull** (to check that a object actually has a value) or **IsInstanceOfType** (to check if the object’s type is the one we expect).

Run the application and launch the tests: this time you’ll see the test run and you’ll be prompted with the results. Obviously, in this case, the test will pass, since 1+2 = 3.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="unit2" src="https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/unit2_thumb.png?resize=228%2C380" alt="unit2" width="228" height="380" border="0" data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/unit2.png)

Let’s test a fail case: change the condition that is tested in a way that is not true anymore, like in the example:

<pre class="brush: csharp;">[TestClass]
public class SimpleUnitTest
{
    [TestMethod]
    public void SimpleTest()
    {
        int a = 1;
        int b = 2;

        int c = a + b;

        Assert.IsTrue(c == 4);
    }
}</pre>

Run again the application: this time you’ll see that test is failed. Clicking on the test will let you see the details and why it failed: in this case, you’ll clearly understand that the **Assert.IsTrue** operation is failed.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="unit3" src="https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/unit3_thumb.png?resize=228%2C380" alt="unit3" width="228" height="380" border="0" data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/unit3.png)

### Using tags

Sometimes you don’t want to run all the tests that are available in your classes, but just a subsets: this is what tags are for. To use them, you have to add to your test class the namespace **Microsoft.Phone.Testing,** which will allow you to decorate a test method with the **Tag** attribute, followed by a keyword. To see this feature in action, let’s add another test method: we’ll set the **Tag** attribute just for one of them.

<pre class="brush: csharp;">[TestClass]
public class SimpleUnitTest
{  
    [Tag("SimpleTests")]
    [TestMethod]
    public void SimpleTest()
    {
        int a = 1;
        int b = 2;

        int c = a + b;

        Assert.IsTrue(c == 3);
    }

    [TestMethod]
    public void SimpleTest2()
    {
        int a = 3;
        int b = 1;

        int c = a + b;

        Assert.IsTrue(c == 4);
    }
}</pre>

Now run again the application and this time, in the first screen, trigger the **Use tags** switch to **On**. You will be prompted to specify a keyword that will be used by the application to determine which test run: insert the keyword **SimpleTests** (which is the tag we’ve set for the test method called **SimpleTest**) and press the Play button. You’ll see that, despite the fact you have two test methods in your class, only the first one will be executed.

Tagging is a great way to group unit tests so that, if you need to test just a specific class or feature, you don’t have to go through all the unit tests.

### Debugging

If a test fails and you don’t know, it’s really easy to debug it: since, when you launch the unit test runner from Visual Studio, you are launching a standard Windows Phone application, you can simply set breakpoints in your test methods: they will be hit when the tests are executed and you can step into the code to see what’s going on.

### Asynchronous tests, mocking and a lot more

In the [next post](http://wp.qmatteoq.com/unit-testing-in-windows-phone-8-asynchronous-operations-and-mocking/) we’ll cover some advanced but quite common scenarios, like mocking and asynchronous method testing. Keep up the good work meanwhile!