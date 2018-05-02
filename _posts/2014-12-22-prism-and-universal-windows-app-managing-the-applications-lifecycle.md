---
id: 6328
title: 'Prism and Universal Windows app &ndash; Managing the application&rsquo;s lifecycle'
date: 2014-12-22T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6328
permalink: /prism-and-universal-windows-app-managing-the-applications-lifecycle/
categories:
  - Universal Apps
  - wpdev
tags:
  - Prism
  - Windows 8
  - Windows Phone
  - wpdev
---
Unless this is your first experience with Windows Store app development, you should be familiar with the concept of “application lifecycle”. Windows Store apps are created for scenarios where battery life, memory and CPU are not unlimited: consequently, the old desktop approach, where applications are able to keep running in background for an indefinite time, doesn’t fit well this new modern world, where PCs aren’t anymore the only device used to connect to the Internet, work, play, etc.

When a Windows Store isn’t in foreground anymore (because the user has returned to the Start screen, he has launched another application, he has tapped on a toast notification, etc.) it’s suspended after 10 seconds: the process is kept in memory, but all the running operations (network connections, thread, etc.) are stopped. The application will continue to use RAM memory, but it won’t be able to “steal” CPU power, network, etc. to the other applications. This way, every other application will be able to use the same resources of the previously opened ones. However, RAM memory isn’t infinite: consequently, when it’s running low, the operating system is able to terminate the older applications, to free some memory.

However, this termination should be transparent to the user: since he didn’t explicitly close the application (by using the task switcher in Windows Phone or by dragging the application from the top to the bottom in Windows, for example), he expects to find it in the same state he left. This is what we, as developers, call “managing the application’s state”: typically, when the application is suspended, we’re going to save in the local storage (which content is persisted across different usages) all the data that we need to recreate the impression that the app has never been closed (the last opened page, the content of a form, etc.). When the application starts, in case it was terminated due to low memory, we need to load this state and to restore the application in the previous state.

In a typical Universal Windows app, we perform this operation in two steps:

  1. The first one is to use the **OnNavigatedTo()** and **OnNavigatedFrom()** events exposed by the page to save and restore the page’s state.
  2. The second one is to use the various methods offered by the **App** class to manage the application’s lifecycle, like **OnSuspended()** (to save the state) and **OnLaunched()** (to restore the state, in case we detect that the app was terminated by the operating system).

However, Prism offers a simpler way to manage this scenario. Let’s see the details.

### 

### Saving simple data

The simplest scenario is when we need to save plain data, like a text in a **TextBox** or boolean in a **CheckBox**. These kind of properties can be automatically saved by Prism when the application is suspended and restored when it’s activated simply by marking them with the **RestorableState** attribute. Let’s say that you have a page with the following XAML:

<pre class="brush: xml;">&lt;StackPanel Margin="12, 0, 12, 0"&gt;
    &lt;TextBox Text="{Binding Path=Name, Mode=TwoWay" /&gt;
    &lt;TextBox Text="{Binding Path=Surname, Mode=TwoWay}" Margin="0, 0, 0, 20 "/&gt;
&lt;/StackPanel&gt;  
</pre>

We have added two **TextBox** controls, which are in binding (in two way mode) with two properties in the ViewModel, called **Name** and **Surname**. Let’s see how they’re defined:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    private string _name;

    [RestorableState]
    public string Name
    {
        get {return _name;}
        set { SetProperty(ref _name, value); }
    }

    private string _surname;

    [RestorableState]
    public string Surname
    {
        get { return _surname; }
        set { SetProperty(ref _surname, value); }
    }
}

</pre>

You can notice that we’re dealing with two standard properties that, thanks to **SetProperty()** method offered by Prism, are able to notify the View every time their value changes. However, you can also notice that we’ve decorated the public properties with the **[RestorableState]** attribute. This is enough to enable the automatic state management by Prism.

