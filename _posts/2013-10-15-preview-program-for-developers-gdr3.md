---
id: 4792
title: 'Preview Program for Developers &amp; GDR3'
date: 2013-10-15T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=4792
permalink: /preview-program-for-developers-gdr3/
categories:
  - Windows Phone
tags:
  - GDR3
  - Windows Phone
---
Yesterday Microsoft has announced two big news for all the Windows Phone users and developers. Let’s start to dig them!

### 

### GDR3 announcement

Microsoft has officialy <a href="http://blogs.windows.com/windows_phone/b/windowsphone/archive/2013/10/14/announcing-our-third-windows-phone-8-update-plus-a-new-developer-preview-program.aspx" target="_blank">revelead the third Windows Phone 8 update</a>, called Update 3 (or GDR3). Like GDR2, the focus of this update are consumers, since it adds a coulpe of much requested features like:

  * A new driving mode, that is automatically enabled when you’re driving and that automatically turns off all the notifications so that you can’t be distracted. 
      * A new option in Settings to lock screen rotation. 
          * Custom ringtones support for all the notification types, like mails and text messages. 
              * Improved task switcher, with an option to close suspended apps. 
                  * Now, during the first wizard, you’ll be able to restore a backup also using a Wi-Fi connection.</ul> 
                Anyway, the most important new feature, which is interesting also for developers, is the support for a new resolution, **which is 1080p (1080&#215;1920)**. Devices with this resolution will be able to display a new column of tiles in the main screen (while, even if some rumors stated the opposite, phones with lower resolution will keep the current layout with two columns). From a developer point of view, 1080p and 720p (which is one of the already supported resolutions) are the same: they have the same aspect ratio (16:9), which is different from the aspect ratio of the other two resolution (15:9). What does it mean? That it’s important to test your app with this new resolution if you want to deliver to your users the best experience: since the aspect ratio is different, some images and look could look different. In some cases, you’ll need to load specific images for the new resolution and to use a fluid layout, which is able to automatically adapt to the screen’s size. If you want to learn more about this topic, <a href="http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj206974(v=vs.105).aspx" target="_blank">you can take a look at this article</a> from the official MSDN documentation.
                
                This new resolution will also make you reconsider the choice to not update your applications to Windows Phone 8: since 1080p has a different aspect ratio, 7.8 applications will look “cut”, with an empty space at the top of the screen. So, if you want that your app looks best on every device, you’ll have to upgrade it to Windows Phone 8, which is able to support all the resolutions.
                
                You’ll be able to test your app against the new resolution very soon: Microsoft is going to release a SDK update, that will introduce new emulators aligned with GDR3. From a developer point of view, there are also other other changes, that are detailed in the <a href="http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj206940(v=vs.105).aspx#BKMK_wp8_gdr3" target="_blank">MSDN documentation</a>:
                
                  * The memory cap for background audio agents has been raised from 20 to 25 MB on devices with 1 GB of RAM or more. 
                      * The memory cap for apps has been raised to 570 MB on devices with 2 GB of RAM. 
                          * There’s a new Uri scheme (ms-settings-screenrotation: ) to quickly access to the Screen Rotation setting. 
                              * Icon and tile of your application are now used also in the updated Task switcher view. 
                                  * You can set a custom sound for toast notifications, using reflection. 
                                      * The Internet Explorer’s viewport has been updated: this can impact applications that use WebBrowser control or mobile websites.</ul> 
                                    If you need to check if the current phone is running GDR3 or not, you’ll need to use the **Environment.OSVersion.Version** property: a GDR3 device will return 8.0.10492.
                                    
                                    ### 
                                    
                                    ### Preview Program for Developers
                                    
                                    But what if you want to test your apps on a real GDR3 device? Here comes the most interesting news for developers: Microsoft has opened a **Preview Program for Developers**, which allows to get early access to new Windows Phone updates, without having to wait for carrier’s approval. This way, developers can get their apps ready when the new update will be publicly distributed to every user. To be part of the program you’ll just need:
                                    
                                      * A developer unlocked device. Phones can be unlocked using the Windows Phone Developer Registration tool that comes with the SDK. If you’re a registered developer, you’ll be able to unlock up to 3 phones; otherwise, you can unlock just one phone. 
                                          * A paid developer account, which allows you to publish your apps on the Store, or a free <a href="http://apps.windowsstore.com/" target="_blank">App Studio</a> account.</ul> 
                                        Once you have met this requirement, you can go to <a href="https://dev.windowsphone.com/en-us/featured/update3" target="_blank">this page</a>, where you can review the requirements and the program’s conditions: you’ll find a link that will redirect you to an application in the Store. You’ll simply have to download it, run it and, after accepting terms and conditions, enable the preview program. From now on, when you look for new updates in the Settings hub, you’ll directly contact Microsoft’s servers to get the latest version of the OS: if you do it right now, the GDR3 download and installation process will start. Cool, isn’t it?
                                        
                                        In the end, here are some important things to know about the program:
                                        
                                          * The GDR3 version that is distributed with the Preview program is final, but it’s just the OS update offered by Microsoft. Diver and firmware updates from the phone’s vendor will be provided with the official GDR3 release. 
                                              * GDR3 can be installed on any phone, but it should already have installed GDR2. The minimum required OS version is 8.0.10322.71. 
                                                  * There’s no turning back: after you’ve installed GDR3, you’ll be able to install future updates but not to go back to the previous state. 
                                                      * The update is incremental, so you won’t lose any data.</ul>