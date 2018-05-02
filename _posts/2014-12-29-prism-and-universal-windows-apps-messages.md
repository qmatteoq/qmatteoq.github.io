---
id: 6330
title: 'Prism and Universal Windows apps &ndash; Messages'
date: 2014-12-29T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6330
permalink: /prism-and-universal-windows-apps-messages/
categories:
  - Universal Apps
  - wpdev
tags:
  - Prism
  - Windows 8
  - Windows Phone
---
Another common scenario when you work with the MVVM pattern is messages support: all the available toolkits and frameworks support them. Messages are a way to exchange data between two classes (typically, two ViewModels) with a decoupled approach: the two classes won’t have to know each other and you won’t need a common reference. One ViewModel will simply send a message, then other ViewModels can register to receive that message’s type. Messages are, at the end, simple classes, which can hold some data that can be passed from one ViewModel to another.

In Prism, messages are named **events**: when a ViewModel needs to send some data to another one, it publishes an event; when a ViewModel wants to receive that data, it subscribes for that event. The “postman” that takes care of managing all the infrastructure and of dispatching the events is a class called **EventAggregator**, which is not included in the base Prism package: you’ll have to install from NuGet a specific package called <a href="http://www.nuget.org/packages/Prism.PubSubEvents/" target="_blank">Prism.PubSubEvents</a>.

After you’ve installed it in both in the Windows and the Windows Phone project, you’ll have to register the **EventAggregator** object in the **App** class, in the same way we did in the previous posts for the **NavigationService** and the **SessionStateService** classes: this way, we’ll be able to use the **EventAggregator** class simply by adding an **IEventAggregator** parameter to the ViewModel’s constructor, thanks to the dependency injection. We register it in the **OnInitializeAsync()** method of the **App** class, like in the following sample:

<pre class="brush: csharp;">protected override Task OnInitializeAsync(IActivatedEventArgs args)
{
    // Register MvvmAppBase services with the container so that view models can take dependencies on them
    _container.RegisterInstance&lt;ISessionStateService&gt;(SessionStateService);
    _container.RegisterInstance&lt;INavigationService&gt;(NavigationService);
    _container.RegisterInstance&lt;IEventAggregator&gt;(new EventAggregator());
    // Register any app specific types with the container

    // Set a factory for the ViewModelLocator to use the container to construct view models so their 
    // dependencies get injected by the container
    ViewModelLocationProvider.SetDefaultViewModelFactory((viewModelType) =&gt; _container.Resolve(viewModelType));
    return Task.FromResult&lt;object&gt;(null);
}
</pre>

To see how to use events in Prism, we’re going to implement a simple application to display a list of persons: the main page will contain a **ListView** control, that will display a list of **Person** objects. The page will contains also a button, in the application bar, to go to an insert page, with a simple form to add a new person to the collection. After the user has filled the name and surname of the user and he has pressed the Save button, we’re going to add the new item in the collection in the main page. We’re going to achieve this goal by using events: when the user presses the Save button, we’re going to send a message to the ViewModel of the main page with the just created person; in the ViewModel of the main page, instead, we’re going to subscribe to this event: when it’s triggered, we’re going to retrieve the new person and add it to the collection displayed in the page.

The first step to implement this scenario is to create a class that identifies our event, like in the following sample:

<pre class="brush: csharp;">public class AddPersonEvent : PubSubEvent&lt;Person&gt;
{
}
</pre>

As you can see, the class is very simple, since it doesn’t contain any property or constructor: the only requirement is to inherit it from the **PubSubEvent<T>** class, where **T** is the type of the object we want to pass inside the message. In this sample, we’re going to pass a **Person** object.

The next step is to define the sender and the receiver of the message: in our case, the sender will be the ViewModel of the add page, while the receiver will be the ViewModel of the main page. Let’s start to see the ViewModel of the main page:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    private readonly INavigationService _navigationService;
    private readonly IEventAggregator _eventAggregator;

    private ObservableCollection&lt;Person&gt; _persons;

    public ObservableCollection&lt;Person&gt; Persons
    {
        get {return _persons;}
        set { SetProperty(ref _persons, value); }
    }  

    public MainPageViewModel(INavigationService navigationService, IEventAggregator eventAggregator)
    {
        _navigationService = navigationService;
        _eventAggregator = eventAggregator;

        _eventAggregator.GetEvent&lt;AddPersonEvent&gt;().Subscribe(person =&gt;
        {
            if (Persons == null)
            {
                Persons = new ObservableCollection&lt;Person&gt;();
            }

            Persons.Add(person);
        }, ThreadOption.UIThread);

        GoToAddPageCommand = new DelegateCommand(() =&gt;
        {
            _navigationService.Navigate("Add", null);
        });
    }

    public DelegateCommand GoToAddPageCommand { get; private set; }
}


</pre>

You can notice that, other than an **INavigationService** parameter (which we already met <a href="http://wp.qmatteoq.com/prism-and-universal-windows-app-navigation/" target="_blank">in another post</a>), we have added a reference to the **IEventAggregator** class, which we’re going to use to send and receive events. In this case, since we’re in the receiver ViewModel, we’re going to subscribe to the event we’ve previously defined: we do it in the ViewModel’s constructor. The first step is to get a reference to the event we want to manage, by using the **GetEvent<T>** method, where **T** is the event’s type (in our case, it’s the **AddPersonEvent** class we’ve previously created). Then, since in this case we want to receive it, we call the **Subscribe()** message, which accepts the action that we want to execute when the event is triggered. As action’s parameter, we get the content of the message (in our case, it’s a **Person** object): in the sample, we simply add the **Person** object we’ve received to a collection called **Persons**, which is connected to a **ListView** control in the page. Optionally, we can pass a second parameter to the **Subscribe()** method to specify in which thread we want to manage the event: in this case, since we’re updating a control in the View, we manage it in the UI thread (**ThreadOption.UIThread**). Otherwise, we could have used **ThreadOption.BackgroundThread** to manage it in background: this approach is useful if we need to execute CPU consuming operations that don’t need to interact with the View.

