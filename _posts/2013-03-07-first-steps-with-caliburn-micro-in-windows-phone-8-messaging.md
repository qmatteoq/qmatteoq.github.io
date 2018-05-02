---
id: 2082
title: 'First steps with Caliburn Micro in Windows Phone 8 &ndash; Messaging'
date: 2013-03-07T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=2082
permalink: /first-steps-with-caliburn-micro-in-windows-phone-8-messaging/
categories:
  - Windows Phone
tags:
  - Caliburn
  - MVVM
  - Windows Phone
---
When you develop an application using MVVM one of the most frequent scenarios is the communication between different view models: for example, you need to exchange data between two view models or you need that, as a consequence of an action in a page, something should happen in another page. In MVVM application the answer is messaging! All the MVVM toolkits offer a messenger class: you can think of it like a postman, that is able to dispatch and to process messages that are exchanged between view models.

**UPDATE:** since many of you have asked, I’ve updated the post to explain how to manage communications also between a View and a ViewModel

The message is represented by a simple class: it can have one or more properties, where you can store the data that you need to pass between the view models. Using messaging, usually, means to apply the following flow:

  * A view model prepares a message, by creating a new instance of the class that represents it and then send it using the messenger. 
      * One or more view models can subscribe to receive that type of message: in that case, they are notified every time another view model sends that type of message and they can act, to process it and to do some operations. </ul> 
    Let’s see how this is implemented in Caliburn Micro. The scenario we’re going to use for our sample is the following: we have two views, each one with its view model. The second page has a button to send a message, that contains a name: when the button is pressed, the message with the name is passed to the first view model, so that it can be displayed in the first page.
    
    The first step is to setup the project: please follow the steps described in <a href="http://wp.qmatteoq.com/tag/caliburn/" target="_blank">all the previous tutorials</a> to create the bootstrapper, the two pages and the two view models and to connect all of them together.
    
    Before taking care of the messaging stuff, let’s prepare the needed infrastructure to make the app working. The first thing is to add, in the first page, a TextBox and a Button: the first one will be used to show the information passed by the other page, the second one will be used to navigate to the second page. Open the main page (usually the **MainPage.xaml** file) and add the following XAML in the Grid labeled **ContentPanel:**
    
    <pre class="brush: xml;">&lt;StackPanel&gt;
    &lt;TextBox x:Name="Name" /&gt;
    &lt;Button Content="Go to page 2" x:Name="GoToPage2" /&gt;
&lt;/StackPanel&gt;</pre>
    
    Again, we are using the standard Caliburn naming conventions: the TextBox control will be bound with a property called **Name** in the ViewModel, while the button, when clicked, will execute the **GoToPage2** method defined in the ViewModel.
    
    And here’s our ViewModel:
    
    <pre class="brush: csharp;">public class MainPageViewModel: Screen
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

    public MainPageViewModel(INavigationService navigationService)
    {
        this.navigationService = navigationService;
    }

    public void GoToPage2()
    {
        navigationService.UriFor&lt;SecondPageViewModel&gt;().Navigate();
    }
}</pre>
    
    There isn’t too much to say about it: it has a **Name** property, which type is string, and it holds a reference to the Caliburn’s navigation service, which is used in the **GoToPage2** method to redirect the user to the second page of the application. Now let’s see the XAML inside the **ContentPanel** of the second page, that is very simple too:
    
    <pre class="brush: csharp;">&lt;StackPanel&gt;
    &lt;Button Content="Send message" x:Name="SendMessage" /&gt;
