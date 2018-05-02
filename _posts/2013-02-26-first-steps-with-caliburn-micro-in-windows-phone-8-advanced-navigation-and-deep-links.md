---
id: 1772
title: 'First steps with Caliburn Micro in Windows Phone 8 &#8211; Advanced navigation and deep links'
date: 2013-02-26T10:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=1772
permalink: /first-steps-with-caliburn-micro-in-windows-phone-8-advanced-navigation-and-deep-links/
categories:
  - Windows Phone
tags:
  - Caliburn
  - MVVM
  - Windows Phone
---
<a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation" target="_blank">In one of the previous posts</a> about Caliburn Micro we’ve explored how navigation is managed in a Windows Phone application developed using MVVM. In this post we’re going to cover some advanced scenarios, like intercepting the navigation events and working with the deep links that are widely used by many Windows Phone APIs.

### Deep links

Windows Phone has always supported a way to pass parameters from a page to another using query string parameters: Windows Phone 7.5 has introduced the deep link concept, that is used to provide the same mechanism also when the application is opened from the outside. This feature is widely used: when you have a secondary tile, the navigation that is triggered uses a deep link to identify which tile has been tapped and which is the context to display; when you receive a toast notification and you tap on it the app is opened and you can use a deep link to identify the context and display a specific information; the same happens also for the new Voice control feature: when a voice command is issued, your app receives it using a deep link.

Intercepting this information is trivial in a MVVM application, because usually the operation consists in two steps:

  * Intercepting the **OnNavigatedTo** event, that is triggered when you navigate towards the current page. 
      * Using the **NavigationContext** class to get access to the query string parameters. </ul> 
    The problem is that both of these operations, usually, can be done only in the code behind of a view: you don’t have access to these events and classes in the view model.
    
    Luckily Caliburn Micro provides a useful naming convention to manage this scenario: it’s enough to declare a property in your view model with the same name of the query string parameter and Caliburn Micro will automatically inject the value retrieved from the URL, in a similar way we’ve seen for the standard navigation <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation" target="_blank">in a previous post</a>.
    
    Let’s try it with a sample application and let’s start with the standard scenario: a view (the **MainPage.xaml**) and a view model, bind together with the standard naming convetion. We’re going to add in the view a button, that will be used to create a secondary tile. Plus, we’ll add also a TextBox, that will be used to display the value returned by the query string.
    
    Here is the XAML:
    
    <pre class="brush: xml;">&lt;StackPanel&gt;
    &lt;TextBox Text="{Binding Name}" /&gt;
    &lt;Button Content="Create secondary tile" x:Name="CreateTile" /&gt;
&lt;/StackPanel&gt;</pre>
    
    And here is the code of the view model:
    
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

    public void CreateTile()
    {
                    ShellTileData tile = new StandardTileData
                                     {
                                         Title = "Test",
                                     };

            ShellTile.Create(new Uri("/Views/MainPage.xaml?Name=Matteo", UriKind.Relative), tile);
    }

}</pre>
    
    The **Name** property is the one connected with the TextBox, while the **CreateTile** method defines the template for the secondary tile with just a title. In the end we effectly create the tile, by passing the template and the deep link that identifies the tile. You can notice that the name of the parameter is the same of the property we’ve defined in the ViewModel: **Name**.
    
    Now launch the application: when you tap on the button the application will be closed, to display the new secondary tile that has just been created. Tap on the secondary tile and… the magic happens again! You’ll see the name “Matteo” displayed in the TextBox. This happens because the parameter in the query string is automatically injected in the property of the view model, since they both share the same name. Cool, isn’t it?
    
    ### 
    
    ### Intercepting navigation events
    
    Another common scenario when you develop a Windows Phone application is to intercept navigation events, so that you can be notified when the view is displayed. Usually the trick is to register the view models of our views with the **PerRequest** method in the bootstrapper: this way, every time a view is requested a new view model is created, so we can initialize it in the constructor. But, sometimes, this isn’t the best approach and it can’t be applied to every page: for example, the main page is always alive, since it’s always part of the stack of the application’s pages. For this reason, you can’t rely on the public constructor of the view model in case you want do something (for example, refreshing the data) every time the user navigates back to the main page.
    
    For the same reasons I’ve explained when I talked about deep links, it’s an hard task to accomplish in a MVVM application: events like **OnNavigatedTo, OnNavigatedFrom** or **Loaded** are available only in the code behind, because a Windows Phone page inherits from the **PhoneApplicationPage** class. For this reason, Caliburn Micro offers a class to use in our view models, that provides similar events in a view model: the class’ name is **Screen** and, to use it, you’ll need to let your view model inherit from it.
    
    This class puts together many pieces of Caliburn Micro, so that you don’t have to inherit your view model from too many interfaces. If we take a look at the Screen class, we’ll notice the following definition:
    
    <pre class="brush: csharp;">public class Screen : ViewAware, IScreen, IHaveDisplayName, IActivate, IDeactivate, IGuardClose, IClose, INotifyPropertyChangedEx, INotifyPropertyChanged, IChild
{

}</pre>
    
    As you can see, this class implements many interfaces, specifically **ViewAware, IActivate** and **IDeactivate** that exposes all the needed methos to interact with the view lifecycle. Plus, it implements also the **INotifyPropertyChanged** event: in case your view model inherits from the **Screen** class you don’t have to make it inherits also from the **PropertyChangedBase** class, the method **NotifyOfPropertyChange** is already supported.
    
    Once your view model is set up, you can override some method to interact with the lifecycle of your view:
    
    <pre class="brush: csharp;">public class MainPageViewModel: Screen
{
    protected override void OnViewAttached(object view, object context)
    {
        base.OnViewAttached(view, context);
        Debug.WriteLine("OnViewAttached");
    }

    protected override void OnInitialize()
    {
        base.OnInitialize();
        Debug.WriteLine("OnInitialize");
    }

    protected override void OnViewReady(object view)
    {
        base.OnViewReady(view);
        Debug.WriteLine("OnViewReady");
    }

    protected override void OnActivate()
    {
        base.OnActivate();
        Debug.WriteLine("OnActivate:");
    }

    protected override void OnViewLoaded(object view)
    {
        base.OnViewLoaded(view);
        Debug.WriteLine("OnViewLoaded");
    }

    protected override void OnDeactivate(bool close)
    {
        base.OnDeactivate(close);
        Debug.WriteLine("OnDeactivate");
    }
}</pre>
    
    Here is a description of the methods:
    
      * **OnViewAttached** is invoked when the view model is set as data context of the view. 
          * **OnInitalize** is called when the view is initalized. 
              * **OnViewReady** is called when the view is ready to be displayed 
                  * **OnViewLoaded** is called when the view is fully loaded and every control in the page is initalized 
                      * **OnActivate** is called every time the view is displayed 
                          * **OnDeactivate** is called every time the view is hided, because you have navigated away to another page or you have closed or suspended the app. </ul> 
                        The most important ones are **OnActivate** and **OnDeactivate**, that match with the **OnNavigatedTo** and **OnNavigatedFrom** events in the page: you can use them to do custom operation that can’t be managed with the helpers provided by Caliburn Micro. Specifically, the **OnActivate** event is very important because it’s raised when the page is fully loaded and you are able to interact with it: typically this event is used to load in your view model the data that should be displayed in the view. Think about the structure of a view model: usually you are used to load all the data in the view model’s constructor, since it’s invoked every time the view is displayed. In case you need to do asynchronous operations there’s a problem: the constructor of a class can’t be asynchronous.&nbsp; One approach is to defer the loading operation to another method (for example, LoadData) that is marked as **async** and that is called in the public constructor.
                        
                        <pre class="brush: xml;">public MainPageViewModel()
{
    LoadData();
}

