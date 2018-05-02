---
id: 6276
title: 'Using Caliburn Micro with Universal Windows app &ndash; Messages'
date: 2014-07-14T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6276
permalink: /using-caliburn-micro-with-universal-windows-app-messages/
categories:
  - Windows 8
  - Windows Phone
tags:
  - Caliburn
  - Windows 8
  - Windows Phone
---
When you’re developing an application using the MVVM pattern, one of the most common needs is to create a communication channel between two ViewModels or a ViewModel and a View: you need to exchange some data between the two actors, but you don’t have an easy way because everything is decoupled.

Messages are the perfect solution for such scenarios: they’re basically objects, that a class (like a ViewModel) can send using a special messenger; any other class can subscribe to receive such kind of messages and perform an operation when it happens. With this approach, we are able to maintain the fundamental concept of MVVM, which is “separation of concerns”. When a class sends a message, it doesn’t know who will be the receiver: it’s the class that wants to receive the message that needs to act and prepare itself to receive messages.

In Caliburn Micro, this scenario is achieved using one of the built-in services, called **EventAggregator:** it’s our postman and we’re going to use it to publish and to receive messages.

Let’s see how it works and how to use it in a Universal Windows app.

### Sending a message

As a sample scenario, we’re going to develop a simple application with two pages: a main page and a detail page. The detail page will offer a button that will send a message containing some text; the main page will subscribe itself to receive this message and it will display the text on the screen, using a TextBlock control.

I won’t go into the details needed to setup the application and to connect the ViewModel’s properties and actions to the UI: you can see my previous posts, where I explained the <a href="http://wp.qmatteoq.com/using-caliburn-micro-with-universal-windows-app-the-project-setup/" target="_blank">basic concepts of a Caliburn Micro application</a>.

The first step to use the **EventAggregator** is to add a reference to it in our ViewModel: in the same way we did <a href="http://wp.qmatteoq.com/using-caliburn-micro-with-universal-windows-app-navigation/" target="_blank">in the previous post</a> for the **NavigationService**, we need to add a parameter in the constructor of the ViewModel’s class, which type will be **IEventAggregator**.

<pre class="brush: csharp;">public class DetailPageViewModel: Screen
{
    private readonly IEventAggregator _eventAggregator;

    public DetailPageViewModel(IEventAggregator eventAggregator)
    {
        _eventAggregator = eventAggregator;
    }
}</pre>

Now that we have a reference to the **EventAggregator** class, we can manage the button’s click: we’re going to define a method in the ViewModel, that will be invoked when the button is clicked. The purpose of this method is to send a message:

<pre class="brush: csharp;">public async void SendMessage()
{
    string message = "This is a simple message";
    await _eventAggregator.PublishOnUIThreadAsync(message);
}</pre>

In this sample the message is simply a text: we send a **string** object by passing it as parameter of the method **PublishOnUIThreadAsync()** offered by the **EventAggregator** class. There are multiple ways to send a message: we’ll deal with them later. For the moment, it’s important to know that the **PublishOnUIThreadAsync()** method simply takes the object passed as parameter and sends it as a message on the UI thread. This means that the receiver class will receive it on the same thread that manages the user interface of the application: it’s perfect for our scenario, since we’re simply going to display the text on the UI.

### Receiving a message

There are two steps to register a class to receive a message: the first one requires to use, again, the **EventAggregator** class. As a consequence, you’ll need to add an **IEventAggregator** parameter also in the receiver’s ViewModel constructor. The difference is that, this time, we’re going to call the **Subscribe()** method, that will communicate to our postman that the current class wants to receive messages.

<pre class="brush: csharp;">public class MainPageViewModel: Screen
{
    public MainPageViewModel(IEventAggregator eventAggregator)
    {
        eventAggregator.Subscribe(this);
    }
}</pre>

As a parameter of the **Subscribe()** method you need to pass which class is going to subscribe to receive messages: since, in this case, it’s the ViewModel itself, we simply pass the value **this**, that is a reference to the current class.

The second step is to implement the **IHandle<T>** interface, where T is the type of message we want to receive: by doing so, you’ll be forced to implemented the **Handle()** method, which will receive as input the message.

