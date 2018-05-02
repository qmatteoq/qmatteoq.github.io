---
id: 452
title: How to deal with WebView and overlaying elements in Windows 8
date: 2012-12-10T15:00:07+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=452
permalink: /how-to-deal-with-webview-and-overlaying-elements-in-windows-8/
categories:
  - Windows 8
tags:
  - Windows 8
---
The WebView control is very useful to display HTML pages and elements into a Windows Store application. For example, I use in my blog’s application to display the content of a post, formatted to be easily read on a tablet using the Instapaper engine.

If you have played a bit with the WebView control you would have find that it has a particular behavior: no matter how many XAML controls you are going to place over the WebView, they will always be displayed below the view. This is not a bug, but a specific security feature: since in the past developers were able to do dirty tricks by simply placing an hidden web view and by executing dangerous Javascript code in the back, Microsoft has chosen not to allow this anymore. No matter how many efforts you’ll spend trying to find a workaround, you won’t be able to place a WebView below another control (neither fully or partial).

During the development of my blog’s application this security feature caused me some troubles: in the latest version I’ve developed I’ve added support both for Italian and English. The application automatically sets the correct language according to the language of the operating system, but the user is able to change it from the Settings panel. For this reason I’ve added an item in the Settings panel called Language: when the user taps on it, a flyout panel with two radio buttons is displayed.

The problem is that, if the user tries to change the language while he’s reading a post, the flyout goes behind the WebView and he’s not able to fully see it. You can see what I’m talking about in the image below:

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image_thumb.png?resize=540%2C304" alt="image" width="540" height="304" border="0" data-recalc-dims="1" />](https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image.png)

How to fix it? Since, as I’ve already said, there’s absolutely no way to change the overlay of the WebView control, the only available workaround is to hide the WebView until the user has done interacting with the controls and the elements that are placed over it.

In my case, I needed to hide the WebView every time the Settings panel was displayed and to show it back as soon as the Settings panel was closed. Since there’s no way, within a Windows Store apps, to know when the charm bar is activated, we have to find another approach: by managing the page’s focus.

The page exposes two events, **LostFocus** and **GotFocus**: the first one is invoked when the current page lost its focus (because, for example, the user is interacting with the settings flyout), the second one instead when the focus is reassigned to the app.

You can subscribe to these two events in the XAML and, more precisely, in the page’s declaration, like in the following example:

<pre class="brush: xml;">&lt;common:LayoutAwarePage
    x:Name="pageRoot"
    x:Class="qmatteoq.com.Views.DetailPage"
    LostFocus="pageRoot_LostFocus" GotFocus="pageRoot_GotFocus"&gt;</pre>

Once you’ve done it, you can mange these events in the code behind. In my case I simply change the WebView’s control (which, in my app, is called **PostDetail**) visibility according to the focus:

<pre class="brush: csharp;">private void pageRoot_LostFocus(object sender, RoutedEventArgs e)
{
    PostDetail.Visibility = Visibility.Collapsed;
}

private void pageRoot_GotFocus(object sender, RoutedEventArgs e)
{
    PostDetail.Visibility = Visibility.Visible;
}</pre>

And here is the final result: when the settings panel is opened the WebView is hided. Plus, I’ve added a message to notify the user that the post is just hidden and that will reappear when the settings panel will be closed.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image_thumb1.png?resize=540%2C304" alt="image" width="540" height="304" border="0" data-recalc-dims="1" />](https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image1.png)