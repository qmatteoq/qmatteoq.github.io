---
id: 1572
title: 'First steps with Caliburn Micro in Windows Phone 8 &ndash; Collections and navigation'
date: 2013-02-15T10:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=1572
permalink: /first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/
categories:
  - Windows Phone
tags:
  - Caliburn
  - MVVM
  - Windows Phone
---
We continue our journey about Caliburn Micro and how to use this MVVM framework in a Windows Phone 8 application by understanding how to manage collections and navigation.

### 

### Collections

One of the most common scenarios in a Windows Phone application is the usage of collections: we have a list of data and we need to display it in a ListBox or in LongListSelector. The standard approach is to create a collection in the code (usually, when we’re talking about the XAML world, we use an **ObservableCollection**) and then to bind it to the **ItemsSource** property. Guess what? Caliburn Micro has another naming convention for that! And a pretty smart one too! To assign the collection to the **ItemsSource** property we use the standard naming convention we already know: **the name of the property in the view model should match the name of the control**. But here comes the smart decision: if you create a property which name is **Selected** followed by the name of the control in singular, it will be automatically bound with the **SelectedItem** property of the control. Let’s make an example:

<pre class="brush: xml;">&lt;ListBox x:Name="Items"&gt;
    &lt;ListBox.ItemTemplate&gt;
        &lt;DataTemplate&gt;
            &lt;StackPanel&gt;
                &lt;TextBlock Text="{Binding}" /&gt;
            &lt;/StackPanel&gt;
        &lt;/DataTemplate&gt;
    &lt;/ListBox.ItemTemplate&gt;
&lt;/ListBox&gt;</pre>

&nbsp;

In the XAML we add a ListBox with a simple template: it will simply display, for every item in the collection, a TextBlock with the value of the single item. The name of the ListBox is **Items** so, by following the Caliburn Micro naming convention, we expect to have in the view model a property called **Items**, which is the collection that will be displayed.

<pre class="brush: csharp;">private ObservableCollection&lt;string&gt; items;

public ObservableCollection&lt;string&gt; Items
{
    get { return items; }
    set
    {
        items = value;
        NotifyOfPropertyChange(() =&gt; Items);
    }

}</pre>

And here’s come the magic:

<pre class="brush: csharp;">private string selectedItem;

public string SelectedItem
{
    get { return selectedItem; }
    set
    {
        selectedItem = value;
        NotifyOfPropertyChange(() =&gt; SelectedItem);
        MessageBox.Show(value);
    }
}</pre>

As you can see the name of the property is **SelectedItem**, which is the name of the other property in singular (Items –> Item) prefixed by the word **Selected**. In the setter of the property we’ve added a line of code to display, using a MessageBox, the value of the selected property, that will be helpful for our tests. Now let’s try it: in the public constructor of the view model let’s add some fake data and assign it to the **Items** property.

<pre class="brush: csharp;">public MainPageViewModel()
{
    Items = new BindableCollection&lt;string&gt;
                {
                    "Matteo",
                    "Mario",
                    "John"
                };
}</pre>

If you launch the application and you’ve correctly followed the steps the ListBox will contain the three test values we’ve defined: by tapping on one of the elements a MessageBox with the selected name will appear.

[<img title="list1" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="list1" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/list1_thumb.png?resize=200%2C333" width="200" height="333"  data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/list1.png)[<img title="list2" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px 0px 0px 15px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="list2" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/list2_thumb.png?resize=200%2C333" width="200" height="333"  data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/list2.png)

### 

### Navigation

Usually managing the navigation from a page to another of the application is one of the trickiest tasks using MVVM. The biggest problem to face is that in Windows Phone we use the **NavigationService**, that can be used only in the code behind that is connected to a view. You can’t directly access to it from another class, for example, like a ViewModel. To support the developer Caliburn Micro comes with a built in NavigationService, that can be used in a ViewModel by simply adding a reference in the public constructor of the application. The built in dependency injection container will take care of resolving the dependency for you and will give you access to it. So, the first thing to use manage navigation in a MVVM application developed with Caliburn Micro is to change the public constructor of your view model, like in the following example:

<pre class="brush: csharp;">public class MainPageViewModel : PropertyChangedBase
{
    private readonly INavigationService navigationService;

    public MainPageViewModel(INavigationService navigationService)
    {
        this.navigationService = navigationService;
    }
}</pre>

From now on, you’ll be able to use the NavigationService inside your view model: you won’t have to register anything in the bootstrapper, since Caliburn Micro will take care of everything (as for every other service that is embedded with the toolkit).

The NavigationService that comes with the toolkit supports a view model first approach: instead of declaring which is the URL of the page where we want to take the user (that is the standard approach), we declare which is the ViewModel we want to display. The service will take care of creating the correct URL and display the view that is associated with the view model. Let’s see how does it work.