Test the scenario is easy, thanks to the tools provided by Visual Studio: launch the application with the debugger connected and write some texts in the two **TextBox** controls. When the debugging session is running, you’ll find a dropdown menu in the **Debug location** toolbar (if you can’t see it, just right click in an empty space in the top area and enable it) called **Lifecycle Events**. This dropdown provides a list of options to simulate the different application’s state, since some of them aren’t deterministic: termination is one of them, since we don’t know if and when the operating system will terminate our application due to low memory. Choose **Suspend and shutdown** from the menu: the application will be terminated and the debugger disconnected. Now launch again the application: you’ll notice that, despite the fact that the process has been terminated, the two values you’ve inserted in the **TextBox** controls will still be there. If you launch the application from scratch, instead, the two controls will be empty: it’s correct, because in this case the user is launching the application for the first time or after he explicitly closed it, so he doesn’t expect to find it in the previous state.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="4.11" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2014/12/4.11_thumb.png?resize=510%2C155" alt="4.11" width="510" height="155" border="0" data-recalc-dims="1" />](https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2014/12/4.11.png)

If you want to make sure that it’s not a trick, but Prism is really managing the state for you, just try to remove the **[RestorableState]** attributes from one of the two properties: if you simulate again the termination, you’ll notice that only the property which is still marked with the attribute will restore its value, while the other **TextBox** will be empty.

### Saving complex data

Another common scenario is when you have to deal with complex data, like classes that are part of your model. Let’s say, for example, that the **Name** and **Surname** properties we’ve previously seen compose a class named **Person**, with the following definition:

<pre class="brush: csharp;">public class Person
{
    public string Name { get; set; }
    public string Surname { get; set; }
}
</pre>

Now let’s change the XAML of our page in the following way:

<pre class="brush: xml;">&lt;storeApps:VisualStateAwarePage
    x:Class="Prism_StateManagement.Views.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:Prism_StateManagement.Views"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:storeApps="using:Microsoft.Practices.Prism.StoreApps"
    xmlns:mvvm="using:Microsoft.Practices.Prism.Mvvm"
    mc:Ignorable="d"
    mvvm:ViewModelLocator.AutoWireViewModel="True"
    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"&gt;

    &lt;Grid&gt;
        &lt;StackPanel Margin="12, 0, 12, 0"&gt;
            &lt;TextBox Text="{Binding Path=Name, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" /&gt;
            &lt;TextBox Text="{Binding Path=Surname, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" Margin="0, 0, 0, 20 "/&gt;
            &lt;StackPanel Orientation="Horizontal"&gt;
                &lt;TextBlock Text="{Binding Path=LatestPerson.Name}" Style="{StaticResource HeaderTextBlockStyle}" /&gt;    
                &lt;TextBlock Text="{Binding Path=LatestPerson.Surname}" Style="{StaticResource HeaderTextBlockStyle}" Margin="30, 0, 0, 0"  /&gt;
            &lt;/StackPanel&gt;
        &lt;/StackPanel&gt;   
    &lt;/Grid&gt;
    
    &lt;Page.BottomAppBar&gt;
        &lt;CommandBar&gt;
            &lt;CommandBar.PrimaryCommands&gt;
                &lt;AppBarButton Label="Ok" Icon="Accept" Command="{Binding Path=ShowMessageCommand}" /&gt;
            &lt;/CommandBar.PrimaryCommands&gt;
        &lt;/CommandBar&gt;
    &lt;/Page.BottomAppBar&gt;
&lt;/storeApps:VisualStateAwarePage&gt;

</pre>

