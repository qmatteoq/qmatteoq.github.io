---
id: 6315
title: 'Prism and Universal Windows apps &ndash; Binding and commands'
date: 2014-12-15T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6315
permalink: /prism-and-universal-windows-apps-binding-and-commands/
categories:
  - Universal Apps
tags:
  - Prism
  - Windows 8
  - Windows Phone
---
<a href="http://wp.qmatteoq.com/prism-and-universal-windows-app-the-basic-concepts/" target="_blank">In the previous post</a> we’ve seen the basic concepts about using Prism in a Universal Windows app: we’ve seen how to create our first project and how to setup the Prism infrastructure, so that we are able to use all the helpers and facilities offered by the framework. In this post we’ll see how to support the two most important MVVM concepts: binding and commands.

### Binding

Binding is the way that, in MVVM, you use to connect properties that are declared in the ViewModel to controls that are placed in the View. This way, you’ll be able to perform two basic but important operations:

  * Display data in the page: every time you change a property in the ViewModel, the control that is connected to the property will automatically update itself to display the new value.
  * Receive input from the user: thanks to the two way binding, the communication channel between a property and a control can be bidirectional. This way, if the user interacts with a control in the page (for example, he writes something in a TextBox), the value will be stored in the property in the ViewModel.

The binding is based on the **INotifyPropertyChanged** interface, which should be implemented by ViewModels: this way, all the properties can notify to the controls in the View when its value is changed. Prism offers a base class that every ViewModel should implement which, among other things, gives us a quick way to implement this interface. The name of the class is **VisualStateAwarePage**: the name should help you to understand one of the other features of this class, which is a way to support, with visual states, the different layouts that a View can assume (portrait, landscape, snapped in case of a Windows Store app for Windows 8).

To support this class in our ViewModel we need to perform two operations: the first one is to replace the **Page** class from which every page inherits from with the **VisualStateAwarePage** one, which is included in the **Microsoft.Practices.Prism.StoreApps** namespace. Here is how a page of a Universal Windows app developed with Prism looks like:

<pre class="brush: xml;">&lt;storeApps:VisualStateAwarePage
    x:Class="PrismDemo.Views.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:PrismDemo.Views"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:storeApps="using:Microsoft.Practices.Prism.StoreApps"
    xmlns:mvvm="using:Microsoft.Practices.Prism.Mvvm"
    mc:Ignorable="d"
    mvvm:ViewModelLocator.AutoWireViewModel="True"
    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"&gt;

    &lt;!-- content of the page --&gt;

&lt;/storeApps:VisualStateAwarePage&gt;

</pre>

As you can see, the base node it’s not called **Page** anymore, but it’s been replaced by the **VisualStateAwarePage** one**.** 

The second step is to change also the code behind class of the page (the .xaml.cs file), so that it doesn’t inherit anymore from the **Page** class, but from the **VisualStateAwarePage** one. The following code shows a sample code behind definition:

<pre class="brush: csharp;">using Microsoft.Practices.Prism.StoreApps;

namespace PrismDemo.Views
{
    public sealed partial class MainPage : VisualStateAwarePage
    {
        public MainPage()
        {
            this.InitializeComponent();
        }
    }
}

</pre>

Now that you’re properly set up the page, you’re ready to start using binding. The main concept behind binding is that, instead of using plain simple properties (with a standard getter and setter), you’re going to define properties that are able to propagate the changes to the user interface when its value changes. To make use of the mechanism offered by Prism to manage the notification propagation, you’ll need to inherit your ViewModels from the **ViewModel** class. The following sample shows a ViewModel with a simple property, which is defined using the Prism approach:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    private string _name;
    public string Name
    {
        get { return _name; }
        set { SetProperty(ref _name, value); }
    }

}

</pre>

