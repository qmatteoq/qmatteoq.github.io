---
id: 1482
title: 'First steps with Caliburn Micro in Windows Phone 8 &ndash; Actions'
date: 2013-02-12T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=1482
permalink: /first-steps-with-caliburn-micro-in-windows-phone-8-actions/
categories:
  - Windows Phone
tags:
  - Caliburn
  - MVVM
  - Windows Phone
---
In the <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-first-project" target="_blank">previous post</a> we’ve seen how&nbsp; to setup a Windows Phone application to use Caliburn Micro as a MVVM Framework. We’ve seen also the first conventions used by the toolkit, to connect together a View with its ViewModel and to bind a property to a control’s property. In this post we’ll see how to connect a control to an action, so that when the user interacts with the control the action is executed.

### Commands in Caliburn Micro

Usually to accomplish this task using MVVM Light or the standard XAML approach we use the **Command** property: we would define an object that implements the **ICommand** interface and define the operation that should be executed when it’s invoked. In the end, the object is connected with the **Command** property of a Button, for example, using binding. This approach is very repetitive, because we need to create an **ICommand** object for every operation while, in the end, what we really need is to define just the method that should be invoked. Caliburn Micro avoid this requirement by using, guess what, a naming convention, similar to the one we’ve seen to bind properties to a control. Basically, you just need to declare in your view model a method which name matches the name of the control.

For example, declare a **Button** like this:

<pre class="brush: xml;">&lt;Button Content="Show name" x:Name="ShowNameAction" /&gt;</pre>

Then, in your code, you’ll have a method like the following one:

<pre class="brush: csharp;">public void ShowNameAction()
{
    MessageBox.Show("Clicked");
}</pre>

Now if you launch the application and you press on the button you’ll see the message on the screen. The magical naming convention did the trick again!  <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-bottom-style: none; border-right-style: none; border-left-style: none" alt="Smile" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/wlEmoticon-smile.png?w=640" data-recalc-dims="1" />And what if we need a property to set the content of the button and, at the same time, an action to be triggered when the button is clicked? In this case we have a conflict with the naming convention rule: we can’t have in the view model both a method and a property with the same name.

In this case, we need to use the long syntax, that uses the behaviors available in the **System.Windows.Interactivity** namespace. For this reason, we need to add to our XAML the following namespaces:

_xmlns:i=&#8221;clr-namespace:System.Windows.Interactivity;assembly=System.Windows.Interactivity&#8221;  
xmlns:cal=&#8221;clr-namespace:Caliburn.Micro;assembly=Caliburn.Micro&#8221;  
_ 

Now you need to change the **Button** declaration like in the following sample:

<pre class="brush: xml;">&lt;Button x:Name="ShowName"&gt;
    &lt;i:Interaction.Triggers&gt;
        &lt;i:EventTrigger EventName="Click"&gt;
            &lt;cal:ActionMessage MethodName="ShowNameAction" /&gt;
        &lt;/i:EventTrigger&gt;
    &lt;/i:Interaction.Triggers&gt;
&lt;/Button&gt;</pre>

As a label for the button will be used a property called **ShowName** declared in the ViewModel; instead, the **ShowNameAction** method will be executed when the button is clicked. One advantage of using this syntax is that you are able to attach a method also to other events: by default, using the standard naming convention, a method is hooked with the default event of the control (in case of a Button, it’s the **Click** event). With this syntax, instead, it’s enough to change the **EventName** property of the **EventTrigger** with another one (for example, **DoubleTap**) to connect the method with that event.

Caliburn Micro provides also a short syntax to do the same:

<pre class="brush: xml;">&lt;Button cal:Message.Attach="[Event Click] = [Action ShowName]" /&gt;</pre>

The syntax is very simple: by using the **Message.Attach** attached property we define first the event we want to hook (with the keyword **Event** followed by the name of the event), then the name of the method that should be executed (with the keyword **Action** followed by the name of the method). The pros of the long syntax is that is supported by Blend, but it’s more verbose; the short syntax is easier to remember and to use, but Blend doesn’t like it. It’s up to you which syntax to use: personally I don’t use Blend too much, so losing the ability to see and interact with the button in Blend is not a big deal for me.