<pre class="brush: csharp;">public class MainPageViewModel: Screen, IHandle&lt;string&gt;
{
    private string _text;

    public string Text
    {
        get { return _text; }
        set
        {
            _text = value;
            NotifyOfPropertyChange(() =&gt; Text);
        }
    }

    public MainPageViewModel(IEventAggregator eventAggregator)
    {
        eventAggregator.Subscribe(this);
    }

    public void Handle(string message)
    {
        Text = message;
    }
}</pre>

Here is a complete sample of the **MainPageViewModel** class: first, we’ve implemented the **IHandle<string>** interface, since we want to receive the message sent by the **DetailViewModel** class, which is a string. Then, we’ve created a method called **Handle()**, which receives as parameter a string, since it’s the type of message we expect: now we are free to manage the message as we prefer, according to our needs. In the sample, we simply take the string and we assign it to a property called **Text**, which is connected to a **TextBlock** control in the View.

If we try this simple application, we’ll see that, if we tap on the button to send the message that we’ve placed in the detail page and then we go back to the main page, the text stored in the message will be successfully displayed on the screen.

### Sending and receiving complex messages

In the previous sample we’ve simply sent a string as a message. However, there are situations when using a base type can be too generic: for example, we could have multiple ViewModel registered to receive string messages, but we want that a particular message is received only by a specific ViewModel.

In this case, the solution is easy: the **EventAggregator** can send not only basic types as messages, but also complex objects. Let’s try to implement the same scenario, but with a different approach: instead of sending a simple string, we’re going to send an object that will store a string.

First, we need to add a new class in our application, that can act as a message. We’re going to call it **SimpleMessage**:

<pre class="brush: csharp;">public class SimpleMessage
{
    public string Text { get; private set; }

    public SimpleMessage(string text)
    {
        Text = text;
    }
}</pre>

It’s a simple class, which exposes a string parameter called **Text**, which is initialized using the constructor. The next step is to send the message, using the same approach we’ve seen before with the **EventAggregator** class and the **PublishOnUIThreadAsync()** method: the only difference is that, this time, instead of passing as parametere a simple string, we’re going to send a **SimpleMessage** object.

<pre class="brush: csharp;">public async void SendComplexMessage()
{
    SimpleMessage message = new SimpleMessage("This is a complex message");
    await _eventAggregator.PublishOnUIThreadAsync(message);
}</pre>

Now, in the **MainPageViewModel**, we need to subscribe to receive the **SimpleMessage** by simply implementing the **IHandle<T>** interface in the proper way, like in the following sample:

<pre class="brush: csharp;">public class MainPageViewModel: Screen, IHandle&lt;SimpleMessage&gt;
{
    private string _text;

    public string Text
    {
        get { return _text; }
        set
        {
            _text = value;
            NotifyOfPropertyChange(() =&gt; Text);
        }
    }

    public MainPageViewModel(IEventAggregator eventAggregator)
    {
        _navigationService = navigationService;

        eventAggregator.Subscribe(this);
    }

    public void Handle(SimpleMessage message)
    {
        Text = message.Text;
    }
}</pre>

Nothing special to mention: it’s the same approach we’ve used before, the only difference is that this time the **Handle()** method will receive a real object (which type is **SimpleMessage**) instead of a simple type. 

### Sending and receiving messages to the code behind

Another common scenario is the communication between a ViewModel and a View: there are, in fact, certain operations that require direct access to the controls, like starting an animation or invoking a method that is exposed only in code behind. A way to solve this scenarios is using behavior, but sometimes they can be complex to define: sending messages is much easier.

Sending and receiving messages in the code behind is the same we’ve seen before with ViewModels: we send messages using the **EventAggregator** class and we receive them by implementing the **IHandle<T>** interface. The only difference is that, in the code behind, we can’t add an **IEventAggregator** paramter to the constructor: dependency injection works fine only for ViewModels. The solution is to manually interact with the Caliburn container, to explicity ask for an **EventAggregator** object: to achieve this result, we first need to do a change in the **App** class since, by default, the container is declared as a private variable, so we can’t use it in another class.

<pre class="brush: csharp;">public sealed partial class App
{
   public WinRTContainer container { get; private set; }

   public App()
   {
       InitializeComponent();
   }
}</pre>

