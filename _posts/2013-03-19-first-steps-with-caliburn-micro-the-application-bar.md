---
id: 2222
title: 'First steps with Caliburn Micro in Windows Phone 8 &ndash; The Application Bar'
date: 2013-03-19T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=2222
permalink: /first-steps-with-caliburn-micro-the-application-bar/
categories:
  - Windows Phone
tags:
  - Caliburn
  - MVVM
  - Windows Phone 8
---
The Application Bar is the joy and the pain of every Windows Phone developer that starts to use the Model-View-ViewModel pattern to develop his applications. Sooner or later you’ll have to face this problem: the Application Bar is a special control, that doesn’t belong to the page and that doesn’t inherit from the **FrameworkElement** class. For this reason, binding simply doesn’t work: you can’t use commands to hook to the Tap event and you can’t bind a property of your ViewModel to the Text or IconUri properties.

Since in most of the cases dealing with the Application Bar will simply force you to “break” the MVVM pattern, many talented developers came up with a solution: an Application Bar replacement. There are many implementations out there: two of the bests I’ve found are the <a href="http://cimbalino.org/" target="_blank">Cimbalino Toolkit</a> by Pedro Lamas and <a href="https://github.com/kamranayub/CaliburnBindableAppBar" target="_blank">Caliburn Bindable App Bar</a> by Kamran Ayub. The first toolkit uses an approach based on behaviors, that are applied on the native application bar. We won’t discuss about this toolkit in this post because, even it’s great, it doesn&#8217;t play well with Caliburn: it’s been designed with support for MVVM Light in mind. We’ll focus on the Caliburn Bindable App Bar, which is a totally new control that replaces the standard Application Bar and that supports all the standard Caliburn naming conventions.

Let’s start!

### Add the application bar to a project

The Caliburn Bindable App Bar is available as a NuGet package: simply right click on your project, choose **Manage NuGet packages** and look for the package called **Caliburn.Micro.BindableAppBar.** Once you’ve installed it, you&#8217;ll have to add the following namespace in the XAML to get a reference to the control:

_xmlns:bab=&#8221;clr-namespace:Caliburn.Micro.BindableAppBar;assembly=Caliburn.Micro.BindableAppBar&#8221;_

Once you’ve done it, you can add the real control which is call **BindableAppBar**. But, pay attention! Unlike the real ApplicationBar control (that is placed **outside the main Grid**,&nbsp; because is not part of the page), this control should be placed **inside the Grid,** right before the closing tag (I’m talking about the Grid that, in the standard template, is called **LayoutRoot**). Here is a sample:

<pre class="brush: xml;">&lt;Grid x:Name="LayoutRoot" Background="Transparent"&gt;
    &lt;Grid.RowDefinitions&gt;
        &lt;RowDefinition Height="Auto"/&gt;
        &lt;RowDefinition Height="*"/&gt;
    &lt;/Grid.RowDefinitions&gt;

    &lt;!--TitlePanel contains the name of the application and page title--&gt;
    &lt;StackPanel x:Name="TitlePanel" Grid.Row="0" Margin="12,17,0,28"&gt;
        &lt;TextBlock Text="Caliburn Micro" Style="{StaticResource PhoneTextNormalStyle}" Margin="12,0"/&gt;
        &lt;TextBlock Text="Sample" Margin="9,-7,0,0" Style="{StaticResource PhoneTextTitle1Style}"/&gt;
    &lt;/StackPanel&gt;

    &lt;!--ContentPanel - place additional content here--&gt;
    &lt;Grid x:Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0"&gt;

    &lt;/Grid&gt;

    &lt;bab:BindableAppBar x:Name="AppBar"&gt;
        &lt;bab:BindableAppBarButton x:Name="AddItem"
                                  Text="{Binding AddItemText}"
                                  IconUri="{Binding Icon}" 
                                  Visibility="{Binding IsVisible, Converter={StaticResource BooleanToVisibilityConverter}}"
                                  /&gt;

        &lt;bab:BindableAppBarMenuItem x:Name="RemoveItem"
                                  Text="Remove"
                                  /&gt;
    &lt;/bab:BindableAppBar&gt;
&lt;/Grid&gt;</pre>

The control is very simple to use! Inside the **BindableAppBar** node you can add two items: **BindableAppBarButton**, which is the icon button (remember that you can add up to four icons) and **BindableAppBarMenuItem**, which is one of the text items that are displayed under the icons.

They share the same properties, because they both are buttons that can be tapped to trigger an action: the only difference is that the **BindableAppBarButton** control has an **IconUri** property, which contains the path of the image that is displayed as icon.