To assign a value to the property we use the **SetProperty()** method offered by Prism, which requires a reference to the private variable that holds its value (in this sample, it’s **_name**) and the value to assign (which is defined by the **value** keyword). Under the hood, the **SetProperty()** method, other than just assigning a value to the property, will take care of notifying the control that is in binding with it that the value is changed, by implementing, under the hood, the **INotifyPropertyChanged** interface. This way, the control in the XAML will update itself to reflect the change. Connecting this property in the ViewModel with a control in the XAML, now, doesn’t require any special knowledge, rather than standard binding. For example, here is how we can display the value of the **Name** property using a **TextBlock** control, by assigning it to the **Text** property using binding:

<pre class="brush: csharp;">&lt;TextBlock Text="{Binding Path=Name}" /&gt;</pre>

Otherwise, if other than just displaying the value we also need to retrieve the input from the user, we can simply apply the **Mode** attribute to the binding and set it to **TwoWay**, like in the following sample:

<pre class="brush: csharp;">&lt;TextBox Text="{Binding Path=Name, Mode=TwoWay" /&gt;
</pre>

This way, when the user will write something in the box, we’ll be able to get the inserted text in the **Name** property of the ViewModel.

### Commands

Commands are the mechanism offered by the XAML to support the user interaction without being forced to use event handlers. Typically, when you write an application without using the MVVM pattern, to manage the user’s interaction with the user interface (for example, he clicks on a button) you subscribe to a event handler: this way, a new method will be generated in code behind, which will contain the code that will be execute when the event is raised.

The downside of this approach is that it creates a strong connection between the View and the code: event handlers can be managed only in code behind, you can’t simply move it to a ViewModel. The solution is to use commands, which are special objects (that implement the **ICommand** interface) that define the operation to perform when the command is executed. The Windows Runtime offers only the basic interface to support commands, which is **ICommand**: a developer, typically, would have to create its own class to implement this interface. However, al the MVVM frameworks usually offer it and Prism makes no exception: in this case, the class is called **DelegateCommand**. Let’s see a sample about how to define a command in a ViewModel:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{

    public MainPageViewModel()
    {
        SayHelloCommand = new DelegateCommand(() =&gt;
        {
            Message = string.Format("Hello {0}", Name);
        });
    }

    public DelegateCommand SayHelloCommand { get; private set; }


    private string _name;

    public string Name
    {
        get { return _name; }
        set
        { SetProperty(ref _name, value); }
    }

    private string _message;

    public string Message
    {
        get { return _message; }
        set { SetProperty(ref _message, value); }
    }
}

</pre>

As you can see, we’ve defined a new property, which type is **DelegateCommand**: in this case, we don’t need to use the **SetProperty()** method to define it, since a command won’t change implementation at runtime, so it will be set only by the ViewModel when it’s instantiated. It’s what we do in the ViewModel constructor (the method called **MainPageViewModel()**): we create a new **DelegateCommand** and we pass, as parameter,� an anonymous method that defines the operation to execute when the command is invoked. In this sample, we display to the user the message “Hello” followed by his name, that we have received in the property called **Name**.

Here is how the View connected to this ViewModel looks like:

<pre class="brush: xml;">&lt;storeApps:VisualStateAwarePage
    x:Class="PrismDemo.Views.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:PrismDemo.Views"
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
            &lt;TextBlock Text="{Binding Path=Message}" Style="{StaticResource HeaderTextBlockStyle}" /&gt;
        &lt;/StackPanel&gt;
    &lt;/Grid&gt;
    
    &lt;storeApps:VisualStateAwarePage.BottomAppBar&gt;
        &lt;CommandBar&gt;
            &lt;CommandBar.PrimaryCommands&gt;
                &lt;AppBarButton Label="confirm" Icon="Accept" Command="{Binding Path=SayHelloCommand}" /&gt;
            &lt;/CommandBar.PrimaryCommands&gt;
        &lt;/CommandBar&gt;
    &lt;/storeApps:VisualStateAwarePage.BottomAppBar&gt;
&lt;/storeApps:VisualStateAwarePage&gt;

</pre>

