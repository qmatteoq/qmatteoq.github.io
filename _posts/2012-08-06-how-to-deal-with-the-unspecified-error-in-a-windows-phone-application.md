---
id: 120
title: How to deal with the “Unspecified error” in a Windows Phone application
date: 2012-08-06T10:00:12+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=120
permalink: /how-to-deal-with-the-unspecified-error-in-a-windows-phone-application/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows Phone
tags:
  - Windows Phone
---
While developing and testing a Windows Phone application you may have to deal with a really annoying problem: your application crashes and the only error reported by Visual Studio is a generic Unspecified error.

Visual Studio itself isn’t of any help, because the error is intercepted by the global error handler available in the **App.xaml.cs** file, so you don’t have a way to understand where exactly the problem occured.

The solution is easy to understand, even if it’s not easy to find: this error is raised usually when your XAML contains a property with an invalid value. For example, when I had to deal with this error I found that, by mistake, the **Margin** property of a control contained the value **0k, 0, 0, 15,** that is obviously not correct (the margin property accepts only numeric values).

To help understanding where your application may contain such an error, keep track of the the time when Visual Studio intercept the &#8220;Unspecified error” exception: usually, this error is raised only when you navigate towards a page that has this problem.

Good debugging!