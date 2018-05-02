---
id: 3182
title: How to override the theme of a custom Application Bar
date: 2013-05-03T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=3182
permalink: /how-to-override-the-theme-of-a-custom-application-bar/
categories:
  - Windows Phone
tags:
  - Windows Phone
---
I recently published a big update for an application I’ve developed a long time ago and that I “abandoned” until today: <a href="http://www.windowsphone.com/s?appid=b6c57aee-f526-e011-854c-00237de2db9e" target="_blank">Speaker Timer</a>. It’s a simple application for speakers, that can be used to track the time of your speech and to immediately see if you’re exceeding or not the time you’ve been assigned.

During the development I’ve encountered some issues to correctly manage the phone’s theme: as you can see from the screenshot <a href="http://www.windowsphone.com/s?appid=b6c57aee-f526-e011-854c-00237de2db9e" target="_blank">on the Store</a>, the application has a blue background and white texts. For this reason, everything works fine when the app is used with the dark theme but, if the user is using the light one, things start to get messy: texts starts to get hard to read and the overall user experience is really bad. In the following screenshots you can see, on the left, the application with the dark theme and, on the right, with the light theme: as you can see, since texts are in black over a blue background, they aren’t easy to read.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="session2" alt="session2" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/session2_thumb.png?resize=200%2C333" width="200" height="333" border="0" data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/session2.png)[<img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px 0px 0px 10px; display: inline; padding-right: 0px; border-width: 0px;" title="session1" alt="session1" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/session1_thumb.png?resize=200%2C333" width="200" height="333" border="0" data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/session1.png)

Luckily <a href="http://www.jeff.wilcox.name/" target="_blank">Jeff Wilcox</a>, that works in Microsoft as leader of the Windows Azure SDK and many other open source projects, has developed a really great library called **PhoneThemeManager**, that is described <a href="http://www.jeff.wilcox.name/2012/01/phonethememanager/" target="_blank">in this blog post</a>. The purpose of this library is really simple: with one line of code you’re able to force a specific theme in your application so that, if your application looks bad with another theme because you’ve used custom colors, you can force all the elements in the page to use your own theme.

Once you’ve installed the library using <a href="http://nuget.org/packages/PhoneThemeManager" target="_blank">NuGet</a>, you’ll just have to use the singleton **ThemeManager** class when your application starts (it can be inside the **App.xaml.cs** constructor or, for example, since Speaker Timer is built using Calibur Micro, I used it in the **OnStartup()** method of the boostrapper).

This class exposes some methods to change the theme: the most important ones are **ToDarkTheme()** and **ToLightTheme()**, that will override every color and force all the elements to use the theme that matches best with your application. In my case, since Speaker Timer looks bad with a light theme, I’ve forced the dark theme by including the following code:

<pre class="brush: csharp;">public App()
{
    // Standard Silverlight initialization
    InitializeComponent();

    ThemeManager.ToDarkTheme();

    // Show graphics profiling information while debugging.
    if (System.Diagnostics.Debugger.IsAttached)
    {
        // Display the current frame rate counters.
        Application.Current.Host.Settings.EnableFrameRateCounter = false;

        // Show the areas of the app that are being redrawn in each frame.
        //Application.Current.Host.Settings.EnableRedrawRegions = true;

        // Enable non-production analysis visualization mode, 
        // which shows areas of a page that are being GPU accelerated with a colored overlay.
        //Application.Current.Host.Settings.EnableCacheVisualization = true;
    }
}</pre>

The result, with this simple code, is that, regardless of the theme chosen by the user, the application will always look like in the first screenshot.

### 

### Managing a custom Application Bar

As I’ve already mentioned, Speaker Timer is developed using Caliburn Micro as a MVVM Framework: for this reason, I’m not using the native Application Bar, but <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">the one I’ve talked about in this post</a>, that supports binding and Caliburn Micro conventions. However, by using this bar I’ve found an issue connected to the user theme: even if I’ve forced the dark theme using Jeff Wilcox’s library, the application bar was still using the light theme. It’s not a big deal, but the biggest issue was that, when I moved to the other pages and then I came back to the first one, the Application Bar suddendly changed and became dark. This happens because the **PhoneThemeManager** library can’t immediately ovveride the Application Bar colors, so you can see the change only at a second time.

How to fix this situation? The library exposes two methods for this scenario: the first one is called **CreateApplicationBar()** and can be used to create a new application bar with the proper theme. This method is useful in case you’re creating and managing the Application Bar in the code behind and not in the XAML. But this is not my case, since Speaker Timer is built using MVVM and the custom application bar is added directly in the XAML. Here comes another method offered by the library, called **MatchOverriddenTheme()**: it’s an extension method applied to the **IApplicationBar** interface, so every custom Application Bar implemention should support it. When you call this method, the application bar colors are immediately overwritten, so you don’t have to deal with the graphic glitch I’ve mentioned before.

There’s only one important thing to keep in mind: this method should be called in the **Loaded** event of every page with an application bar and not in the page costructor, otherwise it won’t work properly, as you can see in the following screenshot.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="glitch" alt="glitch" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/glitch_thumb.png?resize=200%2C333" width="200" height="333" border="0" data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/glitch.png)

So, the first operation is to give a Name to your custom application bar using the **x:Name** property, like in the following sample:

<pre class="brush: xml;">&lt;bindableAppBar:BindableAppBar x:Name="AppBar"&gt;
    &lt;bindableAppBar:BindableAppBarButton Text="{Binding Source={StaticResource LocalizedStrings}, Path=LocalizedResources.Add}" IconUri="/Assets/Icons/Add.png" 
                                         x:Name="NewSession" 
                                         Visibility="{Binding Path=IsCheckModeEnabled, Converter={StaticResource NegativeBooleanToVisibilityConverter}}" /&gt;

    &lt;bindableAppBar:BindableAppBarButton Text="{Binding Source={StaticResource LocalizedStrings}, Path=LocalizedResources.Select}" IconUri="/Assets/Icons/Selection.png"
                                         x:Name="Selection"
                                         Visibility="{Binding Path=IsCheckModeEnabled, Converter={StaticResource NegativeBooleanToVisibilityConverter}}" /&gt;
&lt;/bindableAppBar:BindableAppBar&gt;</pre>

&nbsp;

Then, in the code behind, you should subscribe to the **Loaded** event and call the **MatchOverriddenTheme()** on the custom application bar, like in the following sample:

<pre class="brush: csharp;">public partial class MainPage : PhoneApplicationPage
{
    // Constructor
    public MainPage()
    {
        InitializeComponent();
        Loaded += (obj, args) =&gt;
            {
                AppBar.MatchOverriddenTheme();
            };
    }
}</pre>

That’s all! Happy coding!