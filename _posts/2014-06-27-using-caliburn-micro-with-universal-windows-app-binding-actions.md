---
id: 6254
title: 'Using Caliburn Micro with Universal Windows app &ndash; Binding and actions'
date: 2014-06-27T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6254
permalink: /using-caliburn-micro-with-universal-windows-app-binding-actions/
categories:
  - Windows 8
  - Windows Phone
tags:
  - Caliburn
  - Windows 8
  - Windows Phone
---
<a href="http://wp.qmatteoq.com/using-caliburn-micro-with-universal-windows-app-the-project-setup/" target="_blank">In the previous post</a> we’ve learned how to properly set up Caliburn Micro in a Universal Windows app. In this post we’re going to recap all the most important naming conventions that can be used to connect our data to the user interface.

### Connecting a property in the ViewModel to a control in the View

The simplest way to connect a property in the ViewModel to a control in the View is using binding. Let’s say you have a ViewModel defined in the following way:

<pre class="brush: xml;">namespace CaliburnDemo.ViewModels
{
    public class MainPageViewModel : PropertyChangedBase
    {
        private string name;
        public string Name
        {
            get { return name; }
            set
            {
                name = value;
                NotifyOfPropertyChange(() =&gt; Name);
            }
        }
    }
}</pre>

You can notice two things:

  * **PropertyChangedBase** is the simplest class offered by Caliburn Micro to support the propagation of properties: this way, every time we change the value of the property in the ViewModel, the control that is placed in binding with the property will update its visual layout to reflect the changes.
  * **NotifyOfPropertyChange()** is the method to call to propagate the change to the UI.

You can connect the property **Name** to the View using the standard binding approach, like in the following sample:

<pre class="brush: xml;">&lt;TextBlock Text="{Binding Name}" /&gt;</pre>

Otherwise, you can use the Caliburn Micro naming conventions, which requires to give to the control (using the **x:Name** property) the same name of the property in the ViewModel you want to connect. In the previous sample, you would have simply defined the **TextBlock** control in the following way:

<pre class="brush: xml;">&lt;TextBlock x:Name="Name" /&gt;</pre>

When you use this naming convention, automatically the property called **Name** in the ViewModel will be bound to the **Text** property of the TextBlock control. In addition, it will be a two-way binding and every change of the source will be reflected to the target and vice versa immediately. This means that if you’re using a **TextBox**, for example, every time the user presses a key the updated string will be sent to the ViewModel.

### Performing an action

Another common scenario while developing a Windows app is performing an action: for example, the user presses a button and you want to perform an operation. In MVVM, this is typically achieved using **commands**, which are a way to define an action without being constrained by using the events exposed by the control. Event handlers, in fact, can be managed only from in code behind: you can’t subscribe, for example, to the **Click** event exposed by a button in a class different than the code behind one (like a ViewModel).

Caliburn Micro offers a simple naming convention to achieve this result: give to the control the same name of the method that is defined in the ViewModel.

Let’s say that you have a **Button** control which is defined in the following way:

<pre class="brush: xml;">&lt;Button Content="Conferma" x:Name="Confirm" /&gt;</pre>

Since the name of the control is **Confirm**, when it’s pressed it will trigger a method with the same name defined in the ViewModel, like in the following sample:

<pre class="brush: xml;">public void Confirm()
{
    Greetings = string.Format("Hello, {0}!", Name);
}</pre>

One of the features offered by the command approach is that you’re able to define if the action can be executed or not; the cool thing is that the control that is connected to the action will automatically change its visual status accordingly. The most common example is a page that acts as a form, that the user can fill: until all the fields are filled, the button to send the form should be disabled.

This goal is achieved by creating a boolean property in the ViewModel, which name should be the same of the method plus the prefix **Can**. In the previous example, to control the method **Confirm()** we need to create a property called **CanConfirm**, which should return true or false, based on the condition we need to manage.

For example, let’s say that if the **Name** property is empty, the action should be disabled. We can achieve this result by defining the **CanConfirm** property in the following way:

<pre class="brush: csharp;">public bool CanConfirm
{
    get { return !string.IsNullOrWhiteSpace(Name); }
}</pre>

There’s just one thing missing: every time the **Name** property changes, we need to evaluate again the **CanConfirm** property. We can achieve this result simply by calling the **NotifyOfPropertyChange()�** also for the **CanConfirm** property every time the value of the **Name** property changes, like in the following sample:

<pre class="brush: csharp;">private string name;
public string Name
{
    get { return name; }
    set
    {
        name = value;
        NotifyOfPropertyChange(() =&gt; Name);
        NotifyOfPropertyChange(() =&gt; CanConfirm);
    }
}</pre>

This way, as soon as the **Name** property is not empty, the **Button** control that is connected to the **Confirm()** method will be enabled; otherwise, it will stay in the disabled state.

One of the best new features in Windows Phone Store apps is that now the Application Bar (managed using the **CommandBar** control) fully supports binding: this means that you can use the same exact naming convention also with the **AppBarButton** control, like in the following sample:

<pre class="brush: xml;">&lt;Page.BottomAppBar&gt;
    &lt;CommandBar&gt;
        &lt;CommandBar.PrimaryCommands&gt;
            &lt;AppBarButton Icon="Accept" Label="Confirm"
                         x:Name="Confirm" /&gt;
        &lt;/CommandBar.PrimaryCommands&gt;
    &lt;/CommandBar&gt;
&lt;/Page.BottomAppBar&gt;</pre>

&nbsp;

Since the name of this button is **Confirm**, the method with the same name defined in the ViewModel will be called when the user will tap it.

### 

### Advanced actions management

The previous naming convention works fine when it comes to manage the standard command connected to the control, like the **Click** event for a **Button**. But what if you need to manage some extra events? Let’s say, for example, that you want to manage the **DoubleTapped** event, which is triggered when the button is tapped twice. In this case, you can use a special Caliburn Micro class called **Message**, which allows to attach a method to any event exposed by the control.

First, you need to add the Caliburn Micro namespace in the Page definition in the XAML:

<pre class="brush: xml;">&lt;Page
    x:Class="MoviesTracker.Views.MainPageView"
    xmlns:viewModels="using:MoviesTracker.ViewModels"
    xmlns:micro="using:Caliburn.Micro"
    Background="{ThemeResource ApplicationPageBackgroundThemeBrush}"&gt;

&lt;/Page&gt;</pre>

Then you can use the attached property **Attach** exposed by the **Message** class, like in the following sample:

<pre class="brush: xml;">&lt;Button Content="Conferma" x:Name="Confirm" micro:Message.Attach="[Event DoubleTapped] = [Action Confirm]" /&gt;</pre>

The **Attach** property requires a value split in two parts, each of it defined inside square brackets. The first one is marked with the keyword **Event** and it requires the name of the event we want to manage (in our case, **DoubleTapped**). The second one, instead, is marked with the keyword **Action** and it’s the name of the method defined in the ViewModel that should be triggered when the event is raised.

One cool thing is that you can pass some special values to name of the action, which can be used to pass a parameter to the method in the ViewModel. Let’s see a real example with the **ListView** control, which is one of the new and powerful Windows Runtime controls to display collection of items:

<pre class="brush: xml;">&lt;ListView ItemTemplate="{StaticResource OnlineMoviesTemplate}"
ItemsSource="{Binding Movies}"
IsItemClickEnabled="True"
SelectionMode="None"
micro:Message.Attach="[Event ItemClick] = [Action GoToDetail($eventArgs)]" /&gt;</pre>

The **ListView** control offers a new way to manage the list that is useful when you don’t really want to provide a way to select items, but you just need to redirect the user to a detail page to see more information about the selected item. You can achieve this result by setting the **SelectionMode** property of the control to **None** and the **IsItemClickEnabled** one to **True**. This way, every time the user will select an item from the list, the selection won’t be triggered, but a specific event called **ItemClick** will be invoked: in the parameter of the event handler you’ll get the item that has been selected.