### Controlling the action

One of the most interesting features that is available in the command pattern is the **CanExecute** concept: every command can be controlled by the developer, so that he can enable or disable it according to a specific condition. The coolest thing is that the control that is bound with the command will automatically change his status according to this condition: if the command is disabled, for example, the button will be grayed and the user won’t be able to click on it.

Basically, the **CanExecute** action returns a boolean value, that determines if the command can be executed or not. Every time something changes, this action should be evaluated again. A classic example is a form, that shouldn’t be submitted until the user has filled all the required fields. By default, the submit button is disabled but, every time a field is filled by the user, the action should be evaluated again, because the status of the button can change.

Also in this case Caliburn Micro uses conventions to support this feature: you just need to create in your ViewModel a boolean property, with the same name of the method prefixed by the **Can** word. Here is how to extend the previous example, by adding a **Checkbox** in our page, which we’re going to bound to a boolean property. The button should be activated only when the checkbox is enabled.

Here is the XAML in the view:

<pre class="brush: xml;">&lt;StackPanel&gt;
    &lt;Button x:Name="ShowName" Content="Click me" /&gt;
    &lt;CheckBox x:Name="IsEnabled" Content="IsEnabled" /&gt;
&lt;/StackPanel&gt;</pre>

&nbsp;

And here is the code in our ViewModel:

<pre class="brush: csharp;">public class MainPageViewModel: PropertyChangedBase
{
    private bool isEnabled;

    public bool IsEnabled
    {
        get { return isEnabled; }
        set
        {
            isEnabled = value;
            NotifyOfPropertyChange(() =&gt; IsEnabled);
            NotifyOfPropertyChange(() =&gt; CanShowName);
        }
    }

    public bool CanShowName
    {
        get { return IsEnabled; }
    }

    public void ShowName()
    {
        MessageBox.Show("Clicked");
    }
}</pre>

&nbsp;

As you can see, other than defining the **ShowName** method (that is invoked when the **ShowName** button is clicked), we create a boolean property which name is **CanShowName.** In the end, we create a property called **IsEnabled**, that is connected to the checkbox in the view. This way, every time the checkbox is enabled, the **IsEnabled** property is set to true and the same happens for the **CanShowName** property. The only thing to highlight is that, every time we change the value of the **IsEnabled** property, we call the **NotifyOfPropertyChange** method not only on the property itself, but also on the **CanShowName** property. This way, every time the **IsEnabled** property changes, also the **CanShowName** property is evaluated again and the button’s status is changed according to the new value.

[<img title="action1" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="action1" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/action1_thumb.png?resize=200%2C333" width="200" height="333"  data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/action1.png)[<img title="action2" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px 0px 0px 15px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="action2" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/action2_thumb.png?resize=200%2C333" width="200" height="333"  data-recalc-dims="1" />](https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/action2.png)

### To be continued

In the next posts we’ll keep digging in using Caliburn Micro, to manage collections, tombstoning, messaging and navigation. For the moment, you can start playing with the sample project below to take a deeper look to what we’ve learned in this post.

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:c2d2faa7-edb1-4d35-b16a-914a46662bde" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/07/CaliburnMicro_Actions.zip" target="_blank">Download the sample project</a>
  </p>
</div>

****&nbsp;

**The Caliburn Micro posts series**

  1. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-theory/" target="_blank">The theory</a> 
      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-first-project/" target="_blank">The first project</a> 
          * Actions 
              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">Collections and navigation</a> 
                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-tombstoning/" target="_blank">Tombstoning</a> 
                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-advanced-navigation-and-deep-links/" target="_blank">Advanced navigation and deep links</a> 
                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-messaging/" target="_blank">Messaging</a> 
                              * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-using-launchers-and-choosers/" target="_blank">Using launchers and choosers</a> 
                                  * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-use-your-own-services-and-how-to-pass-data-between-different-pages/" target="_blank">Use your own services </a> 
                                      * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">The Application Bar</a> 
                                          * <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-pivot/" target="_blank">Pivot</a> 
                                              * [Lazy loading with pivot](http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-lazy-loading-with-pivot/) </ol>