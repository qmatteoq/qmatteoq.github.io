---
id: 6377
title: 'Xamarin Forms for Windows Phone devs &#8211; Dependency injection'
date: 2015-01-23T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6377
permalink: /xamarin-forms-for-windows-phone-devs-dependency-injection/
categories:
  - Xamarin
tags:
  - Windows Phone
  - Xamarin
---
If you’re already worked with Windows Phone and Windows Store applications and, especially, with the MVVM pattern, you should be familiar with the **dependency injection** concept. In a typical application, when you need to use a class, you simply create a new instance, like in the following sample:

<pre class="brush: csharp;">private void OnButtonClicked(object sender, EventArgs e)
{
    PopupService popupService = new PopupService();
    popupService.ShowPopup("Sample title", "Sample message");
}
</pre>

This way, objects are created at compile time. However, when you work with the MVVM pattern, this approach has a downside. Let’s say that you’re using the **PopupService** in multiple classes (like ViewModels) and, suddenly, you need to change his implementation with a new one. With the previous approach, you are forced to go in each class where you use the **PopupService** and change the implementation with the new one.

With the dependency injection approach, instead, objects are registered inside a **container**, which is a special class that takes care of dispatching the objects when they’re required. Typically, with this approach, every class is described by an interface. For example, the **PopupService** class can be described with an interface called **IPopupService**, like in the following sample:

<pre class="brush: csharp;">public interface IPopupService
{
    void ShowPopup(string title, string message);
}
</pre>

Then, when the application starts, we specify for each interface which is the concrete implementation we want to use in the application, like in the following code (keep in mind that it’s just a sample, there are multiple libraries to implement dependency injection, each of them with its APIs and methods):

<pre class="brush: csharp;">public App() 
{
    IUnityContainer container = new UnityContainer();
    container.RegisterType&lt;IPopupService, PopupService&gt;();
}
</pre>

In the end, whenever a class needs to use a **PopupService** object, instead of simply creating a new instance, it asks to the container to return the registered one, like in the following sample:

<pre class="brush: csharp;">private void OnButtonClicked(object sender, EventArgs e)
{
    IPopupService popupService = container.Resolve&lt;IPopupService&gt;();
    popupService.ShowPopup("Sample title", "Sample message");
}
</pre>

The advantage of this approach should be clear: whenever we need to change the implementation of the **IPopupService** interface, it’s enough to change the concrete implementation of the interface that is registered in the container, like:

<pre class="brush: csharp;">public App() 
{
    IUnityContainer container = new UnityContainer();
    container.RegisterType&lt;IPopupService, FakePopupService&gt;();
}

</pre>

Automatically, all the classes that are using the **PopupService** class will immediately start to use the new implementation, called **FakePopupService**, simply by changing one line of code.

### 

### Dependency injection and Xamarin Forms

Dependency injection becomes very useful when you work with Xamarin Forms: the purpose of this technology is to allow developers to share as much code as possible between the three different mobile platforms (iOS, Android and Windows Phone). The Xamarin technology was created with this purpose in mind, however Xamarin Forms takes this approach to the next level, by allowing developers to share not just business logic, but also the user interface. Xamarin Forms, in fact, uses a XAML based approach: the user interface is defined using XML, where each control is identified by a specific XML tag. The biggest difference with the standard XAML (which is supported only by Microsoft technologies, like Windows Phone or WPF) is that the controls are automatically translated into the native platform controls. This way, unlike with cross platform applications based on web technologies (which offer the same UI on all the platforms), we’ll be able to keep the UI consistent with the guidelines and the user interface of the platform.

However, there are some scenarios which simply don’t fit the shared code approach. We can’t forget, in fact, that Xamarin, unlike web technologies, doesn’t provide a way to create the application just once and run it everywhere: one of the biggest pros of Xamarin, in fact, is that it allows developers to make use of every platform specific feature, unlike web applications that typically support only the features that are in common between every platform. Consequently, you still need to learn how Android and iOS development work if you want to create a real application. However, thanks to Xamarin, you won’t have to learn also a new language in the process: Xamarin, in fact, offers a way to use the native APIs with the familiar C# syntax.

The same applies for Xamarin Forms: it offers a way to share not just business logic but also user interface code but, in the end, we still need to deal with the specific platform features and implementations. Let’s say, for example, that we want to add the **PopupService** class we’ve previously seen in our Xamarin Forms project, which offers a **ShowPopup()** method that displays an alert to the user. Each platform has a different way to display a popup message: for example, in Windows Phone you use the **MessageBox** class; on Android, you have the **AlertDialog** class; on iOS, instead, you use the **UIAlertView** class. However, we would love to have a way to use the **ShowPopup()** method in our shared page and, automatically, see it rendered on each platform with its specific code.

