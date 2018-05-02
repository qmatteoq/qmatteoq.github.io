---
id: 6375
title: 'Xamarin Forms for Windows Phone devs &ndash; Using the MVVM pattern'
date: 2015-01-20T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6375
permalink: /xamarin-forms-for-windows-phone-devs-using-the-mvvm-pattern/
categories:
  - wpdev
  - Xamarin
tags:
  - Windows Phone
  - Xamarin
---
We’ve already talked many times about the MVVM pattern on this blog and how to implement it in Windows Phone 8 apps using Caliburn Micro or in Universal Windows apps with Caliburn Micro and Prism. The Model-View-ViewModel pattern is very useful in XAML based projects, because the separation between logic and user interface gives many advantages in testability and maintainability. However, when we’re dealing with projects that target multiple platforms with a shared code base (like with Universal Windows apps), using the MVVM pattern is, more or less, a basic requirement: thanks to the separation between logic and user interface, it becomes easier to share a good amount of code with the different projects. 

Xamarin Forms makes no exceptions: applying the MVVM pattern is the best way to create a common codebase that can be shared among the iOS, Android and Windows Phone projects. In this post, we’ll see how to create a simple project using one of the most popular toolkits out there: MVVM Light.

### Why MVVM Light?

MVVM Light is, for sure, the most simple and flexible MVVM toolkit available right now. It’s main advantage is simplicity: since it’s implementation is very basic, it can be easily ported from one platform to another. As you’re going to see in this post, if you have already worked with MVVM Light on other platforms, you’ll find yourself at home: except for some minor difference, the approach is exactly the same you would use in Windows Phone, Windows Store or WPF.

However, the MVVM Light simplicity is also its weak point: compared to frameworks like Caliburn Micro or Prism, it misses all the infrastructure that is often required when you have to deal with platform specific features, like navigation, application lifecycle, etc. Consequently, as we’re going to see in the next posts, you may have the need to extend MVVM Light, in order to solve platform specific scenarios. In the next post I will show you the implementation I did to solve these problem: for now, let’s just focus on implementing a basic Xamarin Forms project with MVVM Light. This knowledge, in fact, it’s important to understand the next posts I’m going to publish.

### Creating the first MVVM project

