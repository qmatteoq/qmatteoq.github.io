---
id: 6246
title: 'Using Caliburn Micro with Universal Windows app &ndash; The project setup'
date: 2014-06-23T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6246
permalink: /using-caliburn-micro-with-universal-windows-app-the-project-setup/
categories:
  - Windows 8
  - Windows Phone
tags:
  - Caliburn
  - Windows 8
  - Windows Phone
---
One of the most important features that was added in Windows Phone 8.1 is the native Windows Runtime support: this new approach opened the doors to Universal Windows app, a new way to write applications for Windows and Windows Phone so that you can share most of the code. One of the biggest limitations in the Microsoft world, in the past, was that, despite the fact that Windows 8 and Windows Phone 8 have always shared many similarities (the application lifecycle, the isolated storage, the navigation approach, etc.), you ended to write two different applications, due to the many differences within the APIs implementation.

Windows Phone 8.1 has finally brought real convergence between the two platforms: there are still some differences, but you’ll be able to share most of the code and the logic between the two applications.

In the Universal Windows app world, using the MVVM (Model-View-ViewModel) pattern is a natural choice: splitting the logic and the user interface is the first rule to keep in mind when it comes to reuse the same logic in two different applications. The Universal Windows app project offers multiple ways to share code also using code behind, but it’s not an ideal solution: dealing with event handlers and having direct access to the controls makes hard to think the app as a combination of multiple aspects (the logic, the UI, etc.).

If you regularly follow my blog, <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-complete-series/" target="_blank">you should have already learned to use Caliburn Micro in a Windows Phone application</a> and to master all the powerful naming conventions and services that it offers to the developers. If you haven’t followed my previous tutorials, don’t worry: we’ll do a recap of the things to know in the next posts.

What does it change when it comes to use Caliburn Micro in a Universal Windows app? Not so much, indeed: the approach is always the same. The major difference is about the bootstrapper, which is the class that takes care of configuring in the proper way all the services needed by Caliburn Micro: we’re going to define it, in fact, in a different way than we did in Windows Phone 8.0, due to the differences between the two runtimes (Silverlight in Windows Phone 8.0, Windows Runtime in Windows Phone 8.1).

Let’s start from the beginning and see how to setup a Universal Windows app with Caliburn Micro.

### Setting up the project

Caliburn Micro has been recently upgraded to version 2.0, which fully supports Universal Windows app projects. The first step is, as usual, adding Caliburn Micro to the solution. It’s important to highlight that Universal Windows app aren’t a real application: in the end, you’re going to have two different projects, that will produce two separate packages, one for Windows 8.1 and one for Windows Phone 8.1. The major difference with a regular project is that, as part of the solution, you’ll find a **shared project,** which contains all the stuff (code, user controls, resources, assets, etc.) that will be shared among the two applications.

As a consequence, you can’t add a reference to an external library directly to the shared project: you’ll have to add Caliburn Micro in both projects, using **NuGet**. Just right click on each of them, choose **Manage NuGet packages** and look for a package called **Caliburn Micro** (<a href="http://www.nuget.org/packages/Caliburn.Micro/" target="_blank">this is the package reference</a> on NuGet’s website).

### The bootstrapper

Caliburn Micro isn’t a toolkit, but a framework: it means that, other than the basic helpers to implement MVVM in an application (like a class to support INotifyPropertyChanged), it offers also a series of services and classes that can help the developers to solve specific scenarios of the platform you’re working on. For example, since we’re talking about Window and Windows Phone, it offers helpers to manage the application’s lifecycle, the sharing feature, the navigation, etc.

As a consequence, Caliburn Micro needs to configure all these helpers when the application is started: this task is demanded to the bootstrapper, which is a special class that takes care of everything. In Caliburn Micro, the boostrapper does more than that: it takes care also of setting up the application itself. For this reason, the bootstrapper totally replaces the **App** class. The biggest difference between Windows Phone 8.0 and 8.1 is that, instead of using a separate class as bootstrapper and cleaning the **App** class by deleting all the initialization methods, we’re going to effectively replace the **App** class, by using another one called **CaliburnApplication.**

This means that the definition of the **App** class will change from this:

<pre class="brush: xml;">&lt;Application
    x:Class="MoviesTracker.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"&gt;

&lt;/Application&gt;</pre>

to this:

<pre class="brush: xml;">&lt;micro:CaliburnApplication
    x:Class="CaliburnDemo.App"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:micro="using:Caliburn.Micro"&gt;

&lt;/micro:CaliburnApplication&gt;</pre>

As you can see, we’ve replaced the **Application** class with the **CaliburnApplication** one. The second step is to change the code behind (defined in the **App.xaml.cs** file): we’re going to remove every event handler defined in the class and we will replace it with the following code.

