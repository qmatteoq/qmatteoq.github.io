---
id: 3762
title: Some clarifications about Windows Phone, Windows 8.1 and Visual Studio 2013
date: 2013-06-27T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=3762
permalink: /some-clarifications-about-windows-phone-windows-8-1-and-visual-studio-2013/
categories:
  - Windows 8
  - Windows Phone
tags:
  - Windows 8
  - Windows Phone
---
Yesterday Microsoft has released the much anticipated Windows 8.1 Preview, together with the first Preview of Visual Studio 2013. The Windows team worked really hard in the latest month and I think that the results are really good: Windows 8.1 is a big step forward from Windows 8, both for tablet users and desktop users. Tablet users can now enjoy many more nice features, like the new live tiles, the new snapped mode, the new control panel (that now can be used to customize almost every option without having to use the old desktop control panel). Desktop users, instead, should now find a more familiar environment, thanks to the return of the start menu, the option to boot the pc directly on the desktop and a new Apps screen, that lists all the apps installed on your computer and that replaces the old Start menu approach.

Windows 8.1 is interesting also for developers: with something like 5000 new APIs, I think that we’ll have much fun in the next months  <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/06/wlEmoticon-smile1.png?w=640" data-recalc-dims="1" />Actually, the best way to be updated on what’s going at BUILD is to pay a visit to <a href="http://channel9.msdn.com/" target="_blank">the Channel 9</a> website, that will make available all the recordings of the sessions that will cover all the new Windows 8.1 features for developers. I think that, in the near future, you’ll see also some stuff covered in this blog <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/06/wlEmoticon-smile1.png?w=640" data-recalc-dims="1" />

With this post, I just would like to highlight some things that can be interesting if you want to start to play with the new stuff and, at the same, keep working on Windows Phone development (as expected, nothing has been announced about the next Windows Phone version, the main focus of the conference is Windows).

Here we go!

  * The commercial Visual Studio 2013 Preview versions have built-in Windows Phone development support, but **just for Windows Phone 8**. If you want to keep developing for Windows Phone 7, **you’ll need Visual Studio 2012**. It’s not a big deal since, fortunately, it works side by side with Visual Studio 2013 without problems. 
  * The same **if you use the Express version of Visual Studio 2013,** that doesn’t support Windows Phone projects: you still need the 2012 Express release that comes with the SDK.
  * If you want to try and install Windows 8.1 Preview on your machine, Visual Studio 2012 and Windows Phone SDK work just fine. You’ll just have to install the <a href="http://www.microsoft.com/visualstudio/eng#visual-studio-update" target="_blank">Visual Studio 2012 Update 3</a> (released yesterday), that will fix some compatibility issues with the emulator. Cliff Simpkins just pointed the fix <a href="http://www.monkeyslaps.com/running-the-windows-phone-emulator-on-windows-8-1-preview/" target="_blank">on his blog.</a>

Here are the links for the download:

  * Windows 8.1 Preview: <http://windows.microsoft.com/en-us/windows-8/download-preview>
  * Visual Studio 2013 Preview: <http://www.microsoft.com/visualstudio/eng/2013-downloads>

Just keep in mind that the upgrade process from the Preview release to the RTM (once it will be released) won’t be as smooth as if you update it from a “clean” Windows 8 installation: you’ll be able to keep your data, but you’ll have to reinstall all the apps (both Windows Store and desktop apps).

But there’s more! With the beginning of the BUILD conference, Microsoft has launched a very cool initiative for Windows Phone developers: the entry price to purchase a new Windows Phone developer account **has been cut from 99$ to 19$** until the end of the August. The promotion is valid just for new accounts: it doesn’t apply for renewals. To get it, you’ll simply have to start the registration process in the standard way <a href="http://dev.windowsphone.com/en-us/join" target="_blank">on the Dev Center</a>.