The ViewModel defines also a **DelegateCommand**, which simply redirects the user to the **Add** page that simply contains a couple of **TextBox** controls and a button to save the data. Here is its definition:

<pre class="brush: xml;">&lt;storeApps:VisualStateAwarePage
    x:Class="Prism_Messages.Views.AddPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:Prism_Messages.Views"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:mvvm="using:Microsoft.Practices.Prism.Mvvm"
    xmlns:storeApps="using:Microsoft.Practices.Prism.StoreApps"
    mc:Ignorable="d"
    mvvm:ViewModelLocator.AutoWireViewModel="True"
    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"&gt;

    &lt;Grid&gt;
        &lt;StackPanel Margin="12, 0, 0, 12"&gt;
            &lt;TextBox PlaceholderText="Name" Header="Name" Text="{Binding Path=Name, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" /&gt;
            &lt;TextBox PlaceholderText="Surname" Header="Surname" Text="{Binding Path=Surname, Mode=TwoWay, UpdateSourceTrigger=PropertyChanged}" /&gt;
        &lt;/StackPanel&gt;
    &lt;/Grid&gt;

    &lt;Page.BottomAppBar&gt;
        &lt;CommandBar&gt;
            &lt;CommandBar.PrimaryCommands&gt;
                &lt;AppBarButton Label="Save" Icon="Save" Command="{Binding Path=SaveCommand}" /&gt;
            &lt;/CommandBar.PrimaryCommands&gt;
        &lt;/CommandBar&gt;
    &lt;/Page.BottomAppBar&gt;
&lt;/storeApps:VisualStateAwarePage&gt;

</pre>

Let’s see now the most interesting part, which is the ViewModel of the Add page:

<pre class="brush: csharp;">public class AddPageViewModel : ViewModel
{
    private readonly INavigationService _navigationService;
    private readonly IEventAggregator _eventAggregator;

    private string _name;

    public string Name
    {
        get { return _name; }
        set { SetProperty(ref _name, value); }
    }

    private string _surname;

    public string Surname
    {
        get { return _surname; }
        set { SetProperty(ref _surname, value); }
    }

    public AddPageViewModel(INavigationService navigationService, IEventAggregator eventAggregator)
    {
        _navigationService = navigationService;
        _eventAggregator = eventAggregator;

        SaveCommand = new DelegateCommand(() =&gt;
        {
            Person person = new Person
            {
                Name = this.Name,
                Surname = this.Surname
            };

            _eventAggregator.GetEvent&lt;AddPersonEvent&gt;().Publish(person);
            _navigationService.GoBack();
        });
    }


    public DelegateCommand SaveCommand { get; private set; }
}

</pre>

In this sample we see the other usage of the **EventAggregator** class, which is publishing an event: we use it in the **SaveCommand**, which is triggered when the user presses the Save button in the page. The first step, also in this case, is to get a reference to the event, by calling the **GetEvent<T>()** method. However, in this situation, we’re going to use the **Publish()** method, which sends the message: as parameter, we need to pass the data that is supported by the event (in our case, the **AddPersonEvent** supports a **Person**’s parameter). In the end, we call the **GoBack()** method of the **NavigationService**, to redirect the user back to the main page.

If we launch the application, we’ll notice that, after pressing the Save button in the Add page, the user will be redirected to the main page and the just created **Person** object will be displayed in the list. If we set some breakpoints in the **MainPageViewModel** and in the **AddPageViewModel** classes, we’ll notice that the messages are successfully exchanged between the two ViewModels.

### Wrapping up

As usual, you can download the sample project used in this post on GitHub at [https://github.com/qmatteoq/Prism-UniversalSample](https://github.com/qmatteoq/Prism-UniversalSample "https://github.com/qmatteoq/Prism-UniversalSample")

### Index of the posts about Prism and Universal Windows apps

  1. [The basic concepts](http://wp.qmatteoq.com/prism-and-universal-windows-app-the-basic-concepts/ "Prism and Universal Windows app – The basic concepts")
  2. <a href="http://wp.qmatteoq.com/prism-and-universal-windows-apps-binding-and-commands/" target="_blank">Binding and commands</a>
  3. <a href="http://wp.qmatteoq.com/prism-and-universal-windows-apps-advanced-commands" target="_blank">Advanced commands</a>
  4. [Navigation](http://wp.qmatteoq.com/prism-and-universal-windows-app-navigation/)
  5. [Managing the application&#8217;s lifecycle](http://wp.qmatteoq.com/prism-and-universal-windows-app-managing-the-applications-lifecycle/ "Prism and Universal Windows app – Managing the application’s lifecycle")
  6. Messages
  7. [Layout management](http://wp.qmatteoq.com/prism-and-universal-windows-apps-layout-management/ "Prism and Universal Windows apps – Layout management")

Sample project: [https://github.com/qmatteoq/Prism-UniversalSample](https://github.com/qmatteoq/Prism-UniversalSample "https://github.com/qmatteoq/Prism-UniversalSample")