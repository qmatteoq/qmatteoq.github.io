---
id: 138
title: SQLite for WinRT is now available on Visual Studio Gallery
date: 2012-08-22T10:00:36+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=138
permalink: /sqlite-for-winrt-is-now-available-on-visual-studio-gallery/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows 8
tags:
  - SQLite
  - Windows 8
---
Thanks to <a href="http://timheuer.com/blog/archive/2012/08/07/updated-how-to-using-sqlite-from-windows-store-apps.aspx" target="_blank">Tim Heuer</a> I’ve found that there’s an easier way to add SQLite to your Windows 8 application rather than the manual one described <a href="http://wp.qmatteoq.com/using-sqlite-in-your-windows-8-metro-style-applications/" target="_blank">in my post</a>. Now SQLite for WinRT is available as a Visual Studio extension, you can simply add it by searching for the keyword **sqlite:** the name of the extension is **SQLite for Windows Runtime**. Once you’ve installed it and restarted Visual Studio, you can now simply add SQLite to your application by adding a reference to two of the available libraries in the **Extensions** panel: **Microsoft Visual C++ Runtime Package** and **SQLite for Windows Runtime**. You can access to the panel by right clicking on the project, choosing **Add reference** and selecting **Extensions** from the **Windows** menu.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="SNAGHTML12499e4" src="https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/SNAGHTML12499e4_thumb.png?resize=477%2C290" alt="SNAGHTML12499e4" width="477" height="290" border="0" data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/SNAGHTML12499e4.png)

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="SNAGHTML125199d" src="https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/SNAGHTML125199d_thumb.png?resize=477%2C328" alt="SNAGHTML125199d" width="477" height="328" border="0" data-recalc-dims="1" />](https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/SNAGHTML125199d.png)

Be aware that, since the library uses C++ extensions, you can’t compile the application using as a platform the **Any CPU** option, but you should target a specific platform. To do that, open the **Configuration manager** from the **Build** menu and set the platform to one of the available options (x86, x64 or ARM).

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="SNAGHTML1270275" src="https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/SNAGHTML1270275_thumb.png?resize=477%2C300" alt="SNAGHTML1270275" width="477" height="300" border="0" data-recalc-dims="1" />](https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/SNAGHTML1270275.png)

If you don’t do this, you’ll see a warning sign on the two references **Microsoft Visual C++ Runtime Package** and **SQLite for Windows Runtime** and the project won’t compile.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/image_thumb.png?resize=295%2C91" alt="image" width="295" height="91" border="0" data-recalc-dims="1" />](https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/image.png)

And… you’re done! No more manually downloading the dll library and copying it to the Visual Studio project: you can simply go on, install **sqlite-net** and use the same code you’ve learned to use <a href="http://wp.qmatteoq.com/using-sqlite-in-your-windows-8-metro-style-applications/" target="_blank">in my first post</a>.

Thanks to <a href="http://timheuer.com/blog/" target="_blank">Tim Heuer</a> for the tip and… happy coding!