We’ve simply changed the container’s type from **private** to **public** and we’ve turned it into a property. Now, from every class, we can access to the container in the following way:

<pre class="brush: csharp;">WinRTContainer container = (Application.Current as App).container;</pre>

To explicity ask for an instance of a class registred in the container, we need to use the **GetInstance<T>** method. Let’s see that we want to receive the **SimpleMessage** object we’ve sent before in the code behind of the **MainPage** View. Here is how we can do it:

<pre class="brush: csharp;">public sealed partial class MainPageView : Page, IHandle&lt;SimpleMessage&gt;
{
    public MainPageView()
    {
        this.InitializeComponent();

        this.NavigationCacheMode = NavigationCacheMode.Required;

        WinRTContainer container = (Application.Current as App).container;

        IEventAggregator eventAggregator = container.GetInstance&lt;IEventAggregator&gt;();

        eventAggregator.Subscribe(this);
    }

    public void Handle(SimpleMessage message)
    {
        MessageContent.Text = message.Text;
    }
}</pre>

As you can see, there aren’t big differences with the previous approach: the class implements the **IHandle<SimpleMessage>** interface and, as a consequence, it defines the **Handle()** method which receives, as parameter, a **SimpleMessage** object. The only difference is that we set up the **EventAggregator** in another way: after we’ve obtained a reference to the **WinRTContainer** object, we ask for the **EventAggregator** instance registered in the container by calling the **GetInstance<IEventAggregator>()**&nbsp; method. Then, we proceed as usual, by calling the **Subscribe()** method passing **this** as parameter, since we want the actual code behind class to be able to receive messages.

### 

### Managing messages in a background thread

One of the new features added in Caliburn Micro 2.0 is the support to send messages in a background thread. This scenario is useful if the receiver class needs to perform intensive operation when the message is received: to avoid impacting on the UI, we can handle the message in a separate thread. Sending a message in a background thread is really easy: just use the **PublishOnBackgroundThread()** method offered by the **EventAggregator** class, like in the following sample:

<pre class="brush: csharp;">public void SendMessageInBackground()
{
    SimpleMessageInBackground message = new SimpleMessageInBackground("This is a message handled in a background thread");
    _eventAggregator.PublishOnBackgroundThread(message);
}
</pre>

&nbsp;

Then, in the receiver class, we’ll handle it in the same way we did in the previous samples: we implement the **IHandle<T>** interface and we manage the **Handle()** method in the class. The only difference is that, this time, the **Handle()** method will be executed on the background thread: we need to remember that, if we need to interact with the View (for example, by changing a control’s property) we need to use the Dispatcher, which takes care of redirecting the operation to the UI thread; otherwise, we will get a cross-thread access exception.

Take a look at the following sample:

<pre class="brush: csharp;">public sealed partial class MainPageView : Page, IHandle&lt;SimpleMessageInBackground&gt;
{
    public MainPageView()
    {
        this.InitializeComponent();

        this.NavigationCacheMode = NavigationCacheMode.Required;

        WinRTContainer container = (Application.Current as App).container;

        IEventAggregator eventAggregator = container.GetInstance&lt;IEventAggregator&gt;();

        eventAggregator.Subscribe(this);
    }

    public void Handle(SimpleMessageInBackground message)
    {
        Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =&gt;
        {
            MessageContent.Text = message.Text;
        });
    }
}
</pre>

This sample is similar to the previous one: in a code behind class we register to receive a message which type is **SimpleMessageInBackground**. The final result is the same: we display the content of the message in a **TextBlock** control called **MessageContent**. The difference, this time, is that we do it using the **RunAsync()** method of the **Dispatcher** class, since we’re interacting with a control in the XAML but the message is being handled in a background thread.

### That’s all!

In this post we’ve covered all the basic concepts about sending and receiving messages in a MVVM application built with Caliburn Micro 2.0. The last week I’ve decided to publish all the samples connected to this series of post about Caliburn Micro and Universal Windows app on GitHub: the repository is available on [https://github.com/qmatteoq/CaliburnMicro-UniversalApp-Demo](https://github.com/qmatteoq/CaliburnMicro-UniversalApp-Demo "https://github.com/qmatteoq/CaliburnMicro-UniversalApp-Demo"). You’ll find in the solution, together will all the samples of the previous posts, also a new one about messages.