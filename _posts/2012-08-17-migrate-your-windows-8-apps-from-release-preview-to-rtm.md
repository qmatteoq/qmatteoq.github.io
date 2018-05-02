---
id: 151
title: Migrate your Windows 8 apps from Release Preview to RTM
date: 2012-08-17T12:55:10+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=151
permalink: /migrate-your-windows-8-apps-from-release-preview-to-rtm/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows 8
tags:
  - Windows 8
---
Here we go again: a new Windows 8 version has been released (luckily, it’s the last one <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/08/wlEmoticon-smile.png?w=640" data-recalc-dims="1" />) and we have to test our application against the RTM, to see if everything is working fine.

Luckily, this time the migration process is much easier than with the Release Preview: I’ve been able to run my applications on the RTM just by compiling them with the final Visual Studio 2012 release; when Release Preview was released, instead, I had to fix a lot of things, since there were a lot of breaking changes from the Consumer Preview.

In this post I would like just two highlight two important things to keep in mind.

### Update the manifest file

Thanks to <a href="http://www.bochicchio.com/" target="_blank">Daniele Bochicchio</a> I’ve found that the Windows 8 manifest file contains a reference to the OS version which the application is compiled for. If you’re working on a project created with the Release Preview, the version number will be **6.2.0,** while the RTM version number is **6.2.1.** Unlucky, for old projects, Visual Studio doesn’t update it for you, so you need to proceed manually.

  * In Visual Studio 2012 right click on the **Package.appxmanifest** file and choose **View code**. 
      * You’ll find a section in the XML file called **Prerequisites**.</ul> 
    <pre class="brush: xml;">&lt;Prerequisites&gt;
  &lt;OSMinVersion&gt;6.2.0&lt;/OSMinVersion&gt;
  &lt;OSMaxVersionTested&gt;6.2.0&lt;/OSMaxVersionTested&gt;
&lt;/Prerequisites&gt;
</pre>
    
      * Change the value **6.2.0** that is stored in both nodes to **6.2.1,** so that it looks like this:
    <pre class="brush: xml;">&lt;Prerequisites&gt;
  &lt;OSMinVersion&gt;6.2.1&lt;/OSMinVersion&gt;
  &lt;OSMaxVersionTested&gt;6.2.1&lt;/OSMaxVersionTested&gt;
&lt;/Prerequisites&gt;
</pre>
    
    And you’re done! This way you’ll be able to take benefit of all the WinRT changes applied to the RTM: otherwise, the app will run in a compatibility mode called **Release Preview Compatibility Mode.** You’ll app will pass the certification process anyway, but it’s better if we set from the beginning the right OS version.
    
    If you want to go deeper about the RTM changes, take a look at <a href="http://www.microsoft.com/en-us/download/details.aspx?id=30706&WT.mc_id=rss_windows_allproducts%23" target="_blank">this document</a> published by Microsoft that lists all the changes between the RP and the RTM from a developer point of view.
    
    ### Unable to activate Windows Store application: the app didn’t start
    
    The first time you open your project with Visual Studio 2012 RTM and you try to debug it, you may occur in a strange error:
    
    **_Unable to activate Windows Store application. The activation request failed with error ‘The app didn’t start’_**
    
    In this case the solution is simple: close Visual Studio, delete the **bin** and **obj** folders inside your project’s folder, open it again with Visual Studio and launch it. This time it will start successfully!