&lt;/StackPanel&gt;</pre>
    
    The page contains a button, that is linked to the method **SendMessage** that is declared in the ViewModel. Let’s take a look at the ViewModel of the second page, that is much more interesting because we introduce the classes needed to exchange messages:
    
    &nbsp;
    
    <pre class="brush: csharp;">public class SecondPageViewModel: Screen
{
    private readonly IEventAggregator eventAggregator;

    public SecondPageViewModel(IEventAggregator eventAggregator)
    {
        this.eventAggregator = eventAggregator;
    }

    public void SendMessage()
    {
        eventAggregator.Publish(new SampleMessage("Matteo"));
    }
}</pre>
    
    The class that is used to manage all the messaging stuff in Caliburn is called **EventAggregator** and, as for the NavigationService, it’s built in in the framework: this means that it’s enough to add a parameter in the constructor of your ViewModel which type is **IEventAggregator** to automatically have a reference to the object. Sending a message is really simple: you call the **Publish** method passing, as a parameter, the object that represents the message you want to send. As I’ve already told you before, the message is a simple class, that can hold one or more properties. Here is an example of the **SampleMessage** class:
    
    <pre class="brush: csharp;">public class SampleMessage
{
    public string Name { get; set; }

    public SampleMessage(string name)
    {
        Name = name;
    }
}</pre>
    
    Nothing special to say about: it’s a class with a property ****called **Name**. In the constructor, we allow the developer to set a value for this property simply by passing it as a parameter. The result is that, in the previous sample, we use the **EventAggregator** to publish a message that contains the value **Matteo** in the **Name** property.
    
    Ok, the **EventAggregator**, that is our postman, has sent the message. How can the ViewModel of the first page receive it? We need to do some changes:
    
    <pre class="brush: csharp;">public class MainPageViewModel: Screen, IHandle&lt;SampleMessage&gt;
{
    private readonly IEventAggregator eventAggregator;
    private readonly INavigationService navigationService;
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

    public MainPageViewModel(IEventAggregator eventAggregator, INavigationService navigationService)
    {
        this.eventAggregator = eventAggregator;
        this.navigationService = navigationService;

        eventAggregator.Subscribe(this);
    }

    public void GoToPage2()
    {
        navigationService.UriFor&lt;SecondPageViewModel&gt;().Navigate();
    }

    public void Handle(SampleMessage message)
    {
        Name = message.Name;
    }
}</pre>
    
    The first step is to inherit our ViewModel from the **IHandle<T>** class, that is part of the Caliburn toolkit and that it’s been created specifically for messaging purposes. When a ViewModel inherits from **IHandle<T>**, we say that the ViewModel will be able to receive messages which type is T. In our sample, since the **SecondPageViewModel** class sends a message which type is **SampleMessage**, we make the **MainPageViewModel** class to inherit from **IHandle<SampleMessage>.**
    
    When we inherit from this interface, we are forced to implement in the ViewModel the **Handle** method, that contains as parameter the message that has been received. As you can see, dealing with messages is really simple: we simply need to include in the **Handle** method the code that we want to execute when the message is received. In this sample, we set the **Name** property of the **ViewModel** with the value of the **Name** property that comes with the message.
    
    There’s another important thing to do to get messages working: we need to tell to the ViewModel that we want to subscribe to receive messages. For this reason, we need a reference to the **IEventAggregator** class also in the view model that will receive the message: we do it in the constructor, as usual, and, once we have it, we call the **Subscribe** method passing a reference to the ViewModel itself (we do this by passing the value **this**).
    
    And we’re done! Now if we launch the application, we go to the second page, we press the **Send message** button and we go back to the first page, we’ll see that in the **TextBox** the string **Matteo** will be displayed.
    
    ### 
    
    ### 
    
    ### Communications between the View and the ViewModel
    
    In some cases you may need to exchange data between the view and the view model, for example to manage animations or events that can’t be subscribed using binding. The approach to do that is exactly the same we’ve just seen: the only difference is that we need a workaround to get access to the **EventAggregator**, since we can’t put a parameter which type is **IEventAggregator** in the constructor of the page: Caliburn Micro won’t be able to resolve it and will raise an exception. First we need to apply a change to the bootstrapper, to make the **PhoneContainer** object (that is the used to register and resolve classes in the application) a public property, so that it can be accessed also from other classes:
    
    <pre class="brush: csharp;">public class Bootstrapper : PhoneBootstrapper
{
    public  PhoneContainer container { get; set; }

    protected override void Configure()
    {
        container = new PhoneContainer();

        container.RegisterPhoneServices(RootFrame);
        container.PerRequest&lt;MainPageViewModel&gt;();
        AddCustomConventions();
    }

    static void AddCustomConventions()
    {
        //ellided  
    }

    protected override object GetInstance(Type service, string key)
    {
        return container.GetInstance(service, key);
    }

    protected override IEnumerable&lt;object&gt; GetAllInstances(Type service)
    {
        return container.GetAllInstances(service);
    }

    protected override void BuildUp(object instance)
    {
        container.BuildUp(instance);
    }
}</pre>
    
    &nbsp;
    
    Then we need, in the code behind of the page, to get a reference to the container:
    
    <pre class="brush: csharp;">private IEventAggregator eventAggregator;

