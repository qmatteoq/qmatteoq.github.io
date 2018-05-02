---
id: 692
title: How to support Windows Phone 8 devices in a Windows Phone 7 application
date: 2012-12-18T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=692
permalink: /how-to-support-windows-phone-8-devices-in-a-windows-phone-7-application/
categories:
  - Windows Phone
tags:
  - Mangopollo
  - Windows Phone
---
As I’ve already highlighted in another blog post, Windows Phone 8 has introduced many new features; plus, lot of things have been changed under the hood: Microsoft has abandoned the old kernel, based on Window Mobile’s core, to introduce a new modern kernel, based on Windows 8’s one. As a consequence, also the architecture has deeply changed: no more Silverlight and .NET Framework, but Windows Runtime.

To allow developers to reuse as much as possible their code and skills, Microsoft has introduced two important features:

  * **Quirk mode:** the old APIs are automatically mapped to the new ones; this way, all the applications written for Windows Phone 7.x are able to run on a Windows Phone 8 device without changes in most of the cases. 
      * **.NET API for Windows Phone**, which is a set of libraries in the Windows Runtime that match the one that were part of the Silverlight for Windows Phone runtime. This way, when the developer decides to port his project to Windows Phone 8 he can focus on supporting the new features rather than rewriting the previous code to use the new Windows Runtime APIs. </ul> 
    &nbsp;
    
    The best way to use the Windows Phone 8 features is to update your project to the new release, by using a specific Visual Studio option that is displayed when you right click on a Windows Phone 7 project.
    
    This scenario, even if it’s the most powerful, requires to maintain two different projects if we want to keep supporting also Windows Phone 7 users, since Windows Phone 8 applications are not compatible with the previous versions. Sometimes this solution can be too expensive for the developer, especially if your application is very simple and can gain few benefits from the Windows Phone 8 features.
    
    In this post we’re going to see two solutions that will help us to maintain a single Windows Phone 7 project that will be adapted so that Windows Phone 8’s devices owners will be able to get the best from their devices.
    
    ### Supporting the new resolutions
    
    One of the most important Windows Phone 8 new features is new resolutions support: Windows Phone 7 supported just the 800&#215;480 resolution; as a consequence all the icons and images that are part of our project can degrade if they are displayed on a device with a higher resolution, like the Lumia 920 (1280&#215;768) or the HTC 8X (1280&#215;720).
    
    Now the Store accepts also Windows Phone 7.x applications with icons with higher resolution: you can embed in your project 336&#215;336 tiles and 99&#215;99 icons. This way, also Windows Phone 8 owners will be able to see images at the best possible quality.
    
    You just have to overwrite the original images, prepare a new XAP and sumit a new update of your application to the store.
    
    ### How to use some of the new Windows Phone 8 features
    
    By using a mechanism called **reflection** developers are able to use some of the new Windows Phone 8 features in a Windows Phone 7.x application, in case it’s running on a Windows Phone 8 device.
    
    What is reflection? This mechanism allows developers to use methods, properties and classes that are stored in a DLL library without adding it as a reference in a Visual Studio project. With this procedure you’ll be able to interact with some of the new Windows Phone 8 libraries from a Windows Phone 7.x project, even if Visual Studio isn’t able to show them.
    
    You’ll be able to use the following features:
    
      * The new launchers and choosers (for example, to add a new appointment to the calendar or download a map). 
          * The new tiles templates and sizes. </ul> 
        &nbsp;
        
        Using the reflection requires to write more complicated code compared to the approach you use when you have a reference in your Visual Studio project: for example, you don’t have access to Intellisense and you have to remember the exact name of every method and class you’re going to use.
        
        For this reason Rudy Hyun, a Windows Phone MVP, has developed a library called **Mangopollo** (the strange name comes from the fusion between Mango, the Windows Phone 7.5 codename, and Apollo, the Windows Phone 8 codename), which features a set of classes and methods that act as a wrapper to the reflection, so that you can interact with the Windows Phone 8 features using the usual approach.
        
        Mangopollo is an open source project that is available both on <a href="http://mangopollo.codeplex.com/" target="_blank">Codeplex</a> and NuGet in two versions: **Mangopollo** and **Mangopollo.Light.** The difference is that the light version contains only the libraries to interact with the new tiles, so that it can be used also in a background agent project. Mangopollo already supports Windows Phone 7.8: it’s possible to use it to interact with the new tiles that are available on Windows Phone 7.8 devices.
        
        The simplest way to add Mangopollo to your project is by using NuGet: right click on your Windows Phone project, choose **Manage NuGet packages** and look for and install the package called **Mangopollo.**
        
        Now you’ll be able to use the new launchers and choosers (that are stored inside the **Mangopollo.Tasks** namespace) and the new tiles (which classes are inside the **Mangopollo.Tiles** namespace).
        
        Let’s see to examples on how to use this library: in the first one we’re going to use the **SaveAppointmentTask** launcher, so that the user will be able to add a new appointment to the calendar.
        
        ### Use the launchers
        
        Here is how to use the **SaveAppointmentTask** launcher:
        
        <pre class="brush: csharp;">private void OnCreateAppointmentClick(object sender, RoutedEventArgs e)
{
    if (Utils.IsWP8)
    {
        SaveAppointmentTask task = new SaveAppointmentTask();
        task.Subject = "Windows Phone Developer Day";
        task.StartTime = new DateTime(2012, 12, 5);
        task.IsAllDayEvent = true;
        task.Show();
    }
    else
    {
        MessageBox.Show("This is Windows Phone 7!");
    }
}</pre>
        
        The first thing to do to use the new features is to check if the app is running on a Windows Phone 8 device, otherwise we would get an exception. We can perform this task by using the **Utils.IsWP8** property, which is a simple Boolean.
        
        In case our code is running on a Windows Phone 8 device we create a new **SaveAppointmentTask** object and we define some properties that describe the appointment, like the title and the date. The definition of this task is the same that is available in the Windows Phone 8 APIs: the name of the methods and properties are the same that you can find in the new SDK.
        
        In the end, we invoke the launcher by calling the **Show** method. Now let’s test the application using both the emulators: the Windows Phone 8 one (at this point, it doesn’t matter which resolution you use) and the Windows Phone 7.1 one.
        
        In the first case we’ll see the launcher, that will ask to the user to fill all the missing information about the appointment; in the second case, instead, we’ll see a warning message we’ve defined in the code.
        
        [<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image_thumb2.png?resize=204%2C337" width="204" height="337"  data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image2.png)[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image_thumb3.png?resize=204%2C337" width="204" height="337"  data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image3.png)
        
        ### Play with the tiles
        
        Windows Phone 8 now supports three new tiles sizes and three new templates. The new sizes are:
        
          * **Small:** the tile size is ¼ of the original size. 
              * **Medium:** it’s the standard size. 
                  * **Wide:** it’s the rectangular size, it’s like having two medium tiles one after the other. </ul> 
                &nbsp;
                
                The supported templates, instead, are:
                
                  * **Flip**: it’s the only template that was available in Windows Phone 7; information (texts and pictures) are placed on the front and on the back of the tile, which is periodically flipped to show all of them. 
                      * **Cycle**: you can include up to 9 images, that are displayed in rotation. 
                          * **Iconic**: it’s used to create tiles with the same look & feel of the native ones, like Mail or Messages. You can use it to show an icon, a counter or text. </ul> 
                        &nbsp;
                        
                        As a developer, other than choosing the template, you’ll be able also to interact with the tile in different ways, according to the size: for example, we can show some content only in case the user is using a bigger size.
                        
                        Here is a sample code that uses the **Iconic** template, which is described by the **IconicTileData** class, to create a secondary tile.
                        
                        <pre class="brush: csharp;">private void OnCreateWideTileClick(object sender, RoutedEventArgs e)
{
    if (Utils.CanUseLiveTiles)
    {
      var tile = new IconicTileData
      {
          Title = "WP Day",
          Count = 8,
          BackgroundColor = Colors.Transparent,
          IconImage = new Uri("/Assets/Tiles/IconicTileMediumLarge.png", UriKind.Relative),
          SmallIconImage = new Uri("/Assets/Tiles/IconicTileSmall.png", UriKind.Relative),
          WideContent1 = "WP Developer Day",
          WideContent2 = "use Windows Phone 8 features",
          WideContent3 = "on Windows Phone 7 apps"
      }.ToShellTileData();

      ShellTileExt.Create(new Uri("/MainPage.xaml?Id=5", UriKind.Relative), tile, true);
    }
    else
    {
      MessageBox.Show("This is Windows Phone 7");
    }
}</pre>
                        
                        <strike>The code is placed inside a statement similar to the one that you’ver seen in the previous example: we’re going to execute the operation only if the app is running on a Windows Phone 8 device.</strike>
                        
                        **UPDATE:** As Alessandro, a reader of this blog, correctly pointed me it’s better to use the **Utils.CanUseLiveTiles** property for this purpose: this way, the code will work also on a Windows Phone 7.8 device. Instead, it’s important to keep using the **Utils.IsWP8** property for launchers and choosers because they are available only in Windows Phone 8.
                        
                        If you already had the chance to work with tiles the code will be familiar: we create a new **IconicTileData** object (which describes the iconic template) and we set up some properties that define the information displayed on the tile. I would like to highlight that we set also three properties (identified by the prefix **WideContent**), that are used only in case the user has chosen the wide size for the tile of our application.
                        
                        The secondary tile is created using the **Create** method of the **ShellTileExt** class, that works exactly like the native **ShellTile** class but that is part of the **Mangopollo** library: in fact, it supports some parameters specific for the new tiles, like (other than deep link and the template) a Boolean which tell to the OS if the tile is going to support or not the wide tile.
                        
                        Also in this case if we’re going to run the code on the Windows Phone 8 emulator everything will be just fine: the app will close and the new secondary tile will be created. If we play with the various tiles formats we can notice that, once we set the wide size, the information that we have specified in the **WideContent** properties will be displayed. Instead, if we run the code on the Windows Phone 7 emulator we’ll get the usual warning message.
                        
                        [<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image_thumb4.png?resize=204%2C337" width="204" height="337"  data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2012/12/image4.png)
                        
                        And what if we would like to update the tile? The task is very easy:
                        
                        <pre class="brush: csharp;">private void OnUpdateTileClick(object sender, RoutedEventArgs e)
{
    if (Utils.IsWP8)
    {
        IconicTileData tile = new IconicTileData
                                  {
                                      WideContent1 = "This is the new content",
                                      WideContent2 = "The tile has been updated",
                                      WideContent3 = "with success"
                                  };

        Uri navigationUri = new Uri("/MainPage.xaml?Id=5", UriKind.Relative);
        ShellTile.ActiveTiles.FirstOrDefault(x =&gt; x.NavigationUri == navigationUri).Update(tile);
    }
}</pre>
                        
                        We just define a new template using the **IconicTileData** class and we set only the properties that describe the information that we want to change compared with the original template definition (in the example, we change just the text displayed in the wide tile).
                        
                        After that the update procedure is the same that we’ve learned to use in Windows Phone 7.5, since it’s indipendent from the template we use: we retrieve the tile to update by using the **ShellTile.ActiveTiles** collection according to the deep link uri, then we call the **Update** method, passing as parameter the updated template. Since this method accepts a **ShellTileData** object, which is the base class which every template inherits from, we can pass any supported template object, even the new ones that are defined in Mangopollo.
                        
                        The code we’ve just seen to update a tile can be used also in a background agent, so that the tile can be periodically updated: the only thing you’ll have to remember, as I’ve anticipated in the introduction of this post, is to install the **Mangopollo.Light** package in your agent’s project.
                        
                        ### In the end
                        
                        In this post we have seen how to use some of the new Windows Phone 8 features in a Windows Phone 7 application, so that you’ll be able to improve the user experience for new device’s owners without forcing you to keep two different projects.
                        
                        This is a very good solution, even if it has a lot of limitations compared to a real upgrade to a Windows Phone 8 project: the new features, in fact aren’t only some new launchers or the new tile sizes, but there are a lot more like NFC, Wallet, Lens Apps that are not available by using one of the strategies that are described in this post.
                        
                        It’s up to you (and to your application’s category) to choose the best path!
                        
                        <div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:9b259a78-5371-46be-bbee-fd482a45bae9" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; padding-right: 0px">
                          <p>
                            <a href="http://wp.qmatteoq.com/wp-content/uploads/2012/12/Mangopollo6.zip" target="_blank">Here is a link to download a project sample.</a>
                          </p>
                        </div>