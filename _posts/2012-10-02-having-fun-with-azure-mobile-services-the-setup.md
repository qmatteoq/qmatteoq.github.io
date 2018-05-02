---
id: 218
title: 'Having fun with Azure Mobile Services &#8211; The setup'
date: 2012-10-02T10:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=218
permalink: /having-fun-with-azure-mobile-services-the-setup/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows 8
tags:
  - Azure
  - Windows 8
  - Windows Phone
---
Azure Mobile Services is one of the coolest feature that has been recently added on top of Azure. Basically, it&#8217;s a simple way to generate and host services that can be used in combination with mobile applications (not only Microsoft made, as we&#8217;ll see later) for different purposes: generic data services, authentication or push notification.

It isn&#8217;t something new: these services are build on top of the existing Azure infrastructure (the service is hosted by a Web Role and data is stored on a SQL Database), they&#8217;re just simpler for the developer to create and to interact with.

In the next posts I&#8217;m going to show you how to use Azure Mobile Services to host a simple service that provides some data and how to interact with these data (read, insert, update, etc.) from a Windows 8 and a Windows Phone application. These services are simply REST services, that returns JSON responses and that can be consumed by any application. As you will see, interacting with Windows 8 is really simple: Microsoft has released an add-on for Visual Studio 2012 that adds a library, that can be referenced by a Windows Store app, that makes incredibly easy to do operations with the service.

Windows Phone isn&#8217;t supported yet (even if I&#8217;m sure that, as soon as the Windows Phone 8 SDK will be released, a library for this platform will be provided too): in this case we&#8217;ll switch to &#8220;manual mode&#8221;, that will be useful to understand how to interact with Azure Mobile Services also from an application not written using Microsoft technology (like an iOS or Android app). In this case, we’ll have to send web requests to the service and parse the response: as we’ll see, thanks to some open source libraries, it won’t be so difficult has it sounds.

Before starting to write some code, let&#8217;s see how to configure Azure Mobile Services.

### Activating the feature

Of course, the first thing you&#8217;ll need is an Azure subscription: if you don&#8217;t have, you can subscribe for the free trial [by following these instructions](http://www.windowsazure.com/en-us/pricing/free-trial/?WT.mc_id=A0E0E5C02).

Then, you&#8217;ll need to enable the feature: in fact, since Azure Mobile Services are in a preview stage, they aren&#8217;t enabled by default. To do that, you&#8217;ll need to access to the [Azure Account Management portal](https://account.windowsazure.com/) and open the **Preview features** section: you&#8217;ll see a list of the features that are available. Click on the **Try now** button next to the **Mobile Services** section, confirm the activation and&#8230; you&#8217;re ready! You should receive within a few minutes a confirm mail and the page should display the message **You are active**.