<pre class="brush: xml;">namespace CaliburnDemo
{
    public sealed partial class App
    {
        private WinRTContainer container;

        public App()
        {
            InitializeComponent();
        }

        protected override void Configure()
        {
            container = new WinRTContainer();

            container.RegisterWinRTServices();

            container.PerRequest&lt;MainPageViewModel&gt;();
        }

        protected override void PrepareViewFirst(Frame rootFrame)
        {
            container.RegisterNavigationService(rootFrame);
        }

        protected override void OnLaunched(LaunchActivatedEventArgs args)
        {
            DisplayRootView&lt;MainPage&gt;();
        }

        protected override object GetInstance(Type service, string key)
        {
            return container.GetInstance(service, key);
        }

        protected override IEnumerable&lt;object&gt; GetAllInstances(Type service)
        {
            return container.GetAllInstances(service);
        }

        protected override void BuildUp(object instance)
        {
            container.BuildUp(instance);
        }
    }
}</pre>

First, the **App** class won’t inherit anymore from the **Application** class. Then, we have a series of methods that are used to setup the container, called **WinRTContainer**: Caliburn Micro uses the **dependency injection** approach, which means that dependencies are resolved at runtime and not at compile time. When the application starts, it registers in the container the ViewModels and the different services we’re going to use (both the standard ones and the custom ones made by us): then, when we’re going to use one of them, we’ll simply add a parameter in the constructor of the ViewModel; the container will take care of “injecting” the concrete implementation of the classes into the ViewModel. This approach is very helpful when we want to support **design time data,** which is a way to display fake data in the application when we’re designing the user interface with Visual Studio or Blend. We’ll talk more deeply about design time data in a separate post.

In the boostrapper definition there are two important methods: typically, they’re the only ones you’re going to modify during the development. The first one is called **Configure()**: it takes care of initializing the container and the Caliburn Micro services. You’re going to use it also to initialize your custom services and your ViewModels: every time you’ll add a new page to the application (which, typically, in the MVVM world means adding a new ViewModel), you’ll have to register the ViewModel inside the container in this method.

The second important method is **OnLaunched()**, which is invoked when the application starts: its important task is to redirect the user to the main page of the application. This operation is achieved by calling the **DisplayRootView<T>()** method, where **T** is the page that needs to be loaded first.

Now you’re all set: the application will automatically starts to use the Caliburn Micro services and to support the naming conventions. Let’s see now how to define the structure of our project.

### Connecting a ViewModel with the View

When you work with MVVM, you are able to connect the ViewModel to the View thanks to the **DataContext** property: the ViewModel is set as data context of the View, so that you are able to access to commands and properties defined in the ViewModel using binding.

In Caliburn, this connection is made possible using the following convention:

  * View and ViewModel should have the same name, but with different suffix: View in case of the View, ViewModel in case of the ViewModel. For example, a page called **MainPageView** and a class called **MainPageViewModel** are connected.
  * The View should be defined in a namespace which ends for **Views** and the ViewModel should be defined in a namespace which ends for **ViewModels**. This goal is typically achieved by grouping the files in folders: just create a folder called **Views**, in which you’re going to place all the pages, and a folder called **ViewModels**, in which you’re going to place all the ViewModels.

Where are we going to create all the required files? As we know, the shared project allows to share more or less everything between Windows 8.1 and Windows Phone 8.1: not only code, but also entire XAML pages. However, it’s not the best approach: sharing the logic is correct, but Windows and Windows Phone (despite the fact that they share many similarities) offers a difference experience to the user. For this reason, the best approach is to keep the Views separated, but sharing the same ViewModel. The following image shows you how to structure a Universal Windows project with Caliburn Micro: the ViewModels folder is defined inside the shared project, while the Views folders belong to every specific project.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2014/06/image_thumb1.png?resize=334%2C458" alt="image" width="334" height="458" border="0" data-recalc-dims="1" />](https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2014/06/image1.png)

### Coming soon

In this post we’ve learned how to setup Caliburn Micro in a Universal Windows app and how to define the structure of the project, so that we’re able to easily connect Views and ViewModels. In the next posts we’ll see how to use the most common naming conventions in order to properly connect the user interface with the data that we’re going to display to the user.

Meanwhile, <a href="http://1drv.ms/1lKqvF2" target="_blank">you can play with the following sample project</a>.

### Caliburn Micro 2.0 in Universal Windows apps &#8211; The complete series

  1. The project setup
  2. <a title="Using Caliburn Micro with Universal Windows app – Binding and actions" href="http://wp.qmatteoq.com/using-caliburn-micro-with-universal-windows-app-binding-actions/" target="_blank">Binding and actions</a>
  3. [Navigation](http://wp.qmatteoq.com/using-caliburn-micro-with-universal-windows-app-navigation/ "Using Caliburn Micro with Universal Windows app – Navigation")

<a href="https://github.com/qmatteoq/CaliburnMicro-UniversalApp-Demo" target="_blank">Samples available on GitHub</a>