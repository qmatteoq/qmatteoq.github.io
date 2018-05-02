---
id: 6354
title: 'Xamarin Forms for Windows Phone devs &ndash; The first project'
date: 2015-01-13T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6354
permalink: /xamarin-forms-for-windows-store-app-devs-the-first-project/
categories:
  - wpdev
  - Xamarin
tags:
  - Xamarin
---
<a href="http://wp.qmatteoq.com/first-steps-with-xamarin-forms-a-brief-introduction/" target="_blank">In the previous post</a>, we’ve seen a brief introduction to Xamarin and Xamarin Forms. In this post we’ll start to play for real with a Xamarin Forms project.

### Creating the project

After you’ve installed the Xamarin Tools, you’ll find a set of new templates in Visual Studio. The ones for Xamarin Forms are inside the category **Mobile apps**:

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/image_thumb.png?resize=536%2C308" alt="image" width="536" height="308" border="0" data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/image.png)

There are two ways to create a Xamarin Form project, which should be familiar to every developer that has already worked with Universal Windows apps:

  * **Xamarin Forms Portable**: with this template, the shared project between all the different platforms will be a Portable Class Library (PCL).
  * **Xamarin Forms Shared**: with this template, we’re going to use the same approach used by the Universal Windows apps template. The shared project isn’t a real library, but it’s a collection of files (classes, assets, etc.) that are linked to the platform specific projects. This way, you’re going to host the common files in a single project, but they will be automatically copied and linked to the platform specific project during the build process.

The two approaches are very similar: the main difference is that, with the the shared template, you’ll be able to use conditional compilation in case you want to execute platform specific code in a common class. However, Xamarin Forms, by using the dependency injection approach, offers a smarter and cleaner way to manage platform specific code, so I’ll suggest you to use the Portable Class Library approach. It’s the template we’re going to use in the next samples.

When you create a new Xamarin Forms project, Visual Studio will create a new solution with four different projects: the shared Portable Class Library and three platform specific projects, one for iOS, one for Android and one for Windows Phone. Please note that, unfortunately, at the time of writing, Xamarin Forms still doesn’t support Windows Phone 8.1 and Universal Windows apps: the Windows Phone project created by Xamarin will target the 8.0 platform.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/image_thumb1.png?resize=424%2C150" alt="image" width="424" height="150" border="0" data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/image1.png)

In a typical Xamarin Forms project, most of the assets, code and views will be included in the shared project: the platform specific projects will just take care of initialing the Xamarin Forms infrastructure. If you take a look at the main page of each project, in fact, you’ll find that it will just take care of initializing Xamarin Forms and loading the **App** class of the shared project, which is is the starting point of the application. Let’s take a look, for example, at the definition of the **MainPage.xaml.cs** file in the Windows Phone project, which should be the most familiar technology for you if you’re reading this blog:

<pre class="brush: csharp;">public partial class MainPage : FormsApplicationPage
{
    public MainPage()
    {
        InitializeComponent();

        Forms.Init();
        LoadApplication(new Sample.App());
    }
}
</pre>

The initialization is performed by the **Forms.Init()** method, while the application is loaded by the **LoadApplication()** method.

However, as we’re going to see in the next posts, the various projects can also contain platform specific classes that use APIs or features that are available only on one platform or that are implemented in a different way.

### Checking for Xamarin Forms upgrades

Xamarin Forms is simply a collection of libraries that are available on NuGet and that are referenced automatically in every new Xamarin Forms project. Consequently, the first step is to check if there are upgrades to the library, since Xamarin can update it independently from the core Xamarin runtime. At the moment of writing, for example, the latest Xamarin Forms version is 1.3, while the stable Xamarin tools still create projects that are based on the old 1.2 version. Consequently, unless you’re using the beta channel (which already offers the updated templates), you will have to manually upgrade the projects.

The first step is to right click on the solution, choose **Manage NuGet Packages**� **for solution** and check for updates on the Xamarin Forms project. If you’re still using the Xamarin stable channel and the 1.2 templates , you will need to make some changes to the code, in order to support the new features. Xamarin has published a <a href="https://developer.xamarin.com/guides/cross-platform/macios/updating-xamarin-forms-apps/" target="_blank">detailed tutorial</a> on the steps to perform.

**Update:�** Xamarin yesterday released the 3.9 version in the Stable channel, which already provides the correct Visual Studio templates to create Xamarin Forms apps based on version 1.3.

### Creating the first view

The main class of a Xamarin Forms app is called **App** and takes care of initializing the first page of the application, which is provided by the **MainPage** property. If you open the standard **App.cs** file created by the Xamarin Forms templates, you’ll find the following code:

<pre class="brush: csharp;">public class App : Application
{
    public App()
    {
        this.MainPage = new ContentPage
        {
            Content = new Label
            {
                Text = "Hello, Forms !",
                VerticalOptions = LayoutOptions.CenterAndExpand,
                HorizontalOptions = LayoutOptions.CenterAndExpand,
            },
        };
    }
}
</pre>

This sample shows you that there are two ways to define a page in Xamarin Forms: by creating the layout in code behind and assigning it to the **Content** property of the page or by using XAML. Many samples in the web use the first approach, but I prefer the XAML one, since it’s more similar to the traditional Windows Store and Windows Phone development. The first step, consequently, is to create a new XAML page, by right clicking on your project and by choosing **Add new item** an selecting **Forms XAML Page** as template. The page will look like a page in a Windows Phone app, with a .xaml file (with the layout) and a xaml.cs file (with the code behind class).

Here is the XAML page:

<pre class="brush: xml;">&lt;?xml version="1.0" encoding="utf-8" ?&gt;
&lt;ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
                       xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                       x:Class="App2.SamplePage"&gt;
    &lt;Label Text="{Binding MainText}" VerticalOptions="Center" HorizontalOptions="Center" /&gt;
&lt;/ContentPage&gt;
</pre>

As you can see, it looks like a regular Windows Phone page, so the approach should be very familiar for you. You will just have to study a bit <a href="http://developer.xamarin.com/guides/cross-platform/xamarin-forms/controls/" target="_blank">the official documentation</a> to learn the differences in the controls and properties naming. For example, the **TextBlock** control is called **Label**, while the **TextBox** control is called **Entry;** the label of a button is set using the **Text** property instead of the **Content** one; binding is supported, but the data context is set using the **BindingContext** property instead of the **DataContext** one; etc.

Here is, instead, the code behind file:

<pre class="brush: csharp;">public partial class SamplePage
{
    public SamplePage()
    {
        InitializeComponent();
    }
}
</pre>

Once you’ve created the first page, you’ll have to change the **App** code to use it, in replacement of the sample page that is created in code. It’s enough to assign to the **MainPage** property a new instance of the page we’ve just created, like in the following sample:

<pre class="brush: csharp;">public class App : Application
{
   public App()
   {
       this.MainPage = new SamplePage();
   }
}

</pre>

However, the previous code is quite useless: the page we’ve created it’s a **ContentPage** (as you can see from the name of the root tag in the XAML), which is just a placeholder for the content and doesn’t provide any option that is required when you develop a complex application, like a navigation system to move to other pages or the ability to manage different sections. This is one of the main differences between Windows Phone development and Xamarin Forms: the different concept of pages.

### Managing the pages

If you’re a Windows Phone developer, there’s an important difference to understand about the views and the navigation framework. When you create a Windows Phone application, you simply work with pages, that are identified by the **Page** class in the Windows Runtime or the **PhoneApplicationPage** class in Silverlight. There are no differences between a page with a Pivot control, a Hub control or a ListView control: they are all pages, what changes are the controls that are placed inside them.

In Xamarin Forms, instead, there are different kind of pages, which all inherit from the base **Page** class. Most of them are mapped with different Windows Phone layouts:

  * **ContentPage** is the standard simple page.
  * **NavigationPage** is a page that offers a navigation framework to move from one page to another.
  * **TabbedPage** is a page that offers quick access to multiple sections. It’s represented as a page with a Pivot control in Windows Phone.
  * **CarouselPage** is a page that allows the user to scroll between different pages. It’s represented as a page with a Panorama / Hub control in Windows Phone.

This approach is required because there are many differences between the navigation system of each platform: for example, since Windows Phone has a hardware Back button, the UI doesn’t need to include a software button. However, Android and iOS have a different approach and, in case it’s a page that supports navigation towards another page, they display a navigation bar at the top, that it’s used to move back and forward to the different pages.

Another example is the usage of tabs: in Windows Phone, when an application has different sections or different contents to display, we use a Pivot or a Hub control, which is a way to divide a page into different sub sections, each of them with its content. In Android and iOS, instead, sections are managed with tabs: there’s a main page, with different tabs, and a page for each section, that are displayed inside the tabs.

Consequently, as we’re going to see later, most of the time you’ll work with the **ContentPage** type, which is the standard page that is used to display information to the user. The other page types, instead, are invisible to the user and they will be used as a container for content pages.

#### The ContentPage

**ContentPage** is the basic page that simply displays some content to the user: when you add a new XAML page to your Xamarin Forms project, by default its type is **ContentPage**, as you can see by the name of the root node in the XAML definition:

<pre class="brush: xml;">&lt;?xml version="1.0" encoding="utf-8" ?&gt;
&lt;ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
                       xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                       x:Class="Views.Views.MainView"&gt;
    &lt;Label Text="{Binding MainText}" VerticalOptions="Center" HorizontalOptions="Center" /&gt;
&lt;/ContentPage&gt;
</pre>

When you need to define a page that displays some information to the user, you’ll always work with a **ContentPage.** Consequently, when you create a new XAML Forms Page using the Visual Studio template, you’ll always create by default a **ContentPage** one, since it’s the only “visible” kind of page, that can display some content to the user. All the other page’s types are “invisible” and simply acts as a container for other **ContentPage** pages.

#### 

#### The NavigationPage

A **ContentPage** alone, however, in most of the cases it’s useless, unless your application is composed by just one page. From a <span style="font-weight: bold;">ContentPage</span>, in fact, the user doesn’t have a way to move to another page or to another section. This is why Xamarin Forms offers a **NavigationPage:** it’s simply a container for a **ContentPage,** which adds a **Navigation** property that offers some method to move from one page to another.

A **NavigationPage** is invisible to the user: this is why, when you create a new instance of the class, you need to pass as parameter the **ContentPage** you want to “encapsulate” into the navigation framework. The following sample shows how to define the **App** class to use this approach:

<pre class="brush: csharp;">public class App : Application
{
    public App()
    {
        NavigationPage navigationPage = new NavigationPage(new MainView());
        this.MainPage = navigationPage;
    }
}
</pre>

By encapsulating the **MainView** (which is a **ContentPage**) into a **NavigationPage**, we will be able to navigate to another page by using the **PushAsync()** method of the **Navigation** property (we’ll talk in details about the navigation system in the next post):

<pre class="brush: csharp;">private async void Button_OnClicked(object sender, EventArgs e)
{
    await Navigation.PushAsync(new SecondView());
}
</pre>

#### The TabbedPage and CarouselPage

As already mentioned, the **TabbedPage** and **CarouselPage** classes are used when you want to display different sections of the application, each of them represented by a different **ContentPage**. Tabbed pages are rendered with tab controls on Android and iOS, while on Windows Phone with a Pivot control, as you can see in the following image:

<img src="http://iosapi.xamarin.com/monodoc.ashx?link=source-id:1:TabbedPage.TripleScreenShot.png" alt="" width="498" height="249" />

The **CarouselPage**, instead, display the pages one near the other and the user can browse them with a swipe gesture. This approach is more familiar to Windows Phone developers, since it’s the same experience offered by the Panorama and Hub controls:

<img src="http://iosapi.xamarin.com/monodoc.ashx?link=source-id:1:CarouselPage.TripleScreenShot.png" alt="" width="492" height="246" />

&nbsp;

Like for the **NavigationPage**, these two page types are just a container for other **ContentPage** pages, which define the real content that is displayed in each page. The following sample shows how to define a **TabbedPage** with two sections:

<pre class="brush: csharp;">public class App : Application
{
    public App()
    {
        TabbedPage tabbedPage = new TabbedPage {Title = "Tabbed page"};
        tabbedPage.Children.Add(new FirstView());
        tabbedPage.Children.Add(new SecondView());

        this.MainPage = tabbedPage;

    }
}
</pre>

**FirstView** and **SecondView** are two **ContentPage** pages, which have been added in the shared project. The title of the tab is taken from the title of the page, which is set by the **Title** property of the **ContentPage**, which can be set directly in XAML like in the following sample:

<pre class="brush: xml;">&lt;?xml version="1.0" encoding="utf-8" ?&gt;
&lt;ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             Title="First view"
             x:Class="Views.Views.FirstView"&gt;
    &lt;Label Text="First view" VerticalOptions="Center" HorizontalOptions="Center" /&gt;
&lt;/ContentPage&gt;
</pre>

Eventually, you can also set an icon to use as section’s identifier, by using the **Icon** property: some platforms (like iOS), in fact, can display also an image other than just a text in the tab to identify the section.

### Wrapping up

In this post we’ve seen how to setup our first project and which are the similarities and the differences between developing a Windows Store app and a Xamarin Forms app. Once you’ve defined the startup page of your application, you can start working with it in the same way you do in a Windows Store app: you can add controls in the XAML page, subscribe to event handlers and manage the user’s interaction in the code behind. In the next post we’ll see in details how to manage the application’s lifecycle and the navigation, then we’ll move on to understand how to implement the MVVM pattern in a Xamarin Foms app. Meanwhile, you can play with the sample project published at [https://github.com/qmatteoq/XamarinFormsSamples](https://github.com/qmatteoq/XamarinFormsSamples "https://github.com/qmatteoq/XamarinFormsSamples")