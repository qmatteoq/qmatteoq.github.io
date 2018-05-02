---
id: 174
title: How to override the default color of a ProgressBar in a Windows 8 application
date: 2012-08-29T13:01:30+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=174
permalink: /how-to-override-the-default-color-of-a-progressbar-in-a-windows-8-application/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows 8
tags:
  - Windows 8
---
So you’re doing Windows 8 development and, very likely, your app needs to display some data. Rarely this data is immediately available: maybe you need to download it, or you have to parse it before displaying it. Here comes in help the **ProgressBar**, so that the user is aware that something is loading and that he needs to wait until the operation is completed.

By default, the ProgressBar automatically uses the accent color the user has selected for his Windows 8 installation: this isn’t always the best choice, because your application may use a background that doesn’t fit with the ProgressBar color.

The first thing you would try to do, as a developer, is to change the **Foreground** or the **Background** property of the control, but you’ll notice that the trick doesn’t work: the color of the ProgressBar doesn’t change.

This happens because the default ProgressBar color is defined in one of the default styles, so you have to override it to change it. To do this, simply add the following statement in the **ApplicationResources** defined in the **App.xaml** file:

<pre class="brush: xml;">&lt;ResourceDictionary.ThemeDictionaries&gt;
    &lt;ResourceDictionary x:Key="Default"&gt;
        &lt;x:String x:Key="ProgressBarIndeterminateForegroundThemeBrush"&gt;White&lt;/x:String&gt;
    &lt;/ResourceDictionary&gt;
&lt;/ResourceDictionary.ThemeDictionaries&gt;</pre>

In this example, the ProgressBar is displayed with a white color: instead of writing the color’s name, you can also put the hexadecimal code of the color.

Enjoy it!