---
id: 167
title: 'Windows Phone 8 unveiled: all the new features for developers'
date: 2012-10-31T10:00:08+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=167
permalink: /windows-phone-8-unveiled-all-the-new-features-for-developers/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows Phone
tags:
  - Windows Phone
---
Finally Windows Phone 8 is here! After the first official Windows Phone 8 presentation, that occurred on 20th June in San Francisco, Microsoft has released during the BUILD conference the SDK, unveiling all the developer and consumer features that were still unknown.

Unlike with the previous versions of the OS, this time Microsoft has released directly the RTM version: in this post we’ll talk the new feature that are available in the final version of Windows Phone 8 and that will be installed on the new devices that will be available in the next weeks. You can download the new tools from the <a href="http://wpdev.ms/wpsdkdl" target="_blank">following link</a> and it’s available in many languages, even if, as for every other development technology, I suggest you to install the English tools.

Before starting, I would like to highlight that Microsoft is going to apply a big promotion to everyone that will register as a developer in the Windows Phone Dev Center in the next 8 days: in 30-45 days you&#8217;ll receive a refund of 91 $ on your credit card, this means that the total price of the subscription will be 8 $ instead of the usual 99 $ fee.

### New tools, new emulator and new requirements

The new SDK is aligned with the latest Microsoft technologies: it’s based on **Visual Studio 2012** (as with the old SDK, the Express version will be installed if you don’t already have a professional one) and it requires **Windows 8** to run. Both products needs to be in RTM release: Windows 8 Release Preview and Visual Studio 2012 RC are not supported.

The Windows 8 requirement has been introduced due to the new emulator, that is now a **Hyper-V** virtual machine: Hyper-V is the Microsoft virtualization technology that, prior to Windows 8, was available only in the server editions of Windows. Now it’s available also in the “consumer” version of the OS, so the emulator can take advantage of it.

This new requirement, anyway, won’t make old computer’s owners very happy: Hyper-V, to run properly, requires your CPU to support **SLAT** (Second Level Address Translation), which is a technology available only in the latest CPUs models (for example, talking about the Intel world, it’s available on processors that belong to the i3 family and later, the old Core 2 Duo is not supported). In the <a href="http://www.howtogeek.com/73318/how-to-check-if-your-cpu-supports-second-level-address-translation-slat/" target="_blank">following article</a> you’ll find a procedure that, using the utility **CoreInfo** that is part of the **SysInternals** suite, will help you to discover if your CPU is supported.

The Windows Phone 8 SDK requires the final Window 8 release, that is available on the market since a few days (even if partners, MSDN and TechNet subscribers have access to the RTM since 15th August): if you’re not one of them, you can install the 90 days evaluation version <a href="http://msdn.microsoft.com/en-us/evalcenter/jj554510.aspx" target="_blank">that is available here</a>. I suggest you to install it in a separate partition: in fact this 90 days trial can’t be converted in a full version, but you’ll be forced to install it from scratch. For this reason, it’s not a good idea to install it as a main operating system.

