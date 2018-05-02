---
id: 2322
title: 'First steps with Caliburn Micro in Windows Phone 8 &ndash; Pivot'
date: 2013-03-25T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=2322
permalink: /first-steps-with-caliburn-micro-in-windows-phone-8-pivot/
categories:
  - Windows Phone
tags:
  - Caliburn
  - MVVM
  - Windows Phone
---
Panorama and pivot are two key concepts in Windows Phone development: they are, probably, the most used UI paradigm in applications, since they allow to create interfaces that are very different (and, for this reason, original) from an iOS or Android applications.

Panoramas are used, usually, as an entry point for the application: it features many pages, that can be browsed by the user simply by swiping on the left or on the right of the screen; every page shows a little preview of the content of the next page, so the user can understand that there’s more to see. They are a starting point for the app, not a data container: a panorama should be used to give a quick peek of what the application offers and a way to access to all the different sections and options. For example, if you’re developing a news reader, it’s not correct to display all the news in a panorama page; it’s correct, instead, to display just the latest 10 news and add a button to go to another section of the application to see all the news.

Pivots, instead, are used to display different information related to the same context, or the same information but related to different items. As example of the first scenario, take the contact’s detail page in the People hub: in this case, the Pivot control is used to display different information (details, pictures, social network’s updates, etc.) related to the same context (the person). A weather app, instead, is a good example of the second scenario: a Pivot item can be used to display the same information (the weather forecast) but for different items (the cities).

Managing a Panorama or a Pivot in a MVVM application isn’t trivial: since the pages that are part of a Panorama or a Pivot are “fake” (they aren’t real different XAML pages, but different item controls, **PivotItem** or **PanoramaItem**, inside the main one, **Panorama** or **Pivot**, all placed in the same page), it’s enough to have a View and a ViewModel and connect them (and the data exposed by the ViewModel) using the Caliburn Micro conventions we’ve learned to use in the previous post.

But there’s a smarter approach to that, which can be used to support features like lazy loading and to have a better organized code. Let’s see what it’s all about.

**IMPORTANT!** Even if, from a code point of view, Panorama and Pivot have the same approach, we’ll be able to implement the mechanism I&#8217;m going to talk about just using a Pivot, due to a bug in the Panorama control in Windows Phone 8. Which is the problem? That if Panorama items are added to the Panorama control using binding and the **ItemsSource** property (and they aren’t directly declared in the XAML or manually added in the code behind), the **SelectedIndex** property (which is important to keep track of the view that is actually visible) doesn’t work properly and returns only the values 0 and 1. The consequence is that the **SelectedItem** property doesn’t work properly, so we are not able to know exactly when a view is displayed.

### The Conductor class

Caliburn Micro supports the concept of **Conductor:** a series of pages that are connected together and that are managed by a single entry point. With this approach, we’re able to have a main page, which acts as conductor, and different pages with different view models, that are the items of the Pivot or the Panorama. The first advantage of this approach should be already clear: instead of having a unique ViewModel and a unique View, that should manage all the items inside the control, we have separate Views and separate ViewModels: this will help us a lot to keep the project cleaner and easier to maintain.

Let’s start to do some experiments using a Pivot. First we need to create the **Conductor** page, that will contain the Pivot control: let’s add a page in the **Views** folder of the project (for example, **PivotView**) and a ViewModel in the **ViewModels** folder (using the standard naming convention, it should be **PivotViewModel**). Don’t forget to register it in the **Configure()** method of the bootstrapper!

Now let’s start with the ViewModel definition:

<pre class="brush: csharp;">public class PivotViewModel: Conductor&lt;IScreen&gt;.Collection.OneActive
{
    private readonly PivotItem1ViewModel item1;
    private readonly PivotItem2ViewModel item2;

    public PivotViewModel(PivotItem1ViewModel item1, PivotItem2ViewModel item2)
    {
        this.item1 = item1;
        this.item2 = item2;
    }

    protected override void OnInitialize()
    {
        base.OnInitialize();

        Items.Add(item1);
        Items.Add(item2);

        ActivateItem(item1);
    }
}</pre>

First, the ViewModel needs to inherit from the class **Conductor<T>.Collection.OneActive:** we’re telling to the ViewModel that it will hold a collection of views (since T is **IScreen**, which is the base interface for the **Screen** class) and, with the **OneActive** syntax, we’re telling that we are in scenario where only one view can be active at the same time. This is the only option that can be used in Windows Phone to manage Panorama and Pivot controls: Caliburn Micro offers other options because it supports also other technologies like WPF or Silverlight, where multiple views can be active at the same tine.

Notice that, in the constructor, we have added two parameters, which types are **PivotItem1ViewModel** and **PivotItem2ViewModel**: these are the ViewModels that are connected to the pages that we’re going to display in the Pivot control and that we’re going to create later.

Then we override the **OnInitialize()** method, that is called when the page is initialized for the first time: since we’re inheriting from the **Conductor<T>** class, we have access to two important helpers: the **Items** property and the **ActivateItem**() method. The first one is a collection of elements which type is T (the one that has been passed to the **Conductor<T>** definition): it will contains all the pages of our Pivot control, so we simply add all the ****ViewModels we’ve added in the constructor. Then we call the **ActivateItem()** method, that focus the view on the specified page: in this case we’re setting the focus on the first one, but we could have done the same on another page. For example, we could have received the page to display as a query string parameter in the navigation url, from another page or a secondary tile.

