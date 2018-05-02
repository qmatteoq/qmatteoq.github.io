---
id: 6318
title: 'Prism and Universal Windows apps &ndash; Advanced commands'
date: 2014-12-17T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6318
permalink: /prism-and-universal-windows-apps-advanced-commands/
categories:
  - Universal Apps
tags:
  - Prism
  - Windows 8
  - Windows Phone
  - wpdev
---
In the <a href="http://wp.qmatteoq.com/prism-and-universal-windows-apps-binding-and-commands/" target="_blank">previous post</a> we’ve learned the basic concepts that are required to implement the MVVM pattern using Prism in a Universal Windows app. In this post, we’ll continue our journey by exploring more details about using commands.

### Managing secondary events with commands

When we use the **Command** property provided by the XAML control, we define the operation that should be executed when the user interacts with the primary event exposed by the control. For example, if it’s a **Button**, the **Command** property is connected to the **Click** event, so the operation is performed when the user taps on the button. However, in some situation we may have the requirement to subscribe to additional events that are exposed by the control. Let’s say, for example, that we want to manage the **DoubleTapped** event exposed by the **Button** control, which is triggered when you perform a double tap (instead of a single tap) on the button.

The goal can be achieved thanks to the behaviors provided by the **Microsoft.Xaml.Interactivity** library, which can be added to your project by right clicking on it, choosing **Add reference** and, in the **Extensions** section, enabling the **Behaviors SDK (XAML)** option. Now you’ll be able to use some behaviors that can be used to connect a secondary event to a command in the ViewModel. Here is a sample in the XAML page:

<pre class="brush: xml;">&lt;storeApps:VisualStateAwarePage
    x:Class="Prism_AdvancedDemo.Views.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:Prism_AdvancedDemo"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:storeApps="using:Microsoft.Practices.Prism.StoreApps"
    xmlns:mvvm="using:Microsoft.Practices.Prism.Mvvm"
    xmlns:interactivity="using:Microsoft.Xaml.Interactivity"
    xmlns:core="using:Microsoft.Xaml.Interactions.Core"
    mc:Ignorable="d"
    mvvm:ViewModelLocator.AutoWireViewModel="True"
    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"&gt;

    &lt;Grid&gt;
        &lt;Button Content="Double tap me"&gt;
            &lt;interactivity:Interaction.Behaviors&gt;
                &lt;core:EventTriggerBehavior EventName="DoubleTapped"&gt;
                    &lt;core:InvokeCommandAction Command="{Binding Path=DoubleTapCommand}" /&gt;
                &lt;/core:EventTriggerBehavior&gt;
            &lt;/interactivity:Interaction.Behaviors&gt;
        &lt;/Button&gt;
    &lt;/Grid&gt;
&lt;/storeApps:VisualStateAwarePage&gt;
</pre>

As you can, we’ve added two new namespaces to the XAML page definition: one is **Microsoft.Xaml.Interactivity**, the other one is **Microsoft.Xaml.Interactions.Core.** Thanks to these namespaces we are able to define, by using the attached property **Interaction.Behaviors**, the list of behaviors that we want to apply to the control (in our case, it’s the **Button** one). The behavior we’re going to use is called **EventTriggerBehavior**, which requires, by using the **EventName** property, the name of the event exposed by the control that we want to connect to it: in this case, it’s the **DoubleTapped** one. Inside the behavior we need to specify which action we want to perform: in this case, we need to use the **InvokeCommandAction** trigger, which is used to invoke an **ICommand** object defined in the ViewModel.

From the ViewModel point of view, nothing changes when it comes to define the **DoubleTapCommand:** it’s a standard implementation of the **DelegateCommand** class, as we’ve seen in the previous post.

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    public MainPageViewModel()
    {
        DoubleTapCommand = new DelegateCommand(async () =&gt;
        {
            MessageDialog dialog = new MessageDialog("Double tap performed");
            await dialog.ShowAsync();
        });
    }


    public DelegateCommand DoubleTapCommand { get; private set; }
}

</pre>

The command simply displays a message to the user, using a **MessageDialog** popup, every time it’s invoked.

### Commands with parameters

Commands also support a way to pass parameters to the method that is invoked when the command is invoked. From the XAML point of view, it’s enough to set the parameter using the **CommandParameter** property offered by every control that support the **Command** property. The following sample shows how to set a parameter with a **Button** control:

<pre class="brush: xml;">&lt;Button Content="Click me" Command="{Binding Path=DisplayMessageCommand}" CommandParameter="Text of the message" /&gt;
</pre>

This approach is not a specific Prism feature, but it’s built-in in the XAML frameworks. What Prism offers is a a different implementation of the **DelegateCommand** class, which supports a parameter using generics. The following sample shows how to declare the **DisplayMessageCommand** so that it can receive a **string** parameter (since the value passed to the **CommandParameter** property is a text):

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    public MainPageViewModel()
    {         
        DisplayMessageCommand = new DelegateCommand&lt;string&gt;(async (args) =&gt;
        {
            MessageDialog dialog = new MessageDialog(args);
            await dialog.ShowAsync();
        });
    }

   
    public DelegateCommand&lt;string&gt; DisplayMessageCommand { get; private set; }

}

</pre>

There are two differences compared to a standard **DelegateCommand** implementation:

  * We’re using the **DelegateCommand<T>** type, where **T** is the parameter’s type (in our sample, it’s **string**).
  * The anonymous method that contains the code that is executed when the command is invoked contains a parameter, which is the value received by the **CommandParameter** property. In the sample, it’s called **args** and we use it to display to the user, using a **MessageDialog**, the parameter’s value.