The previous sample shows you a powerful special value that can be very useful in this scenario: the **GoToDetail()** action is invoked with the parameter **$eventArgs**. This way, the method in the ViewModel will receive the parameter of the event, exactly like if we are managing it in the code behind using an event handler. Here is how the **GoToDetail()** method in the ViewModel looks like:

<pre class="brush: csharp;">public void GoToDetail(ItemClickEventArgs args)
{
    Movie selectedMovie = args.ClickedItem as Movie;
    _navigationService.NavigateToViewModel&lt;DetailMovieViewModel&gt;(selectedMovie.Id);
}</pre>

&nbsp;

The parameter’s type we receive is exactly the same as we would have used the event handler in code behind: in this case, it’s **ItemClickEventArgs**. The sample shows you how it’s easy to use it to retrieve the needed information, in this case the selected item, which is stored inside the **ClickedItem** property. This sample is taken from my Movies Tracker application, so we’re dealing with a collection of movies: this method retrieves the selected movie and redirects the user to the detail page. For the moment, ignore the **NavigationService** usage: we’re going to talk about it in another post.

Another cool thing in Caliburn is that, in the bootstrapper, you can create your own special parameters to use with the **Attach** property. For example, let’s try to add the following line of code in the **Configure()** method of the **App.xaml.cs** file:

<pre class="brush: csharp;">MessageBinder.SpecialValues.Add("$clickeditem", c =&gt; ((ItemClickEventArgs)c.EventArgs).ClickedItem);</pre>

By using the **MessageBinder** class, we are telling to Callburn Micro that we want to create a new special parameter called **$clickeditem**, which will be connected to the **ClickedItem** property of the **ItemClickEventArgs** class. This way, we can change the **ListView** definition in XAML in the following way:

&nbsp;

<pre class="brush: xml;">&lt;ListView ItemTemplate="{StaticResource OnlineMoviesTemplate}"
ItemsSource="{Binding Movies}"
IsItemClickEnabled="True"
SelectionMode="None"
micro:Message.Attach="[Event ItemClick] = [Action GoToDetail($clickeditem)]" /&gt;</pre>

As you can see, we changed the name of the parameter we pass to the action: it’s now called **$clickeditem**, which is the same name we defined in the bootstrapper using the **MessageBinder**.

By applying this change we can simplify the **GoToDetail()** method we’ve defined in the ViewModel:

<pre class="brush: csharp;">public void GoToDetail(Movie movie)
{
    _navigationService.NavigateToViewModel&lt;DetailMovieViewModel&gt;(movie.Id);
}
</pre>

Thanks to the new parameter we’ve created, the method will directly receive the value of the **ClickedItem** property: Caliburn Micro is able to apply the cast for you, so we can simply define the parameter’s type as **Movie** and the framework will take care of converting the **ClickedItem** property in the expected value.

There are also some other special parameters in Caliburn Micro, like **$dataContext** (to get access to the **DataContext** of the control that invoked the action) or **$source** (which is a reference to the control that invoked the action). You can find the list of all the available parameters <a href="https://github.com/Caliburn-Micro/caliburn-micro.github.io/blob/master/documentation/actions.md" target="_blank">in the official documentation</a>.

### Wrapping up

In this post we’ve learned the basic naming conventions and how to manage actions in a Universal Windows app with Caliburn Micro. In the next post we’ll see some advanced scenario which have changed in this new version of Caliburn Micro, like navigation.

In the meanwhile, <a href="http://1drv.ms/1lT2coE" target="_blank">you can play with the sample project.</a>

### Caliburn Micro 2.0 in Universal Windows apps &#8211; The complete series

  1. [The project setup](http://wp.qmatteoq.com/using-caliburn-micro-with-universal-windows-app-the-project-setup/ "Using Caliburn Micro with Universal Windows app – The project setup")
  2. Binding and actions
  3. [Navigation](http://wp.qmatteoq.com/using-caliburn-micro-with-universal-windows-app-navigation/ "Using Caliburn Micro with Universal Windows app – Navigation")

<a href="https://github.com/qmatteoq/CaliburnMicro-UniversalApp-Demo" target="_blank">Samples available on GitHub</a>