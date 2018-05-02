---
id: 359
title: 'Async pack for Visual Studio 2012: why it&#8217;s useful also for Windows Phone 8 projects'
date: 2012-11-19T10:00:07+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=359
permalink: /async-targeting-pack-for-visual-studio-2012-why-its-useful-also-for-windows-phone-8-projects/
categories:
  - Windows Phone
tags:
  - Async
  - Windows Phone
---
Microsoft has recently released the **Async Pack** for Visual Studio 2012: it’s a very helpful package and it’s the successor of the old **Async CTP** that was available for Visual Studio 2010. Its purpose is to add support to the new asynchronous pattern introduced in C# 5 (based on the **async** and **await** keywords) also to “old” technologies that, since are still based on C# 4, couldn’t make use of it. Some example of these technologies are Silverlight or Windows Phone 7: basically, we’re talking about all the stuff that relies on .NET Framework 4, since the new .NET Framework 4.5 (that has been shipped with Windows 8) is based on C# 5.

Installing it is very easy: you simply have to use NuGet, by right clicking on the project and choosing **Manage Nuget packages**. First you have to change the filter that is applied by default: since the async package is still in beta, you have to change the default filter at the top of the window from **Stable only** to **Include prerelease.** After that, you can search for the keyword **Bcl.Async** and install the **Async for .NET Framework 4, Silverlight 4 and 5 and Windows Phone 7.5** package. Automatically NuGet will resolve all the references and it will install also the **Microsoft.Bcl** package for you.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/11/image_thumb.png?resize=438%2C292" alt="image" width="438" height="292" border="0" data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/11/image.png)

But why this package should be interesting also for developing Windows Phone 8 applications? Windows Phone 8 is a new technology, based on a subset of the Windows Runtime, so the async and await support is built in.

The answer is that, due to maintain compatibility with the old platform (that didn’t have async and await support), some old classes doesn’t offer a way to use them with the new pattern. Take the **WebClient** class for example, which is very popular to make network operations (like downloading a file). Unlike the other WinRT APIs (like the ones to work with storage or sensors), it still relies on the old callback pattern: you register for an event handler that is invoked when the operation is finished, you launch the network operation and, inside the event handler (that is the callback), you manage all the operations that should be done with the result (like storing the file in the local storage or displaying it).

Here is an example:

<pre class="brush: csharp;">WebClient client = new WebClient();
client.DownloadStringCompleted += (obj, args) =&gt;
                                      {
                                          if (args.Error == null)
                                          {
                                              MessageBox.Show(args.Result);
                                          }
                                      };
client.DownloadStringAsync(new Uri("http://wp.qmatteoq.com"));</pre>

We download the HTML content of the page of this blog and, when the operation is completed, if no errors have occurred, we display it using a MessageBox.

This approach is not so easy to write and understand like with the async and await keywords, basically because the code we wrote is not sequential: the code that is written after the **DownloadStringAsync** method is executed right after we have issued the network operation but, at a certain point, the execution will stop and the application will start running through the **DownloadStringCompleted** event handler.

Plus, probably we’ll have some sort of “spaghetti” code: we’re using the callback approach to manage network operations but, for example, we’ll have to use the async and await pattern if we want to use the new storage APIs to save the HTML in a file in the local storage.

Here comes the **Async pack**: other than adding support to async and await (that is not needed, since Windows Runtime for Windows Phone already supports it) it adds a bunch of extension methods, that add to many “old style” classes new methods to work with **Task** objects and the new async pattern.

Some examples of classes that can take benefit of these extension methods are **WebClient**, **HttpWebRequest** and **Stream.** This way we change the previous example to use the new approach:

<pre class="brush: csharp;">private async void OnDownloadAsync(object sender, RoutedEventArgs e)
{
    WebClient client = new WebClient();
    string result = await client.DownloadStringTaskAsync("http://wp.qmatteoq.com");
    MessageBox.Show(result);
}</pre>

Or, for example, we can start a network request using the **HttpWebRequest** class in a much simpler way:

<pre class="brush: csharp;">private async void OnDownloadAsync(object sender, RoutedEventArgs e)
{
    HttpWebRequest request = HttpWebRequest.CreateHttp("http://wp.qmatteoq.com");
    HttpWebResponse webResponse = await request.GetResponseAsync() as HttpWebResponse;
    MessageBox.Show(webResponse.StatusDescription);
}</pre>

Or, in the end, we can use the **Stream** extension methods to get the content of the previous **HttpWebRequest** example:

<pre class="brush: csharp;">private async void OnDownloadAsync(object sender, RoutedEventArgs e)
{
    HttpWebRequest request = HttpWebRequest.CreateHttp("http://wp.qmatteoq.com");
    HttpWebResponse webResponse = await request.GetResponseAsync() as HttpWebResponse;
    Stream responseStream = webResponse.GetResponseStream();

    using (StreamReader reader=new StreamReader(responseStream))
    {
        string content = await reader.ReadToEndAsync();
        MessageBox.Show(content);
    }
}</pre>

For all of the reasons I’ve explained in this post, I strongly suggest you to add the **Async targeting pack** to every Windows Phone 8 project you’re going to develop. Have fun!