### Passing as parameter the event handler’s argument

When you work in code behind and you subscribe to an event handler, typically you get a parameter that contains some important information about the event. Let’s use, as sample, a **ListView** control, which is used to display a collection of items. One way to interact with the list (that is often used when you just need to know which item has been chosen, for example in a master – detail scenario) is to set the **IsItemClickEnabled** property to **True**: this way you can simply subscribe to the **ItemClick** event to get which item has been selected. Here is a sample usage of this event in the code behind:

<pre class="brush: csharp;">private void NewsList_OnItemClick(object sender, ItemClickEventArgs e)
{
    News news = e.ClickedItem as News;
}</pre>

As you can see, the event handler contains a parameter (which type is **ItemClickEventArgs**) which stores, inside the **ClickedItem** property, the item that the user has selected in the **ListView**. In the previous sample, we’re displaying a collection of **News** objects so, thanks to a cast, we’re converting the selected item (which is a generic **object**) into the proper type.

This approach works fine in code behind, but what about MVVM? Since we have a command and not an event handler, how we can get, in the ViewModel, the event arguments received by the method (in this case, the **ItemClickEventArgs** object)? Prism makes things simple: it’s enough to create a **DelegateCommand<T>** property in the ViewModel, where **T** is the event argument’s type. Prism will automatically take care of passing, to the command, the event arguments of the event we’ve subscribed. Let’s consider the following sample:

<pre class="brush: xml;">&lt;ListView ItemsSource="{Binding Path=News}" SelectionMode="Single" IsItemClickEnabled="True"&gt;
    &lt;ListView.ItemTemplate&gt;
        &lt;DataTemplate&gt;
            &lt;StackPanel&gt;
                &lt;TextBlock Text="{Binding Title}" Style="{StaticResource SubheaderTextBlockStyle}" 
           TextWrapping="Wrap" /&gt;
            &lt;/StackPanel&gt;
        &lt;/DataTemplate&gt;
    &lt;/ListView.ItemTemplate&gt;
    &lt;interactivity:Interaction.Behaviors&gt;
        &lt;core:EventTriggerBehavior EventName="ItemClick"&gt;
            &lt;core:InvokeCommandAction Command="{Binding Path=ShowDetailCommand}"
                      /&gt;
        &lt;/core:EventTriggerBehavior&gt;
    &lt;/interactivity:Interaction.Behaviors&gt;
&lt;/ListView&gt;
</pre>

We’re using the same approach we’ve seen before with the **Button** control: we’ve added an **EventTriggerBehavior** to the **ListView** control, which is connected to the **ItemClick** event. Then we applied an **InvokeCommandAction**, which invokes a command called **ShowDetailCommand**. As you can see, we’re not specyfing any command parameter. However, we are able to create a **DelegateCommand** object in the ViewModel that looks like this:

<pre class="brush: csharp;">public class MainPageViewModel : ViewModel
{
    public MainPageViewModel()
    {
        ShowDetailCommand = new DelegateCommand&lt;ItemClickEventArgs&gt;((args) =&gt;
        {
            News news = args.ClickedItem as News;
        });
    }

    public DelegateCommand&lt;ItemClickEventArgs&gt; ShowDetailCommand { get; private set; }
}

</pre>

As you can see, we’ve not created a plain **DelegateCommand** object, but a **DelegateCommand<T>** one, where **T** is the event argument’s type we receive in the event handler (in our case, it’s **ItemClickEventArgs**). Prism will automatically inject, into the method, the event handler’s parameter: when we create the **DelegateCommand<ItemClickEventArgs>** object, we are able to access to this parameter and to perform the same operation we did in the code behind (which is accessing to the **ClickedItem** property and casting it to a **News** object). This way we would be able, for example, to use the Prism’s navigation service (which we’ll cover in the next post) to redirect the user to detail page, passing as parameter the selected item.

### Wrapping up

As usual, you can find the sample code used for this post in the GitHub’s project available at the URL [https://github.com/qmatteoq/Prism-UniversalSample](https://github.com/qmatteoq/Prism-UniversalSample "https://github.com/qmatteoq/Prism-UniversalSample")

### Index of the posts about Prism and Universal Windows apps

  1. [The basic concepts](http://wp.qmatteoq.com/prism-and-universal-windows-app-the-basic-concepts/ "Prism and Universal Windows app – The basic concepts")
  2. <a href="http://wp.qmatteoq.com/prism-and-universal-windows-apps-binding-and-commands/" target="_blank">Binding and commands</a>
  3. Advanced commands
  4. [Navigation](http://wp.qmatteoq.com/prism-and-universal-windows-app-navigation/)
  5. [Managing the application&#8217;s lifecycle](http://wp.qmatteoq.com/prism-and-universal-windows-app-managing-the-applications-lifecycle/ "Prism and Universal Windows app – Managing the application’s lifecycle")
  6. [Messages](http://wp.qmatteoq.com/prism-and-universal-windows-apps-messages/ "Prism and Universal Windows apps – Messages")
  7. [Layout management](http://wp.qmatteoq.com/prism-and-universal-windows-apps-layout-management/ "Prism and Universal Windows apps – Layout management")

Sample project: [https://github.com/qmatteoq/Prism-UniversalSample](https://github.com/qmatteoq/Prism-UniversalSample "https://github.com/qmatteoq/Prism-UniversalSample")