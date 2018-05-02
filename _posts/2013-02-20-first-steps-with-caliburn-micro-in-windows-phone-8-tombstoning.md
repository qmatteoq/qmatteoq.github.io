---
id: 1602
title: 'First steps with Caliburn Micro in Windows Phone 8 &ndash; Tombstoning'
date: 2013-02-20T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=1602
permalink: /first-steps-with-caliburn-micro-in-windows-phone-8-tombstoning/
categories:
  - Windows Phone
tags:
  - Caliburn
  - MVVM
  - Windows Phone
---
Tombstoning is the joy and the pain of every Windows Phone developer: it’s a joy because it’s a really smart mechanism to manage the suspension process of an application and to “simulate” that an application has never been closed. But it’s also a pain, because managing it correctly it’s not easy, and this is even more true for a MVVM application. The main reason is that usually tombstoning is managed using the navigation events of a page: when the user navigates away from a page (**OnNavigatedFrom**) we save the status of the page; when the user navigates to the page (**OnNavigatedTo**) we retrieve the previous status. in MVVM we don’t have direct access to the navigation events, since they are available only in the view: for this reason, every MVVM toolkit usually provides a way to hook up to these events also in the view model, to properly save and restore the state.

Caliburn Micro uses another approach: we create another class, that is connected to the ViewModel, and we specify what to do when the application is suspended.

Let’s start with a real example: create a new Windows Phone project and add Caliburn Micro using NuGet. As explained in the other post, create a new ViewModel for the first page and connect them using the standard naming convention.

Now we’re going to add a **TextBox** control inside the page and, using the standard naming convention, we bind it to a string property in our ViewModel so that, everytime the user writes something in the TextBox, the value is saved in the property.

<pre class="brush: xml;">&lt;TextBox x:Name="Name" /&gt;</pre>

&nbsp;

<pre class="brush: csharp;">public class MainPageViewModel: PropertyChangedBase
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
}</pre>

Let’s do an experiment first to test the tombstoning process. Let’s start the application: write something in the TextBox, suspend the application by pressing the Start button and then, by pressing the Back button, go back to the application. The text will still be there: this happens because, as a standard behavior, the application is suspended, but the process is kept in memory, so there’s no need to restore the state.

Now we need to simulate the tombstoning: Visual Studio provides a special option in the project’s properties to test it, since there’s no manual way to trigger it. The operatying system always suspends an application, as a standard behavior: only when the system is getting low on resources it can choose to tombstone an app. You can find this option by right clicking on the project in Visual Studio: choose Properties and, in the Debug tab, you’ll find a checkbox, with the label “**Tomstone upon activation while debugging”.** Enable it and now repeat the previous test: you’ll notice that, this time, when the application is activated the text in the TextBox will be gone.

Here comes tombstoning: we need to save the value in TextBox and restore it when the application is activated. Caliburn Micro provides an easy way to do it: you’ll have to create another class, that will manage the tombstoning for you.

This class will have to inherit from the **StorageHandler<T>,** where T is the ViewModel of the view which state we need to save. Here is an example:

<pre class="brush: csharp;">public class MainPageModelStorage: StorageHandler&lt;MainPageViewModel&gt;
{
    public override void Configure()
    {
        Property(x =&gt; x.Name)
            .InPhoneState();
    }
}</pre>

When you implement the **StorageHandler<T>** class you are required to implement the **Configure** method, that contains the “instructions” for Caliburn Micro about what to do when the app is suspended. We use the **Property** object and, using a lambda expression, we specify which property of the View Model we want to save during the suspension. The Intellisense will help you: since x, in the lambda expression, is your ViewModel (in the sample, **MainPageViewModel**), you’ll have access to all the public properties you have defined (in our sample, the **Name** property). In the end you can specify where the status should be saved: with the method **InPhoneState()** the property is saved in the transient Windows Phone state, that is erased in case the application is closed or opened from the beginning. It’s the perfect solution for our scenario: imagine that our TextBox is part of a form, that the user needs to fill. It’s important to keep the status if the app is suspended, but it’s correct to start over from scratch in case the application is restarted.

In case, instead, we want to restore the value of our property every time the app is restored, we can use the **InAppSettings()** method:

<pre class="brush: csharp;">public class MainPageModelStorage: StorageHandler&lt;MainPageViewModel&gt;
{
    public override void Configure()
    {
        Property(x =&gt; x.Name)
            .InAppSettings();
    }
}</pre>

Let’s try it: open the application, write something in the TextBox and suspend it. If you press Back, the app will be reopened and the text will still be there, even if we’ve told to the the project to tombstone the app: Caliburn Micro has automatically saved and restored the value of the **Name** property of our ViewModel and the final result, for the user, is that it’s like the app has been regularly suspended. If we close the app using the Back button instead of suspending it and we open it again, this time the text will be gone: it’s correct, because we’ve saved the status in the phone state. If we change the **Configure** method of the **MainPageModelStorage** class to save the data in the application settings, instead, we will notice that the text in the TextBox is restored even after the app has been closed.

### 

### 

### It’s not over yet

We haven’t finished yet to dig about Caliburn Micro: we still have to see how to manage messaging. Stay tuned, meanwhile you can play with the sample project below.

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:6a3f1149-aee7-4b55-934a-1c10451cd79e" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/07/Caliburn_Tombstoning.zip" target="_blank">Download the sample project</a>
  </p>
</div>

&nbsp;

**The Caliburn Micro posts series**

  1. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-theory/" target="_blank">The theory</a> 
      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-first-project/" target="_blank">The first project</a> 
          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-actions/" target="_blank">Actions</a> 
              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">Collections and navigation</a> 
                  * Tombstoning 
                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-advanced-navigation-and-deep-links/" target="_blank">Advanced navigation and deep links</a> 
                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-messaging/" target="_blank">Messaging</a> 
                              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-using-launchers-and-choosers/" target="_blank">Using launchers and choosers</a> 
                                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-use-your-own-services-and-how-to-pass-data-between-different-pages/" target="_blank">Use your own services</a> 
                                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">The Application Bar</a> 
                                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-pivot/" target="_blank">Pivot</a> 
                                              * [Lazy loading with pivot](http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-lazy-loading-with-pivot/) </ol>