The first step is the same we’ve seen in a previous post: creating a new Xamarin.Forms Blank App. After we’ve checked that we’re using the latest Xamarin Forms version, we need to install also MVVM Light in the shared project: you’ll need to install the specific version for PCL libraries, which is [http://www.nuget.org/packages/Portable.MvvmLightLibs/](http://www.nuget.org/packages/Portable.MvvmLightLibs/ "http://www.nuget.org/packages/Portable.MvvmLightLibs/")

Now you’re ready to set up the infrastructure to create the application: let’s start by adding our first View and our first ViewModel. It’s not required, but to better design the application I prefer to create a folder for the views (called Views) and a folder for the ViewModels (called ViewModels). Then add a new Xamarin Forms page into the Views folder (called, for example, **MainView.xaml**) and a new simple class into the ViewModels folder (called, for example, **MainViewModel.cs).**

### The ViewModelLocator

One of the basic requirements in a MVVM application is to find a way to connect a View to its ViewModel: we could simply create a new instance of the ViewModel and assign it as data context of the page, but this way we won’t be able to use techniques like dependency injection to resolve ViewModels and the required services at runtime (we’ll talk again about this approach in the next post). The typical approach when you work with MVVM Light is to create a class called **ViewModelLocator**, which takes care of passing to every View the proper ViewModel. Here is how the **ViewModelLocator** class looks like in a Xamarin Forms project:

<pre class="brush: csharp;">public class ViewModelLocator
{
    static ViewModelLocator()
    {
        ServiceLocator.SetLocatorProvider(() =&gt; SimpleIoc.Default);
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

When the class is created, we register the default dependency injection provider we want to use: in this case, we use the native one offered by MVVM Light, called **SimpleIoc**. Then, we register in the container, by using the **Register<T>()** method, all the ViewModels and services we want to use. In this sample, we won’t use any service: we’ll see in the next post how to manage them; so we just register our **MainViewModel** in the container. The next step is to create a property that will be used by the View to request the proper ViewModel instance: we use again the dependency injection container, in this case to get a registered class, by using the **GetInstance<T>()** method (where **T** is the object’s type we need).

Now we can use the ViewModelLocator to assing a ViewModel to its View: in our case, the **MainView** should be connected to the **MainViewModel**. In a Windows Phone app, this goal is typically achieved by declaring the **ViewModelLocator** as a global resource in the **App** class and then, in the XAML, assigning the proper property (in this case, **Main**) to the **DataContext** property of the page. This way, the ViewModel will be assigned as **DataContext** of the entire page and every nested control will be able to access to the commands and properties that are exposed by the ViewModel.

However, this approach doesn’t work in Xamarin Forms, since we don’t have the concept of global resources: we don’t have an **App.xaml** file, where to declare resources that are shared across every page of the application. The easiest way to solve this problem is to declare the **ViewModelLocator** as a static property of the **App** class in the Xamarin Forms shared project, like in the following sample:

<pre class="brush: csharp;">public class App: Application
{
    public App()
    {
        this.MainPage = new MainView();
    }

    private static ViewModelLocator _locator;

    public static ViewModelLocator Locator
    {
        get
        {
            return _locator ?? (_locator = new ViewModelLocator());
        }
    }
}
</pre>

This way, you’ll be able to connect the ViewModel to the View by using this static property in the code behind file of the page (in our case, the file **MainView.xaml.cs**):

<pre class="brush: csharp;">public partial class MainView
{
    public MainView()
    {
        InitializeComponent();
        this.BindingContext = App.Locator.Main;
    }
}
</pre>

You can notice one of the most important differences between the XAML in Windows Phone and the XAML in Xamarin Forms: the **DataContext** property is called **BindingContext**. However, its purpose is exactly the same: define the context of a control.

### 

### Define the ViewModel

Creating a ViewModel it’s easy if you’ve already worked with MVVM Light, since the basic concepts are exactly the same. Let’s say that we want to create a simple applications where the user can insert his name: by pressing a button, the page will display a message to say hello to the user. Here is how the ViewModel to manage this scenario looks like:

<pre class="brush: csharp;">public class MainViewModel: ViewModelBase
{
    private string _name;

    public string Name
    {
        get { return _name; }
        set
        {
            Set(ref _name, value);
            ShowMessageCommand.RaiseCanExecuteChanged();
        }
    }

    private string _message;

    public string Message
    {
        get { return _message;}
        set { Set(ref _message, value); }
    }

    private RelayCommand _showMessageCommand;

    public RelayCommand ShowMessageCommand
    {
        get
        {
            if (_showMessageCommand == null)
            {
                _showMessageCommand = new RelayCommand(() =&gt;
                {
                    Message = string.Format("Hello {0}", Name);
                }, () =&gt; !string.IsNullOrEmpty(Name));
            }

            return _showMessageCommand;
        }
    }
}
</pre>

You can see, in action, all the standard features of a ViewModel created using MVVM Light as a toolkit:

  * The ViewModel inherits from the **ViewModelBase** class, which gives you some helpers to properly support the **INotifyPropertyChanged** interface that is required to notify the controls in the View when the properties that are connected through binding are changed. 
      * Every property isn’t defined with the standard get – set approach but, when the value of the property changes (in the **set** method), we call the **Set()** method offered by MVVM Light which, other than just assigning the value to the property, takes care of dispatching the notification to the controls in the View. 
          * When you work with the MVVM pattern, you can’t react to user’s actions using event handlers, since they have a strict dependency with code behind: you can’t declare an event handler inside another class. The solution is to use **commands**, which are a way to express actions with a property, that can be connected to the View using binding. MVVM Light offers a class that makes this scenario easier to implement, called **RelayCommand**. When you create a **RelayCommand** object, you need to set: 1) the action to perform (in our case, we define the message to display to the user) 2) optionally, the condition that needs to be satisfied for the command to be activated (in our case, the user will be able to invoke the command only if the property called **Name** isn’t empty). If the condition isn’t met, the control be automatically disabled. In our sample, if the user didn’t insert his name in the box, the button will be disabled. 
              * When the value of the **Name** property changes, we also call the **RaiseCanExecuteChanged()** method offered by the **RelayCommand** we’ve just defined: this way, every time the **Name** property changes, we tell to the command to evaluate its status, since it could be changed.</ul> 
            ### 
            
            ### The View
            
            The following code, instead, show the Xamarin Forms XAML page that is connected to the ViewModel we’ve previously seen:
            
            <pre class="brush: xml;">&lt;?xml version="1.0" encoding="utf-8" ?&gt;
&lt;ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
                       xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
                       x:Class="MvvmLight_Sample.Views.MainView"&gt;
  &lt;StackLayout Padding="12, 0, 12, 0"&gt;
    &lt;Label Text="Insert your name:" /&gt;
    &lt;Entry Text="{Binding Path=Name, Mode=TwoWay}" /&gt;
    &lt;Button Command="{Binding Path=ShowMessageCommand}" Text="Show message" /&gt;
    &lt;Label Text="{Binding Path=Message}" FontSize="30" /&gt;
  &lt;/StackLayout&gt;
&lt;/ContentPage&gt;
</pre>
            
            It’s a simple form, made by a text area where the user can insert his name and a button that, when it’s pressed, displays the message using a label. You can notice some difference with the standard XAML available in Windows Phone:
            
              * <div align="left">
                  The <strong>StackPanel</strong> control, which is able to display the nested controls one below the other, is called <strong>StackLayout</strong>.
                </div>
            
              * <div align="left">
                  To define the distance of the control from the screen’s border, we use the <strong>Padding</strong> property instead of the <strong>Margin</strong> one.
                </div>
            
              * The **TextBlock** control, used to display a text to the user, is called **Label**.
              * The **TextBox** control, used to receive the input from the user, is called **Entry.**
              * The content of the button (in this case, a text) is set using the **Text** property, while in the standard XAML is called **Content** and it accepts also a more complex XAML layout.
            
            Except for these differences in the XAML definition, we’re using standard binding to connect the controls with the properties defined in the ViewModel:
            
              * The **Entry** control has a property called **Text**, which contains the name inserted by the user: we connect it to the **Name** property of the ViewModel, using the two-way binding.
              * The **Button** control has a property called **Command**, which is connected to the **ShowMessageCommand** we’ve defined in the ViewModel. This way, the button will be enabled only if the **Name** property isn’t empty; if it’s enable, by pressing it we’ll display the hello message to the user.
              * The hello message is stored into the **Message** property of the ViewModel, which is connected using binding to the last **Label** control in the page.
            ### Wrapping up
            
            In this post we’ve seen the basic concepts of using MVVM Light in a Xamarin Forms: except for some differences (like the **ViewModelLocator** usage), the approach should be very familiar to any Windows Phone developer that has already worked with the MVVM pattern and the MVVM Light toolkit. In the next posts we’ll take a look at how the dependency injection approach works in Xamarin Forms and how we can leverage it in a MVVM Light project. As usual, you can find the sample code used in this post on GitHub at [https://github.com/qmatteoq/XamarinFormsSamples](https://github.com/qmatteoq/XamarinFormsSamples "https://github.com/qmatteoq/XamarinFormsSamples")