Thanks to the dependency injection approach, we can: in the shared page we’re going to get a reference to the **IPopupService** class and to use the **ShowPopup()** method. At runtime, the dependency container will inject into the **IPopupService** object the specific implementation for the platform. However, compared to a regular dependency injection approach (like the one we’ve previously seen), there are some differences when we need to use it in Xamarin Forms.

**Please note:** the sample we’re going to see has been created just for demonstration purposes. Xamarin Forms, in fact, already offers a way to display popups to the user in a shared page, without having to deal with the different implementations for each platform.

### One interface, multiple implementations

The first step is to create a common interface in the shared project, since it will be unique for each platform:

<pre class="brush: csharp;">public interface IPopupService
{
    void ShowPopup(string title, string message);
}
</pre>

Then we need a concrete implementation of this interface, one for each platform: we’re going to create in every specific platform project this time. For example, here is how it looks like the implementation in the Windows Phone project:

<pre class="brush: csharp;">public class PopupService: IPopupService
{
    public void ShowPopup(string title, string message)
    {
        MessageBox.Show(message, title, MessageBoxButton.OK);
    }
}
</pre>

Here is, instead, how it looks like in the Android project:

<pre class="brush: csharp;">public class PopupService: IPopupService
{
    public void ShowPopup(string title, string message)
    {
        AlertDialog.Builder alert = new AlertDialog.Builder(Forms.Context);
        alert.SetTitle(title)
            .SetMessage(message)
            .SetPositiveButton("Ok", (sender, args) =&gt;
            {
                Debug.WriteLine("Ok clicked"); 
            })
            .Show();
    }
}
</pre>

The next step is to understand how to use the proper implementation for each platform. In our shared code, we’re going to simply use the interface, like in the following sample:

<pre class="brush: csharp;">private void OnButtonClicked(object sender, EventArgs e)
{
    IPopupService popupService = new PopupService();
    popupService.ShowPopup("Sample title", "Sample message");
}
</pre>

However, this code won’t simply work: we don’t have a single implementation of the **PopupService** class, but three different implementations, each of them with its namespace. Also the previous dependency injection approach we’ve seen doesn’t solve our problem: when we register the implementation in the container, we still need to specify which is the concrete class to use and, in our scenario, we have three of them.

Luckily, Xamarin Forms offers a smart approach to solve this situation: instead of manually registering the implementations into a container, we decorated the classes with an attribute. At runtime, automatically, Xamarin Forms will detect which is the interface connected to the implementation and will return to the application the proper object. To make it working, it’s enough to add the following attribute each concrete implementation of the class:

<pre class="brush: csharp;">using System.Windows;
using DependencySample.Services;
using DependencySample.WinPhone.Services;


[assembly: Xamarin.Forms.Dependency(typeof(PopupService))]
namespace DependencySample.WinPhone.Services
{
    public class PopupService: IPopupService
    {
        public void ShowPopup(string title, string message)
        {
            MessageBox.Show(message, title, MessageBoxButton.OK);
        }
    }
}

</pre>

The only variable part of the attribute is the parameter of the **Dependency** class: we need to specify the type of the current class (in our sample, it’s **PopupService**). Then, in our shared project, when we need to use the **PopupService**, we’re going to retrieve it using a Xamarin Forms class called **DependencyService**, like in the following sample:

<pre class="brush: csharp;">private void OnButtonClicked(object sender, EventArgs e)
{
    IPopupService popupService = DependencyService.Get&lt;IPopupService&gt;();
    popupService.ShowPopup("Sample title", "Sample message");
}
</pre>

We use the **Get<T>** method, where **T** is the interface of the class we want to use. Automatically, Xamarin Forms will analyze the registered DLLs in the project and will return the concrete implementation that is available in the platform’s specific project. 

### 

### Wrapping up

In this post we’ve seen how to use the native dependency container that Xamarin Forms offers to developers. In the next post, we’ll see how to combine it with a traditional dependency injection approach, which comes useful when you’re developing a Xamarin Forms app using MVVM. You can download the sample project used in this post on GitHub: [https://github.com/qmatteoq/XamarinFormsSamples](https://github.com/qmatteoq/XamarinFormsSamples "https://github.com/qmatteoq/XamarinFormsSamples")