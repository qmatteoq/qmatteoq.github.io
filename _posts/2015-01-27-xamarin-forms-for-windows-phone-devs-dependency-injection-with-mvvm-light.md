---
id: 6388
title: 'Xamarin Forms for Windows Phone devs &ndash; Dependency injection with MVVM Light'
date: 2015-01-27T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6388
permalink: /xamarin-forms-for-windows-phone-devs-dependency-injection-with-mvvm-light/
categories:
  - wpdev
  - Xamarin
tags:
  - Windows Phone
  - Xamarin
---
In the <a href="http://wp.qmatteoq.com/xamarin-forms-for-windows-phone-devs-dependency-injection/" target="_blank">previous post</a> we’ve seen how Xamarin Forms offers an easy way to manage dependency injection, so that you can support the different ways how some features are managed on every platform. This way, you can use a common interface in your shared project and then have three different concrete implementation of that interface, one for each platform. In the previous sample we’ve used a **PopupService**, which is a class that is used to display a popup to the user by taking advantage of the specific APIs offered for each platform.

In this post, we’re going to see how to combine this approach in a MVVM project created using MVVM Light.

### One project, two dependency injection’s containers

When we talked about how to create a Xamarin Forms project using MVVM Light, we’ve seen that the toolkit created by Laurent Bugnion offers a dependency injection’s container called **SimpleIoC.** Using a dependency container in a MVVM application is very useful, because it makes much easier to switch the implementation of a class we want to use in a ViewModel. In the previous post, we’ve seen that we are able to easily achieve this goal by using a code similar to the following one:

<pre class="brush: csharp;">public MainViewModel()
{
    IPopupService popupService = container.Resolve&lt;IPopupService&gt;();
    popupService.ShowPopup("Sample title", "Sample message");
}
</pre>

However, this code still requires you to go in each class and manually get a reference to the concrete implementation by using the container (in this sample, we’re using Unity as dependency injection library, so we use the **Resolve<T>()** method of the **UnityContainer** class.

Thanks to dependency injection, there’s a smarter way to do this: by registering both the ViewModel and the service in the container, like in the following sample.

<pre class="brush: csharp;">public class ViewModelLocator
{
    static ViewModelLocator()
    {
        ServiceLocator.SetLocatorProvider(() =&gt; SimpleIoc.Default);
        SimpleIoc.Default.Register&lt;IPopupService, PopupService&gt;();
        SimpleIoc.Default.Register&lt;MainViewModel&gt;();
    }

    /// &lt;summary&gt;
    /// Gets the Main property.
    /// &lt;/summary&gt;
    [System.Diagnostics.CodeAnalysis.SuppressMessage("Microsoft.Performance",
        "CA1822:MarkMembersAsStatic",
        Justification = "This non-static member is needed for data binding purposes.")]
    public MainViewModel Main
    {
        get
        {
            return ServiceLocator.Current.GetInstance&lt;MainViewModel&gt;();
        }
    }
}
</pre>

As you can see, in the **ViewModelLocator** definition we’ve registered not only the **MainViewModel** (like we did in the previous post) but also the **PopupService** class, which is connected to the **IPopupService** interface. Thanks to this code, now we are able to get a reference to the **PopupService** class simply by adding an **IPopupService** paramter in the ViewModel’s constructor, like in the following sample:

<pre class="brush: csharp;">public class MainViewModel : ViewModelBase
{
   private readonly IPopupService _popupService;

   public MainViewModel(IPopupService popupService)
   {
       _popupService = popupService;
   }
}
</pre>

The dependency injection mechanism will take care of automatically injecting, into the **IPopupService** paramater, the concrete implementation (the **PopupService**) class we’ve registered in the container in the **ViewModelLocator**.

### How to combine the two approaches

However, <a href="http://wp.qmatteoq.com/xamarin-forms-for-windows-phone-devs-dependency-injection/" target="_blank">in the previous post</a>, we’ve seen that Xamarin Forms offers another approach to manage dependency injection: by decorating the concrete implementation of the interface (in our case, the **PopupService** class) with an attribute, that allows us to use a single interface in our shared project and have three different implementations in each platform’s specific project. This way, we can deal with the fact that some features are in common across every platform (like displaying a popup or geo localizing the user) but they are implemented with different APIs and approaches.

How can we combine this approach with the standard one, so that we take the best of both worlds? Our goal is to have the MVVM Light container to automatically inject, in every ViewModel, the specific **PopupService** implementation we’ve included into every platform’s specific project. It’s easy, thanks to a feature offered basically by each dependency injection’s library, which is a method to register a specific instance of a class into the container. This way, when we need a concrete implementation of a class, the container will return us that specific instance, instead of creating a new one on the fly.

The following code shows how we can achieve our goal by combining the code we’ve seen in this post and in the previous one:

<pre class="brush: csharp;">public class ViewModelLocator
{
    static ViewModelLocator()
    {
        ServiceLocator.SetLocatorProvider(() =&gt; SimpleIoc.Default);
        IPopupService popupService = DependencyService.Get&lt;IPopupService&gt;();
        SimpleIoc.Default.Register&lt;IPopupService&gt;(() =&gt; popupService);
        SimpleIoc.Default.Register&lt;MainViewModel&gt;();
    }

    /// &lt;summary&gt;
    /// Gets the Main property.
    /// &lt;/summary&gt;
    [System.Diagnostics.CodeAnalysis.SuppressMessage("Microsoft.Performance",
        "CA1822:MarkMembersAsStatic",
        Justification = "This non-static member is needed for data binding purposes.")]
    public MainViewModel Main
    {
        get
        {
            return ServiceLocator.Current.GetInstance&lt;MainViewModel&gt;();
        }
    }
}
</pre>

Unlike in the previous sample, where we generically registered the **PopupService** class for the **IPopupService** interface, in this case we register a specific instance, which we have retrieved using the **DependencyService** class offered by Xamarin Forms. This way, we make sure that the **IPopupService** object we get in return is the specific implementation for the platform where the app is running. Then, we proceed to register this instance in the **SimpleIoc** container, by passing it as parameter of the **Register<T>()** method.

That’s all: now, in our **MainViewModel**, the container will automatically inject, into the constructor’s parameter, the proper implementation for the current platform.

### Wrapping up

In this post we’ve seen how to combine a standard dependency injection approach (which is useful to manage the ViewModel dependencies) with the Xamarin Forms one (which is useful to manage platform specific implementations of the same feature). You can play with a working sample by downloading the source code published on GitHub: [https://github.com/qmatteoq/XamarinFormsSamples](https://github.com/qmatteoq/XamarinFormsSamples "https://github.com/qmatteoq/XamarinFormsSamples")