Both controls share the same Caliburn naming convention that is used for actions: the value of the **x:Name** property of the control is the name of the method, declared in the ViewModel of the page, that is triggered when the user taps on it. The best part of this control that all the other properties supports binding, even the **Visibility** property, that can be used to show or hide an item according to some conditions.

Before using it, there’s an important step to do: add a custom convention. Caliburn Micro supports a way to define your own custom conventions, that are added at the top of the already existing one. The place where to do this is in the **boostrapper**, inside the **AddCustomConventions()** method that, by default, is called when the boostrapper is registered.

Here is the code to insert:

<pre class="brush: csharp;">static void AddCustomConventions()
{
    ConventionManager.AddElementConvention&lt;BindableAppBarButton&gt;(
    Control.IsEnabledProperty, "DataContext", "Click");
    ConventionManager.AddElementConvention&lt;BindableAppBarMenuItem&gt;(
    Control.IsEnabledProperty, "DataContext", "Click");
}</pre>

With this code basically we’re adding a convention to manage the **Click** event on the button, so that it’s enough to give to a method the same name of the Button control to bind them together.

Now it’s the ViewModel’s turn to manage the **BindableAppBar:**

<pre class="brush: csharp;">public class MainPageViewModel: Screen
{

    private string addItemText;

    public string AddItemText
    {
        get { return addItemText; }
        set
        {
            addItemText = value;
            NotifyOfPropertyChange(() =&gt; AddItemText);
        }
    }

    private Uri icon;

    public Uri Icon
    {
        get { return icon; }
        set
        {
            icon = value;
            NotifyOfPropertyChange(() =&gt; Icon);
        }
    }

    private bool isVisible;

    public bool IsVisible
    {
        get { return isVisible; }
        set
        {
            isVisible = value;
            NotifyOfPropertyChange(() =&gt; IsVisible);
        }
    }

    public MainPageViewModel()
    {
        AddItemText = "Add";
        Icon = new Uri("/Assets/AppBar/appbar.add.rest.png", UriKind.Relative);
        IsVisible = false;  
    }

    public void AddItem()
    {
        MessageBox.Show("Item added");
    }

    public void RemoveItem()
    {
        MessageBox.Show("Item removed");
    }
}</pre>

Nothing special to say if you’ve already read the other posts about Caliburn Micro: we have defined some properties and methods, that are connected to the Application Bar using the Caliburn naming conventions. When the ViewModel is created, we set the text, the icon and the visibility status of one of the buttons in the Application Bar, instead of defining them in the XAML. This approach is very useful when the state of the buttons in the Application Bar needs to change while the application is executed. For example, think about an application to read news: the user is able to save a news in the favorites&#8217; list using a button in the Application Bar. In this case, the status of the button should change according to the status of the news: if the news has already been marked as read, probably the text of the button will be something like “Remove” and the icon will display a minus sign; vice versa, if the news hasn’t been added yet to the list the button’s text will say “Add” and the icon will display a plus sign.

With the ViewModel we’ve defined, it’s simple to change some properties according to the status of the news and automatically reflect the change to the control in the XAML.

Also the **Visibility** property can come in handy in many situations: for example, let’s say that the same news application as before allows the user to pin a news in the start screen, by creating a secondary tile. In this case, only if the application is opened from a secondary tile we want to display a “Home” button in the Application Bar, to redirect him to the home page of the app; otherwise, we don’t need it, because the user can use the Back button to accomplish the same task. In this scenario, the **Visibility** property is perfect for us: it’s enough to change it according to the fact that the app has been opened from a secondary tile or not.

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:67a4e7b0-af52-4c05-a8c3-a3542dcf12d9" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/07/CaliburnMicro_AppBar.zip" target="_blank">Download the sample project</a>
  </p>
</div>

&nbsp;

**The Caliburn Micro posts series**

  1. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-theory/" target="_blank">The theory</a> 
      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-first-project/" target="_blank">The first project</a> 
          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-actions/" target="_blank">Actions</a> 
              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">Collections and navigation</a> 
                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-tombstoning/" target="_blank">Tombstoning</a> 
                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-advanced-navigation-and-deep-links/" target="_blank">Advanced navigation and deep links</a> 
                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-messaging/" target="_blank">Messaging</a> 
                              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-using-launchers-and-choosers/" target="_blank">Using launchers and choosers</a> 
                                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-use-your-own-services-and-how-to-pass-data-between-different-pages/" target="_blank">Use your own services</a> 
                                      * The Application Bar 
                                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-pivot/" target="_blank">Pivot</a> 
                                              * [Lazy loading with pivot](http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-lazy-loading-with-pivot/) </ol>