Now we need to create the other pages, that will compose our Pivot: simply add two new Views in the **Views** folder (called **PivotItem1View** and **PivotItem2View**) and the related ViewModels (called **PivotItem1ViewModel** and **PivotItem2ViewModel**). They are simple pages, nothing special to say about it: if you use the standard Windows Phone page template to create the View, just remember to remove the not needed stuff (since the page will be contained by the Pivot, the header with the application name and the page title are not needed). Here is a sample of a page:

<pre class="brush: xml;">&lt;phone:PhoneApplicationPage
    x:Class="CaliburnMicro.Views.PivotItem1View"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:phone="clr-namespace:Microsoft.Phone.Controls;assembly=Microsoft.Phone"
    xmlns:shell="clr-namespace:Microsoft.Phone.Shell;assembly=Microsoft.Phone"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    FontFamily="{StaticResource PhoneFontFamilyNormal}"
    FontSize="{StaticResource PhoneFontSizeNormal}"
    Foreground="{StaticResource PhoneForegroundBrush}"
    SupportedOrientations="Portrait" Orientation="Portrait"
    mc:Ignorable="d"
    shell:SystemTray.IsVisible="True"&gt;

    &lt;!--LayoutRoot is the root grid where all page content is placed--&gt;
    &lt;Grid x:Name="LayoutRoot" Background="Transparent"&gt;
        &lt;Grid.RowDefinitions&gt;
            &lt;RowDefinition Height="Auto"/&gt;
            &lt;RowDefinition Height="*"/&gt;
        &lt;/Grid.RowDefinitions&gt;

      &lt;!--ContentPanel - place additional content here--&gt;
        &lt;Grid x:Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0"&gt;

        &lt;/Grid&gt;
    &lt;/Grid&gt;

&lt;/phone:PhoneApplicationPage&gt;</pre>

And here is the related ViewModel:

<pre class="brush: xml;">public class PivotItem1ViewModel: Screen
{
    public PivotItem1ViewModel()
    {
        DisplayName = "First pivot";
    }
}</pre>

Nothing special: it just inherits from the **Screen** class (like every other ViewModel we’ve created in the previous posts). Just notice that, in the constructor, I set a property called **DisplayName,** that is part of the **Screen** base class: it’s the name of the pivot item and it will be used as header.

Now it’s the turn to see the XAML of the Conductor page (the one called **PivotView**):

<pre class="brush: xml;">&lt;Grid x:Name="LayoutRoot" Background="Transparent"&gt;
    &lt;!--Pivot Control--&gt;
    &lt;phone:Pivot Title="MY APPLICATION" x:Name="Items" SelectedItem="{Binding ActiveItem, Mode=TwoWay}"&gt;
        &lt;phone:Pivot.HeaderTemplate&gt;
            &lt;DataTemplate&gt;
                &lt;TextBlock Text="{Binding DisplayName}" /&gt;
            &lt;/DataTemplate&gt;
        &lt;/phone:Pivot.HeaderTemplate&gt;
    &lt;/phone:Pivot&gt;
&lt;/Grid&gt;</pre>

You can notice some Caliburn magic here: the first thing is that the name of the **Pivot** control is **Items;** this way, it’s automatically connected to the **Items** property in the ViewModel, which contains the collections of view to add. The second thing is that the **SelectedItem** property is connected (in two way mode) to a property called **ActiveItem**. This property is declared in the **Conductor<T>** base class and it’s used to hold a reference to the ViewModel of the current screen. It’s important to set it, otherwise the **ActivateItem()** method we’ve seen in the **PivotViewModel** class won’t work.

The last thing to notice is that we override the **HeaderTemplate** of the **Pivot** control: we simply set a **TextBlock** in binding with the **DisplayName** property. This way, the title of the page will be automatically taken from the **DisplayName** property we’ve set in the page’s ViewModel.

Ok, now are we ready to test the application? Not yet! First we have to register, in the **Configure()** method of the boostrapper class, all the ViewModels we’ve just created (the conductor’s one plus all the single pages), otherwise Caliburn won’t be able to resolve them.

<pre class="brush: xml;">protected override void Configure()
{
    container = new PhoneContainer(RootFrame);

    container.RegisterPhoneServices();

    container.PerRequest&lt;PivotViewModel&gt;();
    container.PerRequest&lt;PivotItem1ViewModel&gt;();
    container.PerRequest&lt;PivotItem2ViewModel&gt;();

    AddCustomConventions();
}</pre>

Now you’re ready to test the application: if you did everything correct, you’ll see your Pivot with 2 pages, one for every View and ViewModel you’ve added. You can now play with the app: you can add some data to display in a specific page, or you can add more items to the Pivot control. It’s up to you!

In the next post we’ll see how to deal with the Pivot control and how to implement lazy loading. In the meantime, have fun with the sample project!

<div class="wlWriterEditableSmartContent" id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:c68ffdc4-2fcf-47c7-9a58-fbdc7005c199" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/07/CaliburnMicro_Pivot.zip" target="_blank">Download the sample project</a>
  </p>
</div>

**The Caliburn Micro posts series**

  1. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-theory/" target="_blank">The theory</a>
  2. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-first-project/" target="_blank">The first project</a>
  3. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-actions/" target="_blank">Actions</a>
  4. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">Collections and navigation</a>
  5. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-tombstoning/" target="_blank">Tombstoning</a>
  6. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-advanced-navigation-and-deep-links/" target="_blank">Advanced navigation and deep links</a>
  7. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-messaging/" target="_blank">Messaging</a>
  8. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-using-launchers-and-choosers/" target="_blank">Using launchers and choosers</a>
  9. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-use-your-own-services-and-how-to-pass-data-between-different-pages/" target="_blank">Use your own services</a>
 10. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">The Application Bar</a>
 11. Pivot
 12. [Lazy loading with pivot](http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-lazy-loading-with-pivot/)