We’ve added a couple of new **TextBlock** controls, which are in binding with a property in the ViewModel called **LatestPerson**: by using the dot as separator, we access to the **Name** and **Surname** properties of the class. We’ve also added a button in the application bar, which is connected to a command in the ViewModel called **ShowMessageCommand.** Let’s see, now, the new definition of the ViewModel:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    private string _name;

    [RestorableState]
    public string Name
    {
        get {return _name;}
        set { SetProperty(ref _name, value); }
    }

    private string _surname;

    [RestorableState]
    public string Surname
    {
        get { return _surname; }
        set { SetProperty(ref _surname, value); }
    }

    private Person _latestPerson;

    public Person LatestPerson
    {
        get {return _latestPerson;}
        set { SetProperty(ref _latestPerson, value); }
    }

    public MainPageViewModel()
    {
        ShowMessageCommand = new DelegateCommand(() =&gt;
        {
            LatestPerson = new Person
            {
                Name = Name,
                Surname = Surname
            };
        });
    }

    public DelegateCommand ShowMessageCommand { get; private set; }
}

</pre>

We’ve added a new Person property, called **LatestPerson**, which is the one in binding with the new **TextBlock** controls we’ve added in the page. We’ve also defined a new **DelegateCommand**, which name is **ShowMessageCommand**, that is triggered when you press the button in the application bar: the command simply takes the values inserted by the user in the two **TextBox** controls in the page and use them to create a new **Person** object, which is displayed on the page simply by assigning it to the **LatestPerson** property.

Now let’s say that we want to preserve also the value of the **LatestPerson** property so that, when the user restores the app, even if it was terminated, both the **TextBox** controls and the new **TextBlock** ones will hold the previous value. In this case, we can’t simply add the **[RestorableState]** attribute to the **LatestPerson** property, since it’s a complex one. We need to use another approach, thanks to another helper offered by Prism: a class called **SessionStateManager**. Like the **NavigationService**, this class is registered in the Unity container in the **App** class, inside the method **OnInitializeAsync():**

<pre class="brush: csharp;">protected override Task OnInitializeAsync(IActivatedEventArgs args)
{
    // Register MvvmAppBase services with the container so that view models can take dependencies on them
    _container.RegisterInstance&lt;ISessionStateService&gt;(SessionStateService);
    _container.RegisterInstance&lt;INavigationService&gt;(NavigationService);
    // Register any app specific types with the container

    // Set a factory for the ViewModelLocator to use the container to construct view models so their 
    // dependencies get injected by the container
    ViewModelLocationProvider.SetDefaultViewModelFactory((viewModelType) =&gt; _container.Resolve(viewModelType));
    return Task.FromResult&lt;object&gt;(null);
}
</pre>

Thanks to the dependency injection approach, you’ll be able to use the **SessioneStateService** class simply by adding an **ISessionStateService** parameter in the ViewModel’s constructor, like in the following sample:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    private readonly ISessionStateService _sessionStateService;

    public MainPageViewModel(ISessionStateService sessionStateService)
    {
        _sessionStateService = sessionStateService;
    }
}

</pre>

The **SessionStateService** class offers a property called **SessionState**, which type is **Dictionary<string, object>:** you’ll be able to save, inside this collection, any type of complex data that you want to keep in case of termination. Under the hood, the content of the collection will be serialized in the storage. The **SessionStateService** class is very useful because it takes care of automatically saving and restoring its content when the app is suspended and resume: as developers, we’ll have just to take to save inside the **SessionState** collection the data we want to save. That’s it: Prism will take care of saving it when the app is suspended and to restore it in case the app is resumed and it was terminated by the operating system.

