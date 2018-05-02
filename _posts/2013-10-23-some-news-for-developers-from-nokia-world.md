---
id: 4862
title: Some news for developers from Nokia World
date: 2013-10-23T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=4862
permalink: /some-news-for-developers-from-nokia-world/
categories:
  - Windows Phone
tags:
  - Nokia
  - Windows Phone
---
Yesterday Nokia has showed off to the world, during Nokia World, many great news about Windows Phone and the Lumia series. In fact, Nokia has announced 2 new phones which falls into the “phablet” category (they both share a 6” screen), a tablet (based on Windows 8.1 RT) and many important apps that are coming in the next weeks, like Instagram, Vine and Flipboard.

They all are really good news for developers, since I’m sure that they will push even further the Windows Phone ecosystem and more phones around the world means more opportunities for devs <img class="wlEmoticon wlEmoticon-smile" style="border-style: none;" alt="Smile" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/10/wlEmoticon-smile.png?w=640" data-recalc-dims="1" />

To get all the information about the new products and apps, I suggest you to visit the official Nokia blog called <a href="http://conversations.nokia.com/" target="_blank">Nokia Conversation</a>, which covers in details all the news. In this post, I would like to focus on what’s interesting for developers regarding the new announcements.

### Lumia 1320 and Lumia 1520

The new members of the Lumia family are part of the so called “phablet” category: they are phones, but with a very big screen (6 inches). The Lumia 1520 is a high end phone, which features the new resolution introduced with GDR3, which is 1080p (1080&#215;1920), together with a quad core processor (another new feature for a Windows Phone device) and a 20 MPixel camera. The Lumia 1320, on the other side, is a mid end phone, with lower tech specs but with a much cheaper price. It features a 720p resolution (720&#215;1280), which is not new for Windows Phone (it’s one of the new resolutions introduced in Windows Phone 8), but it’s new for Nokia, since previously all the other phones used only the WVGA (480&#215;800) and WXGA (768&#215;1280) resolutions.

And here comes the first news for developers: 1080p and 720p resolutions are very different from the other ones, since they offer a 16:9 aspect ratio, instead of the usual 15:9. This means basically two things:

  * If you’ve used a rigid layout in your application (using, for example, fixed margins and sizes), there’s a good chance that your app will not look very good on one of these devices. The solution is to use a fluid layout, which is able to adapt to the screen size. For example, when you create a Grid, try not to use fixed sizes for columns and rows. There&#8217;s a [good article in the MSDN documentation](http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj206974(v=vs.105).aspx) about this topic.
  * Windows Phone 7 apps don’t support the new resolutions: if for WXGA phones is not a problem (aspect ratio is the same, so layout is simply adapted but not distorted), it’s not the same for 720p and 1080p. Since aspect ratio is different, there’s the chance that the layout will look bad: for this reason, Windows Phone 7 apps that are executed on a 720p or 1080p device will look “cut” at the top, since a black bar will be added to keep the 15:6 aspect ratio. The only solution to fix it is to upgrade your project to Windows Phone 8: only this way you’ll be able to fully support the new devices.

If, in the past, developers didn’t care too much about 720p, since only a few models with low market share supported it, now it’s definitely time to change and fully embrace it: otherwise, people that will buy the Lumia 1320 or the Lumia 1520 won’t be able to experience your apps with full quality.

Nokia has scheduled one webinar in two different dates and times (<a href="http://forumnokia.adobeconnect.com/lal16-1080p-ssn1/event/event_info.html" target="_blank">6 November</a> and <a href="http://forumnokia.adobeconnect.com/lal16-1080p-ssn2/event/event_info.html" target="_blank">7 November</a>) on how to adapt your application to fully support the new resolutions. Don’t miss it!

Plus, if you remember my previous post, you should know that Microsoft is going to release soon a SDK update which features a new GDR3 emulator with the 1080p resolution. However, since aspect ratio is the same, you can start right now to test your apps for the new Lumia devices with the 720p one. If your app looks good a on a 720p phone, it will look good also on a 1080p one (unless you’ve used low quality images, in this cases they can look bad on a high resolution screen).

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image_thumb[5]" alt="image_thumb[5]" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/10/image_thumb5_thumb.png?resize=378%2C258" width="378" height="258" border="0" data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/10/image_thumb5.png)

### Lumia 2520

Nokia has announced also his first tablet, a Windows 8.1 RT device with a look and feel very similar to the Lumia smartphones. It’s a quad core ARM device with a 1080p resolution which, as every other Windows RT device, is optimized for the new Windows Store apps introduced in Windows 8: desktop apps won’t run, except for the ones provided by Microsoft (like the Office suite).

From a developer point of view, it’s a new opportunity: if you’ve already have a Windows Phone app that could make sense on a larger screen, you can start thinking about a porting for Windows 8. There’s some work to do, but there are many shared classes between the Windows Runtime for Windows 8 and the Windows Runtime for Windows Phone, like storage APIs, sensors APIs, geolocalization APIs, etc. I suggest you to start looking <a href="http://channel9.msdn.com/Series/Building-Apps-for-Both-Windows-8-and-Windows-Phone-8-Jump-Start" target="_blank">at this series of webinars published on Channel 9</a> made by Ben Riga: they will introduce you to concepts like Model-View-ViewModel (which <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-complete-series/" target="_blank">we covered deeply in this blog</a> too) and Portable Class Libraries, which are two of the tools that will help you to reuse your Windows Phone code for a Windows 8 app.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image_thumb[11]" alt="image_thumb[11]" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/10/image_thumb11_thumb.png?resize=378%2C258" width="378" height="258" border="0" data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/10/image_thumb11.png)

### Nokia Developer Offers

The most interesting news for developers is, probably, the launch of a new program called Nokia Developer Offers, which gave to developers, for free, a set of very useful tool and resources:

  * A token to subscribe to the Store or to renew an existing subscription
  * A license for BugSense Performance Monitoring Solution, which is an analytics platform
  * A 1 year license for Telerik controls for Windows Phone
  * A 1 year license for Infragistics controls for Windows Phone

However, there are a couple of requirement to be able to join the program and get all these goodies for free:

  * You have to be subscribed to DVLUP, so if DVLUP is not available in your country yet, I’m sorry but you’ll have to wait
  * If you’re a new developer, you should have published at least 2 application on any mobile platform: not just Windows Phone or Asha, but also Android, iOS, etc.
  * If you already had a Nokia Developer Premium account in the past, you should have released at least one app after you’ve subscribed.

The Nokia team will take a few days to verify that you’re eligible. Once you’re in, it’s important to know that, in the future, you could get access to new benefits (for example, discounts on the XP points required to get rewards on DVLUP): to continue to be part of the program, you’ll have to publish at least a new app or an update to an existing app within 6 months from your subscription.

If everything I told you sounds good<a href="http://developer.nokia.com/Developer_Programs/nokia_developer_offers.xhtml" target="_blank">, go to the official website and apply!</a>