private async void LoadData()
{
    MyData = await service.LoadData();
}</pre>
                        
                        Which is the problem of this approach? That LoadData needs to be a **void** method: it can’t return a **Task**, because in the view model’s constructor we can’t use the **await** keyword. This makes the LoadData method a “fire and forget” method: we launch it and we don’t wait until it’s finished. Most of the time this approach works, especially if the loading operation isn’t too long: usually the user won’t immediately interact with the application, so there’s enough time to wait that the data is fully loaded. But, in case the user starts an action that does some operations on our data, that isn’t ready yet, we can have problems. The solution is moving the loading operation inside the **OnActivate** method, that can be marked as async: this way we can safely execute all the needed asynchronous operations and be sure that when the user will start to interact with the application the data will be ready.
                        
                        <pre class="brush: xml;">public MainPageViewModel()
{

}

protected override async void OnActivate()
{
    MyData = await service.LoadData();
}</pre>
                        
                        &nbsp;
                        
                        In this case it’s correct that the **OnActivate()** method’s type is “fire and forget”, since the activation operations are automatically managed by the operating system. It’s the same that happens when we mark as async an event handler: since the event is automatically managed by the OS, there’s no need to wait that the operations are finished before releasing the handler.
                        
                        The **OnActivate** method is also very important when we need to manage deep links: it can happen, in fact, that when the view model is created and the constructor is invoked the query string parameters aren’t stored yet in our properties. This happens because the **OnNavigatedTo** event (that is used to read the parameters) is triggered after that the view model is created. In the deep link sample we’ve seen earlier we couldn’t notice the issue, thanks to the binding: after that the page is fully loaded, the value of the query string parameter was stored in the **Name** property; thanks to the binding and to the **NotifyOfPropertyChange** event, as soon as it happened, the TextBox in the view was automatically updated with the value.
                        
                        However, we could have noticed the issue if we would have tried to manipulate the value of the **Name** property inside the view model’s constructor: in this case we would have noticed that the value is still null, so any operation on the data would have caused an exception. The workaround is to execute these operations inside the **OnActivate** method: when it’s triggered the navigation is already completed, so we will find the query string parameter in our **Name** property as expected.
                        
                        You can download a sample project the implements what we’ve learned in this post with the link below.
                        
                        <div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:471f8804-d972-4a8c-9b8d-c70f3e92cf73" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
                          <p>
                            <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/07/CaliburnMicro_AdvancedNavigation.zip" target="_blank">Download the sample project</a>
                          </p>
                        </div>
                        
                        &nbsp;
                        
                        **The Caliburn Micro posts series**
                        
                          1. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-theory/" target="_blank">The theory</a> 
                              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-first-project/" target="_blank">The first project</a> 
                                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-actions/" target="_blank">Actions</a> 
                                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">Collections and navigation</a> 
                                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-tombstoning/" target="_blank">Tombstoning</a> 
                                              * Advanced navigation and deep links 
                                                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-messaging/" target="_blank">Messaging</a> 
                                                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-using-launchers-and-choosers/" target="_blank">Using launchers and choosers</a> 
                                                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-use-your-own-services-and-how-to-pass-data-between-different-pages/" target="_blank">Use your own services</a> 
                                                              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">The Application Bar</a> 
                                                                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-pivot/" target="_blank">Pivot</a> 
                                                                      * [Lazy loading with pivot](http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-lazy-loading-with-pivot/) </ol>