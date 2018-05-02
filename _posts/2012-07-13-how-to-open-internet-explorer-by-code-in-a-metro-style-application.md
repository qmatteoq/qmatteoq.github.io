---
id: 54
title: How to open Internet Explorer by code in a Metro style application
date: 2012-07-13T10:00:51+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=54
permalink: /how-to-open-internet-explorer-by-code-in-a-metro-style-application/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows 8
tags:
  - Windows 8
---
Recently I’ve worked on a Windows 8 application to promote my blog: people can use it to read my posts within the application and to be notified (using both toast and tile notifications) every time a new post is published. The application uses a web view to display the detail of a post: the problem came when I had to manage the snapped view, since there isn’t enough space on screen to display a web view.

During the pre certification lab that I’ve attended in Microsoft Italy the trainer gave me a good tip: in case the application is snapped, the post detail should be opened using Internet Explorer and not inside the application. In fact, in this scenario, Internet Explorer will open in filled mode and the user will be able to read the post without any difficulty.

In a Windows Phone application this behavior is easy to implement, thanks to the **WebBrowserTask** launcher. How to do the same in a Windows 8 application?

My first challenge was to identify the current visual status of the application: I need, in fact, a different behavior according to the visual status. When the application is in “standard” mode I use a GridView to display all the posts: when the user taps on one of them, it should be opened inside the application; when the application, instead, is in snapped mode the GridView is replacted by a ListView: in this scenario when the user taps on one of the posts it should be opened inside Internet Explorer.

For this purpose WinRT features a class called **ApplicationView**, that is part of the **Windows.UI.ViewManagement** namespace. This class exposes, as a static object, the **Value** property, which type is **ApplicationViewState**: it’s an enumerable, that offers a value for each one of the available visual states (that are **Snapped, Filled, FullScreenLandscape** and **FullScreenPortrait**). The value of this property matches the current visual state of the application. This is exactly what I needed to manage the different behavior.

The second step is to find the **WebBrowserTask** corresponding in the WinRT world: please welcome the **Launcher** static class, that is part of the namespace **Windows.System.Launcher.** This classes exposes the method **LaunchUriAsync**, that does exactly what we need: it opens a new page in the Internet Explorer Metro application, according to the **Uri** that is passed as parameter.

Here is an example where we apply this code in the **SelectionChanged** event handler, that is raised every time the user taps on an item of a GridView or a ListView.

<pre class="brush: csharp;">private void Selector_OnSelectionChanged(object sender, SelectionChangedEventArgs e)
{
    if (ApplicationView.Value == ApplicationViewState.Snapped)
    {
        Launcher.LaunchUriAsync(new Uri(e.AddedItems[0]));
    }
    else
    {
        Frame.Navigate(typeof (DetailPage), e.AddedItems[0]);
    }
}</pre>