---
id: 99
title: How to display GIF images in a Windows Phone application
date: 2012-07-27T10:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=99
permalink: /how-to-display-gif-images-in-a-windows-phone-application/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows Phone
tags:
  - Windows Phone
---
Some time ago I was talking with a friend that needed a special requirement for his application: displaying GIF images downloaded from the web. This image format is not supported by Silverlight nor Windows Phone: if you try to download and display a GIF image inside an Image control you’ll get an exception.

So I searched a little bit on the Internet to find a possible solution and I came out with <a href="http://blogs.msdn.com/b/jaimer/archive/2010/11/23/working-with-gif-images-in-windows-phone.aspx" target="_blank">ImageTools</a>, a third party library that contains many converters and tools to convert one image from a format to another on the fly. One of the supported scenario is GIF images conversion (that can be animated, too) for Windows Phone.

Let’s see how to use it: the first thing is to add the library to your project and, as usual, the easiest way to do is with NuGet.

<img style="background-image: none; padding-top: 0px; padding-left: 0px; padding-right: 0px; border-width: 0px;" src="https://i1.wp.com/qmatteoq.tostring.it/UserFiles/uploaded/qmatteoq/SNAGHTML89ae30b.png?resize=489%2C327" alt="" width="489" height="327" border="0" data-recalc-dims="1" />

To use this library to achieve our goal we need to use two components:

  * The first is a control called **AnimatedImage**, which is going to replace the standard Silverlight **Image**control. It’s the container that will display the image.
  * The second is a converter called **ImageConverter**, which takes care of converting the image from one format to another.

Both objects are part of the **ImageTools.Controls** namespace, that should be declared in the XAML page where we’re going to use them.

_xmlns:imagetools=&#8221;clr-namespace:ImageTools.Controls;assembly=ImageTools.Controls”_

Once you have imported the namespace, you can add the converter as a resource for the page or, if you prefer, for the global application. Here is an example on how to do it for a single page:

<pre class="brush: xml;">&lt;phone:PhoneApplicationPage.Resources&gt;
    &lt;imagetools:ImageConverter x:Key="ImageConverter" /&gt;
&lt;/phone:PhoneApplicationPage.Resources&gt;</pre>

If you want to make this resource globally available, you just have to declare it inside the **Application.Resources** section inside the **App.xaml** file.

Now you are ready to insert the **AnimatedImage** control into your page: you simply have to bind the **Source** property with a **Uri** object (which contains the URL of the gif image) and apply the **Image** converter.

Here is an example of the XAML declaration:

<pre class="brush: xml;">&lt;StackPanel&gt;
    &lt;imagetools:AnimatedImage x:Name="Image" Source="{Binding Path=ImageSource, Converter={StaticResource ImageConverter}}" /&gt;
&lt;/StackPanel&gt;</pre>

And here, instead, is how the **ImageSource** property is defined in the code:

<pre class="brush: csharp;">ImageSource = new Uri("http://www.nonstopgifs.com/animated-gifs/games/games-animated-gif-002.gif", UriKind.Absolute);</pre>

We’re almost done: the trick to make the “magic” working is to use one of the decoders that are available in the **ImageTools** library: these decoders take care of the conversion process and they should be initialized when the application or the page is created, specifying which is the image format to use.� We are using this library for GIF conversion, so here is how to register the GIF decoder.

<pre class="brush: csharp;">public MainPage()
{
    InitializeComponent();
    ImageTools.IO.Decoders.AddDecoder&lt;GifDecoder&gt;();
}</pre>

As I anticipated in the beginning of this post, this control supports animated GIFs too: if you try the example GIF used in this post you can see it by yourself.