Another important new feature is that now the emulator is split into three virtual machines, one for every supported resolution: one of the new Windows Phone 8 features (already announced in June) is that two new resolutions are available other than the original one (that is **480&#215;800**, called **WVGA**). These new resolutions are **768&#215;1280 (WXGA)** and **720&#215;1280 (720p)**.

Other than this new feature, the emulator should be familiar to every Windows Phone developer: it’s a virtual machine, that can take benefit of the computer’s hardware to simulate, for example, touch screen, video acceleration and microphone recording. One difference you’ll notice immediately (other than the shiny new start screen) is that, this time, **the OS image is not restricted**, but it contains all the data and applications that are available in a regular device. The Windows Phone 7 emulator, instead, contained a minimal OS image, with just Internet Explorer and a bunch of fake data to test launchers and choosers. This new OS image has been introduced since, as you will learn reading this post, now developers have more options to integrate their apps with the operating system.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/image_thumb1.png?resize=222%2C400" alt="image" width="222" height="400" border="0" data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/image1.png)

There are many news to discover also in Visual Studio 2012: the first you’ll notice when you create a new Windows Phone project is that you’ll be able to develop applications both for Windows Phone 7 and 8 (like, with the previous SDK, developers were able to develop apps for Windows Phone 7.0 and 7.5). This is a very important feature, because Windows Phone 8 is able to run Windows Phone 7 application, but the opposite is not true. So, if you want to keep your application compatible with both worlds, you’ll need to maintain two different Visual Studio projects (with, maybe, a common codebase): thanks to this retro compatibility, you’ll be able to maintain both your apps with just one environment, that is Visual Studio 2012.

Another nice addiction is the **Simulation Dashboard**, which is a new tool (available in the Visual Studio’s **Tools** menu) that can be used to simulate some real phone behaviors, like the strength of the cellular signal, the speed of the data connection (3G, 4G, etc.), the lock screen activation or the incoming of a reminder.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/image_thumb2.png?resize=262%2C353" alt="image" width="262" height="353" border="0" data-recalc-dims="1" />](https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/image2.png)

One new feature that is really appreciated is **the new manifest visual editor**: with the Windows Phone 7 SDK developers had to manually edit the manifest (which is a XML file) to edit the information, change the required images or set the capabilities. Now, instead, you can use a visual editor that looks very similar to the one available when you develop Windows 8 applications: the good news is that it isn’t a specific Windows Phone 8 feature, but you can take advantage of the new editor also when you develop Windows Phone 7 applications. Anyway, even if it’s a nice addiction, the visual editor is still not perfect: some changes (like registering an application in the Pictures hub) still requires to be applied manually.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image_thumb10.png?resize=481%2C321" alt="image" width="481" height="321" border="0" data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image10.png)

&nbsp;

Do you remember the nice tools that have been added with the Windows Phone 7.5 SDK to simulate location, accelerometer and to take screenshot? They are still there, with a nice addiction: now, to grab a screenshot, you don’t need anymore to set the emulator zoom to 100%, but the image is automatically scaled at the correct resolution. Plus, you can find a new section with the details of the network configuration of the emulator.

### A new development platform

After the June announcement, one of the most interesting discussion about Windows Phone 8 has been about the new development platform: during the June event Microsoft has generically spoke about XAML and C# / VB.NET, without directly mentioning Silverlight. Every developer took this information as a sign that Windows Phone 8 would have been aligned with Windows 8 and that it would have be used WinRT as a runtime instead of Silverlight.

The real truth is somewhere in the middle: **Windows Phone 8 is effectively based on a WinRT subset**, but the XAML isn’t native like in Windows 8, but it’s managed like in Silverlight. This means that the XAML we wrote for our Windows Phone 7 application is 100% compatible with Windows Phone 8 and can be reused with no changes. For example, you’ll be able to use also behaviors, that is a feature that, instead, is missing in the Windows 8 XAML.

The other important thing that you’ll notice is that, in order to maintain compatibility with Windows Phone 7, many APIs and classes are duplicated: for example, to access to the application storage you can use the old classes (like **IsolatedStorageFile,** that belongs to the namespace **Microsoft.Phone.Storage**) or the new API **ApplicationData.Current.LocalFolder**, that belongs to the **Windows.Storage** namespace and that can be used using the new async / await pattern and that works exactly like in the Windows 8 world.

The same happens with some controls: for example, in the SDK you can find the old Bing Maps control and the new Nokia Maps control, which can take benefit of all the new enhancements and features.

The other important news, already announced in June, is that now you can develop application using **native code (C++ and DirectX)**, even if some features (like background tasks) are supported only using managed languages like C# and VB.NET. This feature will be very appreciated by games developers, that now will be able to port games from other platform without having to rewrite them from scratch using the XNA framework.

And, speaking about XNA, I have a sad news for XNA developers: **Windows Phone 8 doesn’t support anymore XNA** as a game development platform. You can still use it, but you have to develop a Windows Phone 7 application: this means that you won’t be able to take advantage of the new Windows Phone 8 features in your XNA games. The new official platform to develop games, both for Windows 8 and Windows Phone 8, is C++ with DirectX support.

Another sad news is for web developers: unlike the full WinRT runtime, that supports WinJS, a projection to write native applications using HTML and Javascript, **Windows Phone 8 doesn’t support native development using web languages**. Anyway, you can take advantage of the new Internet Explorer 10 based web control, that will allow platforms like PhoneGap to support much more features than in the past. You’ll even find a template dedicated to HTML 5, but I would like to underline that there is no WinJS support: the template simply creates a Windows Phone application with an embedded WebBrowser control.

### 

### The WinRT runtime for Windows Phone

The new WinRT runtime is definitely one of the most interesting new features in Windows Phone 8 from a developer’s point of view: now, thanks to this common base, you’ll be able to port your applications to Windows 8 and vice versa much more easily than you actually do in Windows Phone 7, since a lot of code should be rewritten due to the API differences between WinRT and Silverlight.

As already anticipated, the Windows Phone 8 APIs are split into two containers: the **Microsoft.Phone** namespace, that contains all the old APIs and some of the new ones that can be used with the old approach, and the **Windows** namespace, that contains all the native WinRT APIs and that has the same structure of the full runtime for Windows 8. The biggest difference with the old APIs is that all the WinRT classes and methods are based on the async / await pattern: in Windows Phone 7 the standard pattern to manage asynchronous method was to provide an event handler that was called when the async operation was completed. With the new WinRT APIs every asynchronous method returns a **Task<T>** object, that allows to write asynchronous code that looks like synchronous and that’s easier to write and to understand.

Pay attention that, to keep the compatibility with the Windows Phone 7 world, not every class supports this new pattern: for example, the **HttpWebRequest** and **WebClient** classes (that are used to perform network operations) don’t provide methods that returns a **Task** object, unlike in the Windows 8 world.� Luckily, thanks to the Task APIs, it’s easy to write one or more extension methods that can avoid this limitation.

Speaking about compatibility, it’s nice to see that old libraries will still work in Windows Phone 8: you’ll able to reuse your favorite toolkits and libraries, at least until they will be ported to Windows Phone 8, to take advantage of all the new features.

The WinRT subset available in Windows Phone 8 looks, from an architectural point of view, like the Silverlight Runtime for Windows Phone: the Windows Phone team took the full Silverlight runtime and created a subset, by removing the not needed stuff (for example, the print APIs) and adding support for specific phone features (like launchers, or GPS and sensors support). WinRT for Windows Phone is pretty much the same: the team took the full WinRT runtime and removed some not needed APIs (like the ones to interact with the charm bar) and added new ones (like the one to manage the Back button).

Here is a more detailed list of the main differences between the full runtime and the Windows Phone subset:

  * **Input:** there is no support to gestures, but just to raw data.
  * **Launchers and choosers:** it’s available a subset of the original APIs, since the phone offers limited interaction than a tablet or a regular computer. For example, the **FileOpenPicker** can be used just to access photos, since there’s no file system access.
  * **Core:** in the Windows Phone ecosystem there’s only one **CoreWindow** and only the **Launch** contract is supported (that is used to start the application), while in Windows 8 you have access to many contracts (like Search or Share).
  * **In App Purchase:** the in app purchase APIs are pretty much the same, except that Windows Phone 8 support **consumables,** that are items that can be purchased more than once during the application lifetime.
  * **Sensors:** inclinometers APIs are not supported, since this kind of sensor is not available on phones.
  * **Storage:** the APIs to access to the local storage are the same, but in Windows Phone 8 temporary storage and roaming storage (that contains data that is synced between different devices using the Microsoft account) are not supported. They are available, but you’ll get a **NotSupportedException**if you try to use one of them.
  * **Device communication:** unlike in Windows 8, it’s not possible to set up a direct connection between different devices using Wi-Fi connection. This feature, anyway, is available using the new NFC and bluetooth APIs.

### 

### What about controls?

There aren’t many news about controls, but the new features are really a nice addiction: the first news is that **Panorama** and **Pivot** controls have been removed from the APIs and **now are stored directly in the phone memory**. This should solve all the performance problems that you can actually experience using these controls in a Windows Phone 7 application.

The other news is that the **LongListSelector** control, that is actually part of the Silverlight Toolkit, has been promoted as an official control: its purpose is to create lists of data that are easier to navigate, thanks to jump lists that allow the users to quickly jump from a group of item to another.

### 

### Tile, notifications and lock screen

In the June event Microsoft has revealed just one of the Windows Phone 8 consumer features: the new start screen, which now fit all the screen size and supports three tile sizes: **159&#215;159 (small)**, **336&#215;336 (medium)** and **691&#215;336 (large)**.

As developers we can use the different sizes to display different information: for example, the native Mail application displays just a counter when the tile is set to small or medium, while it displays a preview of the last unread mail when it’s set to large.

Another new feature is templating support, similar to the one that is available in Windows 8 (even if with a smaller range of choices). Basically, as a developer, you can choose to use one of the three available tile templates and interact with it at runtime: once you’ve set it, you can’t change it, unless you release a new version of the application.

Let’s take a look to the available templates:

**Flip**� is the standard template, that is available also in Windows Phone 7. Basically, you can set some information both on the front and on the rear of the tile which, periodically, flips to show both sides.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/image_thumb4.png?resize=506%2C137" alt="image" width="506" height="137" border="0" data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/image4.png)

**Cycle�** is a new template that allows up to 9 images to be cycled in the tile. The images can be fixed (and set in the manifest file) or can be dynamically changed at runtime. You can see an example of this template also in Windows Phone 7 in the Pictures hub tile, where random images from your library are cycled.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/image_thumb5.png?resize=506%2C164" alt="image" width="506" height="164" border="0" data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/image5.png)

### 

**Iconic�** is a template that can be used to create tiles with the same look and feel of some native applications, like Mail or Messages. You can set an icon, some text, a background color and an optional counter.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/image_thumb6.png?resize=506%2C148" alt="image" width="506" height="148" border="0" data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/image6.png)

&nbsp;

As in Windows Phone 7, all these tiles templates can be updated with push notifications, both local or remote.

One feature that, personally, I love in Windows Phone 8 is that now **notifications can also be used to update the lock screen**, exactly like some native applications do. There are two ways to update the lock screen:

  * by showing an icon with a counter (like Mail does to show the number of unread mails)
  * by showing a text (like Calendar does to show the next appointment)

&nbsp;

As a developer you’ll be able to write applications that, using notifications, can update these information on the lock screen: obviously, the user has total control over it and, from the Settings hub, can choose which applications are able to show a counter (up to 5) or a text (just one) in the lock screen.

The best part is that the effort to implement this feature is minimum: you’ll just have to register the application by adding a new declaration in the manifest file and you’re done. In fact, the lock screen counter is based on the same notifications that are used to update a tile: if you send a tile notification that changes the value of the **Count** property, this value is displayed both in the tile and in the lock screen. The only other requirement is that you’ll have to provide a **24&#215;24** icon, that will be displayed in the lock screen near the counter.

And it’s not over yet: there’s another new feature connected to the lock screen, that is the capability to **register an application as a wallpaper provider**. Once you’ve done that (the user will be asked to confirm) you’ll be able to change the lock screen wallpaper, even periodically by using a background agent. Windows Phone 8 includes an already built in wallpaper provider: Bing. When you activate it, the lock screen is periodically updated with the image of the day that is posted periodically on Microsoft’s search engine’s home page.

From a developer point of view, before starting to use the lock screen APIs you’ll have to register your application as a provider by adding a declaration in the manifest file, like you did for the lock screen notifications.

All these new features have been grouped with the name of� **Live Apps**, that refers to applications that, other than using live tiles, are able to interact with the lock screen.

### Maps

The collaboration between Nokia and Microsoft is going to take a big step forward with Windows Phone 8, since **Bing Maps has been completely replaced by Nokia Maps**. This way, all the Windows Phone users will take benefit of all the Nokia features, like offline maps.

For developers there’s also a new map control, that replaces the Bing Maps one and that provides better performances and features. The Bing Maps control is still there, but it’s deprecated: it’s been maintained just for compatibility with Windows Phone 7.

The downside of this control is that, especially for advanced scenarios (like layers, pushpins and so on), has many differences with the Bing Maps one, so if you want to migrate your application to use the new control you’ll probably have to rewrite a lot of code.

### Speech API

This is absolutely one of the most interesting new features in Windows Phone 8: the platform already has a good built in support for voice commands, for example to execute actions (like calling a contact) and to write text without using the keyboard (like the voice-to-sms feature).

Now, as a developers, you’ll be able to introduce speech functionalities in your application:

  * **Voice commands support**, so that you can execute actions by using the voice (for example, performing an operation or navigating to a specific page).
  * **Text-to-speech,** both to convert voice in text and vice versa.

&nbsp;

The voice commands features is based on special XML files, called **VCD files**, that stores for every information command all the required information, like the keyword to activate it and the navigation command to issue. You can intercept a voice command in the same way you intercept deep links when you use multiple tiles: in the **OnNavigatedTo** event (that is raised when the user lands to a page) you’ll have to check if the **NavigationContext** contains one or more parameters query string parameters.

### 

### Wallet

Wallet is another Windows Phone 8 feature that was announced in June: it’s a new native application that can be used to store payment information, like credit cards, debit cards or membership cards. The data is encrypted, so it can be stored safely: in some countries (like in United States) you’ll be able to link a secure SIM with the Wallet application, in order to make payments directly with your phone using the NFC chip.

As developers we’ll be able to interact with the Wallet application by adding new cards, deals or items to the transaction history.

### In-app Purchase

Also this feature was very awaited from developers: the capability to **make purchases within an application**, for example to buy a subscription or unlock some features.

Microsoft has always allowed in-app purchase, but without providing its own payment infrastructure: developers were forced to implement their own backend to manage payments and transactions. Now in-app purchase is fully integrated with the new Windows Phone Dev Center (the old AppHub), so that you’ll be able to create new items to purchase, choose the price and differentiate it according to the market, exactly like you do with a regular application.

Within the application, thanks to the new APIs, you’ll be able to get the list of the available items and to allow the user to buy one or more of them: all the transactions will be directly managed by Microsoft, using the same architecture provided to purchase apps and music in the Marketplace.

One of the features that is different from Windows 8 is the concept of **consumables**: in Windows Phone 8 you can create some in app-purchase items that can be purchased multiple times (while the standard items can be purchased just once and the purchase status is replicated on every device on which you have installed the app).

### Proximity

Under the keyword **proximity** Microsoft has included many APIs that allow Windows Phone 8 devices to interact with other near devices using both new and old technology. The new one is **NFC (Near Field Communication)**, a chipset that allows two devices to communicate simply by putting them very closely. This is a new Windows Phone 8 feature, since old devices don’t have this chip. One of the most interesting NFC features is that is able to interact both with passive and active devices. Passive devices are chips that can be inserted in a sticker or in a magazine, for example, and that can be used to send small amount of data, like the URL of a website. Active devices are other smartphones or tablets: in this case, one of the most powerful NFC features is that can be used to pair a device using bluetooth simply by putting them at a close distance, without having to do the manual pairing using a PIN. Since Bluetooth is a more stable and faster technology, it can be used to exchange bigger data (like an image) or to establish the communication required to play a multiplayer game.

The other available technology is **Bluetooth**, that it’s always been one of the required chipset since the first Windows Phone release: anyway, in Windows Phone 7 Bluetooth is very limited since, as developers, we aren’t able to use it. Finally Windows Phone 8 has introduced many new APIs to the interact with the Bluetooth chipset and to exchange data with other devices.

Here are some scenarios covered by the Proximity APIs:

  * Communication between the same app installed on different devices, for example to join a multiplayer game.
  * Communication between different devices to exchange data using standard protocols, for example to send images, contacts and so on to another smartphone or a tablet.
  * Communication between a smartphone and a different hardware device, like a game controller or a training device.

### 

### Data sharing

This is one of the new features I like most, since it expands a lot the capabilities of our devices. Now third party apps are able to interact with other apps by sharing data. There are two ways to do that:

  * **By registering an extension:** an application can register one or more extensions (like .log, for example). If another applications tries to open a file with such an extension (using the method **Windows.System.Launcher.LaunchFileAsync**), our application is able to intercept it so that, for example, it can display it in a specific page. Pay attention that there are some native extensions (like Office files) that can’t be registered by a third party application.
  * **By registering a protocol:** an application can also register a protocol (for example, skype:/). If another application launches a URI request (using the method **Windows.System.Launcher.LaunchUriAsync**) our application is able to intercept it and collect the parameters attached to the URI.

&nbsp;

The two features are very similar: they are registered in the same way (in the manifest file) and the data is intercepted by the same class. The only difference is that, in the first case, we’re talking about file sharing: the file is copied from the storage of the launching application to the storage of our application, so that we can work with it. In the second case we work just with plain strings: we can collect the query string parameters of the URL and interact with them.

One scenario that is actually not covered by data sharing is sending files back to the original application: you can&#8217;t do something like launching a file, opening it with another application, editing it and then returning back to the application that launched it.

Data sharing is strictly connected to another new feature, that is the **capability to read data from the SD card:** if the micro SD card (that is supported by some of the new generation devices) contains files which extension has been registered by the application, we can look for them and open them.

### Geo location and background applications

Applications that use geo location services can now run in background, so that they can keep track of the user’s location if when they are not in foreground. In this case, the application uses a new lifecycle model: since the app is not suspended anymore, the current instance is always restored when you open it, regardless if you’re opening it using the Back button or the task manage or by tapping on its tile. Standard apps, instead, will continue to behave like in Windows Phone 7: the app is only restored by using the Back button or by using the task manager, launching it from the tile or from the app list will always open a new instance.

As developers, we will have to manage this new application lifecycle and some of the new APIs will help us to accomplish this task.

### New camera features

There are many new features that photo applications can use to enhance the user experience: the two most interesting ones are **lens apps** and **custom cloud services** support.

Let’s start from lens apps: imagine a real professional camera, where you are able to change the lens to apply different effects to the photos. Lens apps are the same, just applied to the phone’s camera: lens apps are application that can apply various effects to the photos taken by the camera and that can be activated directly from the native camera app. This way, apps can provide a better experience to the user: prior to this feature, users were forced to use third party apps instead of the native one to apply custom effects, preventing them, for example, to use the dedicated button to start the camera. Now, instead, a user can simply launch the camera app in the standard way and then apply one of the available lens apps.

The other cool feature I’ve mentioned is **custom cloud services support:** in Windows Phone 7 users are able to automatically upload a photo, right after it’s taken, directly to one of the the built in services, like SkyDrive or Facebook. In Windows Phone 8 you can develop your own custom services, on which photos will be immediately uploaded.

### Data Sense

Data Sense is a new application that is available in Windows Phone 8, that can be used to keep track of the data traffic made by the phone and, more specifically, by every single application that is installed. Plus, users are able to set the monthly traffic limit allowed by their plans: if the limit is reached, the phone will automatically turn off the data connection, in order to avoid surprises on the phone’s bill.

The Data Sense features can be used also from third party applications using an API, that can be used to check if the user is approaching the monthly limit. This way, we can change the behavior of our application according to this limit: for example, a RSS reader can decide not to download images if we’re approaching it.

The only downside of this feature is that mobile operators will be able to decided if Data Sense should be shipped or not with their phones: if not, developers will be able to use the Data Sense API just to identify the user’s connection (or, for example, if he’s using roaming or not), but they won’t have access to traffic limit information, since the user won’t have a way to set it.

### A lot more!

There are many new features for developer that, for sure, we’ll cover in details in the next months. Here is a short preview:

  * **New launchers and choosers**: Windows Phone 8 features some new launchers (to interact with the new Nokia Map application) and choosers (you’ll surely appreciate the new chooser to save an appointment to the calendar).
  * **VoIP**: developers will be able to develop **VoIP** applications using a subset of the **Windows Audio Sessions API** **(WASAPI)**. The most interesting feature is that VoIP apps will be able to deeply integrate with the operating system, so that you can launch a VoIP call, for example, directly from the detail page of a contact.
  * **New media library APIs**, that enhance the capabilities of what we can do with the files in the media library. There are still some limitations, but we have more flexibility than in the past.
  * **Support for new languages:** application can now support also bidirectional languages.

### And what about consumers?

We are developers, it’s true, but probably we are Windows Phone users too. Which are the new features that have been introduced for consumers in the new platform, other than the shiny new start screen? Let’s see a brief list:

  * **The latest versions of the Microsoft apps:** Windows Phone 8 comes with the most recent technologies by Microsoft, like Office 2013, Internet Explorer 10 and XBox Music.
  * **Kid’s corner:** this is a very cool feature if you have children or kids that are used to play with your phone. Basically, this feature allows you to create a sandbox, that contains just the apps and the music that you have chosen as suitable for kids. This way, you can give the phone to your children without having to worry that they can use apps that aren’t suitable for them or that they can accidentally damage your data.
  * **New accent color:** now you can choose between up to 20 colors to customize your phone.
  * **Offline maps:** Bing Maps has been replaced by Nokia maps, with offline map support. This way you’ll be able to use maps even without a connection: this feature is very useful especially when you’re abroad and you can’t use your data connection due to high roaming charges.
  * **MicroSD support:**now the operating system supports external Micro SD cards, that can be used to store music, videos and pictures (no apps, they can be installed only in the main memory). Even if this feature is supported by the OS, it’s up to the manufacturer if to implement it or not (for example, the Nokia Lumia 820, since it has a removable back cover, supports Micro SD, while the bigger brother Lumia doesn’t).
  * **Backup and restore:**now you can backup your settings, the list of apps you’ve installed and your text messages, that can be restored if you buy a new Windows Phone 8 device or, for any reason, you have to reset your current one.
  * **Native NFC support:** other than being accessible from third party apps using the Proximiy APIs, the operating system itself supports NFC to share data between two devices (for example, to share photos).
  * **OTA (Over The Air) updates:** if Microsoft or the manufacturer release an update for the phone, now you don’t have anymore to connect it to your computer, you can download and install directly from your phone.
  * **Rooms**, that is a way to create special group of contacts, so that you can share calendars, photos or start a chat with.

### The beginning of a new adventure

This is was a really long post: you’ll hardly see another post so long on this blog, but I thought that it didn’t make sense to split it in multiple parts. You are all eager to know the new stuff in Windows Phone 8, right?  <img class="wlEmoticon wlEmoticon-smile" style="border-style: none;" src="https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/wlEmoticon-smile3.png?w=640" alt="Smile" data-recalc-dims="1" />And now the fun begins: in the next months I’ll surely cover in details all the new features and how to implement them in your application.

The only tip I can give you, at the moment, is to start installing the SDK, playing with the new emulator and, if you already are a Windows Phone developer, test your existing applications with it. The compatibility’s level is very high and, very likely, your app will run just fine, even better due to the performance improvements and to the higher specs of the new devices. But there can be some scenarios where the Windows Phone codebases has changed a bit, so your application can have some problems. Make sure that your app runs fine both in Windows Phone 7 and Windows Phone 8 and, if not, update it and publish it!