Since it’s a standard **Dictonary** collection, working with is really easy. Here is the complete ViewModel:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    private readonly ISessionStateService _sessionStateService;
    private string _name;

    [RestorableState]
    public string Name
    {
        get {return _name;}
        set { SetProperty(ref _name, value); }
    }

    private string _surname;

    [RestorableState]
    public string Surname
    {
        get { return _surname; }
        set { SetProperty(ref _surname, value); }
    }

    private Person _latestPerson;

    public Person LatestPerson
    {
        get {return _latestPerson;}
        set { SetProperty(ref _latestPerson, value); }
    }

    public MainPageViewModel(ISessionStateService sessionStateService)
    {
        _sessionStateService = sessionStateService;
        ShowMessageCommand = new DelegateCommand(() =&gt;
        {
            LatestPerson = new Person
            {
                Name = Name,
                Surname = Surname
            };

            if (sessionStateService.SessionState.ContainsKey("Person"))
            {
                sessionStateService.SessionState.Remove("Person");
            }
            sessionStateService.SessionState.Add("Person", LatestPerson);
        });
    }

    public DelegateCommand ShowMessageCommand { get; private set; }

    public override void OnNavigatedTo(object navigationParameter, NavigationMode navigationMode, Dictionary&lt;string, object&gt; viewModelState)
    {
        base.OnNavigatedTo(navigationParameter, navigationMode, viewModelState);
        if (_sessionStateService.SessionState.ContainsKey("Person"))
        {
            LatestPerson = _sessionStateService.SessionState["Person"] as Person;
        }
    }
}

</pre>

When the **ShowMessageCommand** is executed, other than just assigning a value to the **LatestPerson** property, we save it in the **SessionState** collection, simply by using the **Add()** method. Then, in the **OnNavigatedTo()** method (which we’ve discussed in <a href="http://wp.qmatteoq.com/prism-and-universal-windows-app-navigation/" target="_blank">the previous post</a> and it’s triggered when the user navigates to the current page, also in case of resume), we can check if the **SessionState** contains the value we’ve previously saved, which is identified by the **Person** key. If it exists, we retrieve it and we assign it to the **LatestPerson** property, after performing a cast since the **SessionState** collection contain generic objects.

If you’ll try to execute the application, however, you’ll get an exception when the app is suspended: this happens because **Person** is a custom class, it’s not part of the Windows Runtime, so Prism doesn’t know how to handle it when it comes to save the state by serializing it. We can solve this problem by overriding a method in the **App** class called **OnRegisterKnownTypesForSerialization(),** in which we have to register, in the **SessionStateService**, every custom class we’re going to use in the application, like in the following sample:

<pre class="brush: csharp;">protected override void OnRegisterKnownTypesForSerialization()
{
    base.OnRegisterKnownTypesForSerialization();
    SessionStateService.RegisterKnownType(typeof(Person));
}
</pre>

We simply call the **RegisterKnownType()** method, passing as parameter the class’ type (in this case, **Person**).

That’s all! Now, if you launch the application and, again, by simulating the termination using the **Suspend and shutdown** option in Visual Studio, you’ll notice that both the simple properties (**Name** and **Surname**) and the complex one (**LatestPerson**) we’ll be correctly restored.

### Wrapping up

As usual, you can find the sample project used for this post on GitHub at [https://github.com/qmatteoq/Prism-UniversalSample](https://github.com/qmatteoq/Prism-UniversalSample "https://github.com/qmatteoq/Prism-UniversalSample")

### Index of the posts about Prism and Universal Windows apps

  1. [The basic concepts](http://wp.qmatteoq.com/prism-and-universal-windows-app-the-basic-concepts/ "Prism and Universal Windows app – The basic concepts")
  2. <a href="http://wp.qmatteoq.com/prism-and-universal-windows-apps-binding-and-commands/" target="_blank">Binding and commands</a>
  3. <a href="http://wp.qmatteoq.com/prism-and-universal-windows-apps-advanced-commands" target="_blank">Advanced commands</a>
  4. [Navigation](http://wp.qmatteoq.com/prism-and-universal-windows-app-navigation/)
  5. Managing the application&#8217;s lifecycle
  6. [Messages](http://wp.qmatteoq.com/prism-and-universal-windows-apps-messages/ "Prism and Universal Windows apps – Messages")
  7. [Layout management](http://wp.qmatteoq.com/prism-and-universal-windows-apps-layout-management/ "Prism and Universal Windows apps – Layout management")

Sample project: [https://github.com/qmatteoq/Prism-UniversalSample](https://github.com/qmatteoq/Prism-UniversalSample "https://github.com/qmatteoq/Prism-UniversalSample")