[<img title="mobile services" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; padding-right: 0px; border-top-width: 0px" border="0" alt="" src="https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/mobile-services.png?resize=535%2C123" width="535" height="123"  data-recalc-dims="1" />](https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/mobile-services.png)

### Creating the service

Now that the feature is enabled, we can start using it from the new [Azure Management portal](https://manage.windowsazure.com/): press the **New** button and choose **Create** in the **Mobile Services** section. In the first step of the wizard you&#8217;ll be asked to choose:

  * A name for the service (it will be the first part of the URL, followed by the domain **azure-mobile.net** (for example, myservice.azure-mobile.net) 
      * The database to use (actually, you&#8217;ll be forced to select **Create a new SQL database**, unless you already have other SQL Azure instances). 
          * The region where the service will be hosted: for better performance, choose the closest region to your country. </ul> 
        &nbsp;
        
        [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image_thumb.png?resize=495%2C353" width="495" height="353"  data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image.png)
        
        &nbsp;
        
        The next step is about the database: in the form you&#8217;re going to set some important options.
        
          * The name of the database. 
              * The server where to store the database (use the default option, that is **New SQL Database server**). 
                  * Login and password of the user that will be used to access to the database. 
                      * The region where the database will be hosted: for better performance, choose the same region that you&#8217;ve selected to host the service. </ul> 
                    &nbsp;
                    
                    [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image_thumb1.png?resize=495%2C353" width="495" height="353"  data-recalc-dims="1" />](https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image1.png)
                    
                    And you&#8217;re done! Your service is up and running! If you go to the URL that you&#8217;ve chosen in the first step you&#8217;ll see a welcome page. This is the only &#8220;real&#8221; page you&#8217;ll see: we have created a service, not a website, specifically it&#8217;s a standard REST service. ****As we&#8217;ll see in a moment, we&#8217;ll be able to do operations on the database simply by using standard HTTP requests.
                    
                    ### Let&#8217;s create a table
                    
                    To host our data we need a table: for this example, since I&#8217;m a comic addicted, we&#8217;ll create a simple table to store comics information, like the title, the author and the publishing year. Creating the table is the only part a little bit tricky: the Azure Mobile Service interface, as we&#8217;ll see later, provides built in functions just to create a table, without providing functionalities to change the schema and add new columns.
                    
                    The first thing is to create the table by choosing the service with just created in the portal (in the **Mobile services** section), switching to the **Data** tab and pressing the **Create** button. You’ll be asked to give to the table a name and to set the permissions: by default, we’ll give full access to any application that has been authorized using the secret application key.
                    
                    [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://i0.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image_thumb2.png?resize=495%2C425" width="495" height="425"  data-recalc-dims="1" />](https://i2.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/10/image2.png)
                    
                    Now it’s time to move to the specific Azure tool to manage our SQL instance: in fact, by default, the table will contain just an **Id** field, already configured to be an identity column (its value will be auto generated every time a new row is inserted) and to act as the primary key of our table. In the management panel, click on the **SQL Databases** tab; you&#8217;ll find the database you&#8217;ve just created in the previous wizard. Click on the **Manage** button that is placed below: if it&#8217;s the first time you connect to this database from your connection, the portal will prompt you to add your IP address to the firewall rules: just say **Yes**, otherwise you won&#8217;t be able to manage it.
                    
                    Once you&#8217;ve added it, you&#8217;ll see another confirmation prompt, that this time will ask you if you want to manage your database now: choose **Yes**and login to the account using the database credentials you&#8217;ve created in the first step.
                    
                    Now you have full access to the database management and we can start creating our table: click on the ****database connected to your service (look for the name you&#8217;ve chosen in the second step of the wizard), then click on the **Design** tab and choose **Create new table**. If you&#8217;re familiar with databases, it should be easy to create what we need: let&#8217;s simply add some columns to store our information.
                    
                      * A title, which is a varchar with label **Title** 
                          * An author, which is another varchar with label **Author** </ul> 
                        There’s another way to add new columns: by using **dynamic data.** When this feature is enabled (you can check it in the **Configure** tab of your mobile service, but it’s enabled by default), you’ll be able to add new columns directly from your application, simply by adding new properties to the class that you’re going to use to map the table. We’ll see how to do this in the next post.
                        
                        As we&#8217;ve seen before, if we call the URL of our service (for example, http://myapp.azure-mobile.net) you&#8217;ll see a welcome page: to actually query our tables, we have to do some REST calls. To get the content of a table, we simply need to do a GET using the following URL.
                        
                        **_http://myapp.azure-mobile.net/tables/Comic_**
                        
                        If everything worked fine, the browser should return you a JSON response with the following format:
                        
                        <pre>{"code":401,"error":"Unauthorized"}</pre>
                        
                        This is the expected behavior: Azure Mobile Services require authentication, to avoid that everyone, just with the URL of your service, is able to access your data. By receiving this kind of error we have a confirmation that the table has been successfully created: otherwise, we would have received an error saying that the requested table doesn&#8217;t exist.
                        
                        <pre>{"code":404,"error":"Table 'Comic' does not exist."}</pre>
                        
                        Now that we have setup everything we need, we are ready to write some code: in the next posts we&#8217;ll see how to develop a Windows 8 and a Windows Phone application that is able to connect to our service.