// Constructor
public MainPage()
{
    InitializeComponent();

    Bootstrapper bootstrapper = Application.Current.Resources["bootstrapper"] as Bootstrapper;

    IEventAggregator eventAggregator =
        bootstrapper.container.GetAllInstances(typeof (IEventAggregator)).FirstOrDefault() as IEventAggregator;

    this.eventAggregator = eventAggregator;

    eventAggregator.Subscribe(this);
}</pre>
    
    First we get a reference to the bootstrapper: since it’s declared as a global resource, we can access to it with its key by using the **Application.Current.Resources** collection. Then, thanks to the modify we did before in the bootstrapper, we are able to access to the container and, using the **GetAllInstances** method, we get a reference to the **EventAggregator** class that has been automatically registered by Caliburn Micro at startup. We need to specify, as parameter of the method, which is the type of the class we want to get a reference, in this case **typeof(IEventAggregator).** Since there’s just one **EventAggregator** registered in the application, this collection will always contain just one element: we take it using the **FirstOrDefault()** LINQ operation.
    
    Now that we have a reference to the **EventAggregator** class, we can use the same approach we’ve seen before for view models. If we want to receive a message, we have to call the **Subscribe()** method and we need to inherit our page from the **IHandle<T>** class, where **T** is the message we need to manage. ****By implementing this interface we’ll have, also in the view, to implement the **Handle** method, that receives the message as parameter. Here is an example:
    
    <pre class="brush: csharp;">public partial class MainPage : PhoneApplicationPage, IHandle&lt;SampleMessage&gt;
{
    private IEventAggregator eventAggregator;

    // Constructor
    public MainPage()
    {
        InitializeComponent();

        Bootstrapper bootstrapper = Application.Current.Resources["bootstrapper"] as Bootstrapper;

        IEventAggregator eventAggregator =
            bootstrapper.container.GetAllInstances(typeof (IEventAggregator)).FirstOrDefault() as IEventAggregator;

        this.eventAggregator = eventAggregator;

        eventAggregator.Subscribe(this);
    }

    public void Handle(SampleMessage message)
    {
        MessageBox.Show(message.Name);
    }

}</pre>
    
    In case, instead, we want to send a message we call the **Publish** method, passing the object that represents the message to send:
    
    <pre class="brush: csharp;">public partial class MainPage : PhoneApplicationPage
{
    private IEventAggregator eventAggregator;

    // Constructor
    public MainPage()
    {
        InitializeComponent();

        Bootstrapper bootstrapper = Application.Current.Resources["bootstrapper"] as Bootstrapper;

        IEventAggregator eventAggregator =
            bootstrapper.container.GetAllInstances(typeof (IEventAggregator)).FirstOrDefault() as IEventAggregator;

        this.eventAggregator = eventAggregator;

        eventAggregator.Subscribe(this);
    }

    private void OnSendOtherMessageClicked(object sender, RoutedEventArgs e)
    {
        eventAggregator.Publish(new SampleMessage("Matteo"));
    }
}</pre>
    
    In this sample we’ve created an event handler to manage the **Click** event of a button: when the user taps on it the message is sent directly from the View. Obviously, this is just a sample, in a real scenario is not useful: it would be better to create a method in the ViewModel and connect it to the button using the Caliburn naming convention. But there are some events that can’t be connected to a command using binding: in this case, using messages is a good alternative.
    
    Keep visiting this blog because we’re not done yet  <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-bottom-style: none; border-right-style: none; border-left-style: none" alt="Smile" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/03/wlEmoticon-smile.png?w=640" data-recalc-dims="1" />More posts about Caliburn Micro and Windows Phone are coming! In the meantime, you can play with a sample project that implements what we’ve seen in this post.
    
    <div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:53ac6b8d-fbe1-4d52-8335-66d78c27330f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
      <p>
        <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/07/CaliburnMicro_Messaging1.zip" target="_blank">Download the sample project</a>
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
                              * Messaging 
                                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-using-launchers-and-choosers/" target="_blank">Using launchers and choosers</a> 
                                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-use-your-own-services-and-how-to-pass-data-between-different-pages/" target="_blank">Use your own services</a> 
                                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">The Application Bar</a> 
                                              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-pivot/" target="_blank">Pivot</a> 
                                                  * [Lazy loading with pivot](http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-lazy-loading-with-pivot/) </ol>