First we need a new page where to redirect the user: add to your project a new page and a new class in the **ViewModels** folder. Remember to use the naming convention we’ve learned in the second post of the series: if the name of the page is **Page2View.xaml**, the name of the ViewModel will have to be **Page2ViewModel.cs**. Before moving on, you have to remember to register the new view model in the **Configure** method of the bootstrapper, like in the following example:

<pre class="brush: csharp;">protected override void Configure()
{
    container = new PhoneContainer(RootFrame);

    container.RegisterPhoneServices();
    container.PerRequest&lt;MainPageViewModel&gt;();
    container.PerRequest&lt;Page2ViewModel&gt;();
    AddCustomConventions();
}</pre>

Now add a button in the main page of your application and, using the naming convention we’ve learned in the previous post, assign to it a method in your view model, that will trigger the navigation towards the second page.

<pre class="brush: xml;">&lt;Button Content="Go to page 2" x:Name="GoToPage2" /&gt;</pre>

&nbsp;

<pre class="brush: csharp;">public void GoToPage2()
{
    navigationService.UriFor&lt;Page2ViewModel&gt;()
        .Navigate();
}</pre>

With the **UriFor<T>** method of the navigation service we get the needed URL for our view model, then we call the **Navigate**()method to trigger the navigation and redirect the user to the requested page.

### Navigation with parameters

You should already know that, when navigating from a page to another, you are able to carry some parameters in the query string, that can be used in the new page, like in the following sample

_/Page2View.xaml?Name=Matteo_

Using the NavigationContext you are able, in the view Page2View.xaml, to retrieve the value of the **Name** property that is passed using a query string. How to do it using Caliburn Micro? The first thing is to define, in our destination page’s view model (in our example, the class **Page2ViewModel**) a property, that will hold the&nbsp; value of the parameter.

<pre class="brush: csharp;">public class Page2ViewModel: PropertyChangedBase
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
}</pre>

Then, we change the navigation operation like this:

<pre class="brush: csharp;">public void GoToPage2()
{
    navigationService.UriFor&lt;Page2ViewModel&gt;()
                     .WithParam(x =&gt; x.Name, "Matteo")
                     .Navigate();
}</pre>

We’ve added the method **WithParam**, that accepts two parameters: the first one is the property of the destination view model that will hold our value and it’s specified using the lambda syntax (x represents the destination view model, in our example the instance of the **Page2ViewModel** class); the second parameter is the value that the property will have. When the Page2View.xaml view will be loaded, the Page2ViewModel will hold, in the **Name** property, the value of the parameter we’ve passed during the navigation, so we can use it for our purposes. For example, we can simply display it by adding a in the XAML a TextBlock with the same name of the property (do you remember the naming convention?)

<pre class="brush: xml;">&lt;Grid x:Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0"&gt;
    &lt;StackPanel&gt;
        &lt;TextBlock x:Name="Name"&gt;&lt;/TextBlock&gt;
    &lt;/StackPanel&gt;
&lt;/Grid&gt;</pre>

**Important!** It’s true that Caliburn Micro does a lot of magic, but the navigation with parameter feature is still based on query string parameter. The only magic is that these parameters are automatically injected in the properties of your view model, but they still are strings: you can use the **WithParam** method to pass to the new view just plain values, like strings and number. You can’t use it to pass complex objects.

### To be continued

The journey is not ended yet, we still have to see how to manage messages and tombstoning with Caliburn Micro. Coming soon in the next posts! While you wait, you can download the sample project and start playing with it!

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:344881bc-fbde-4ab3-91d7-c7b717532ce1" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/07/CaliburnMicro_CollectionsNavigation.zip" target="_blank">Download the sample project</a>
  </p>
</div>

&nbsp;

**The Caliburn Micro posts series**

  1. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-theory/" target="_blank">The theory</a> 
      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-first-project/" target="_blank">The first project</a> 
          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-actions/" target="_blank">Actions</a> 
              * Collections and navigation 
                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-tombstoning/" target="_blank">Tombstoning</a> 
                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-advanced-navigation-and-deep-links/" target="_blank">Advanced navigation and deep links</a> 
                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-messaging/" target="_blank">Messaging</a> 
                              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-using-launchers-and-choosers/" target="_blank">Using launchers and choosers</a> 
                                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-use-your-own-services-and-how-to-pass-data-between-different-pages/" target="_blank">Use your own services</a> 
                                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">The Application Bar</a> 
                                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-pivot/" target="_blank">Pivot</a> 
                                              * [Lazy loading with pivot](http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-lazy-loading-with-pivot/) </ol>