As you can see, we’ve added a **TextBox** control (which we’re going to use to collect the user’s name), a **TextBlock** control (which will display the hello message) and a **CommandBar**, which contains the button that the user will have to click to see the message. The button, which is an **AppBarButton** control, shows how to use commands: we simply put in binding the **Command** property of the control with the **DelegateCommand**’s object we’ve defined in the ViewModel. Now, every time the user will press the button, the operation we’ve defined in the command will be executed.

### Enabling and disabling commands

One of the most powerful features provided by commands is the automatic support to the state management. With a simple boolean condition, we can control if the command should be enabled or not: the control that is connected to the command will automatically change his appearance to reflect this status. This means that if, in the previous sample, the **SayHelloCommand** is marked as disabled, the **AppBarButton** control will be grayed and the user won’t be able to click on it.

To support this feature, you just need to pass a boolean condition as second parameter when you define the **DelegateCommand** object, like in the following sample:

<pre class="brush: csharp;">public MainPageViewModel()
{
    SayHelloCommand = new DelegateCommand(() =&gt;
    {
        Message = string.Format("Hello {0}", Name);
    }, () =&gt; !string.IsNullOrEmpty(Name));
}
</pre>

As you can see, the second parameter is a boolean condition that checks if the **Name** property contains an empty value or not: this way, in case the user doesn’t type anything in the **TextBox** control, we disable the button, since we are not able to display the hello message.

There’s one last step to do to properly support this feature: the application is not able, automatically, to know when the boolean condition that we’ve specified could have changed, so we need to explicitly tell it. We do this by calling the **RaiseCanExecuteChanged()** method offered by the **DelegateCommand** class, which tells to the command to evaluate again his status. It’s up to us to decide when it’s the best moment to call it, according to the purpose of the command: in our sample, since we want to avoid that the user can press the button if he hasn’t written anything in the box, we’re going to evaluate the status of the command every time the value of the **Name** property changes. Here is how the definition of the **Name** property changes to support our scenario:

<pre class="brush: csharp;">private string _name;

public string Name
{
    get { return _name; }
    set
    {
        SayHelloCommand.RaiseCanExecuteChanged();
        SetProperty(ref _name, value);
    }
}
</pre>

Simply, before calling the **SetProperty()** method to save the value and update the control, we call the **RaiseCanExecuteChanged()** method of the **SayHelloCommand** object.

### Wrapping up

In this post we’ve seen the basic concepts which are important to start writing a Universal Windows app using prism: binding and commands. In the next posts we’ll cover more advanced scenarios, like navigation, state management, advanced commands features, etc. You find this post’s sample on GitHub: [https://github.com/qmatteoq/Prism-UniversalSample](https://github.com/qmatteoq/Prism-UniversalSample "https://github.com/qmatteoq/Prism-UniversalSample")

### Index of the posts about Prism and Universal Windows apps

  1. [The basic concepts](http://wp.qmatteoq.com/prism-and-universal-windows-app-the-basic-concepts/ "Prism and Universal Windows app – The basic concepts")
  2. Binding and commands
  3. <a href="http://wp.qmatteoq.com/prism-and-universal-windows-apps-advanced-commands" target="_blank">Advanced commands</a>
  4. [Navigation](http://wp.qmatteoq.com/prism-and-universal-windows-app-navigation/)
  5. [Managing the application&#8217;s lifecycle](http://wp.qmatteoq.com/prism-and-universal-windows-app-managing-the-applications-lifecycle/ "Prism and Universal Windows app – Managing the application’s lifecycle")
  6. [Messages](http://wp.qmatteoq.com/prism-and-universal-windows-apps-messages/ "Prism and Universal Windows apps – Messages")
  7. [Layout management](http://wp.qmatteoq.com/prism-and-universal-windows-apps-layout-management/ "Prism and Universal Windows apps – Layout management")

Sample project: [https://github.com/qmatteoq/Prism-UniversalSample](https://github.com/qmatteoq/Prism-UniversalSample "https://github.com/qmatteoq/Prism-UniversalSample")