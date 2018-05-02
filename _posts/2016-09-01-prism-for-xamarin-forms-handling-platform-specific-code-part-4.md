---
id: 6616
title: 'Prism for Xamarin Forms &ndash; Handling platform specific code (Part 4)'
date: 2016-09-01T16:00:00+00:00
author: qmatteoq
layout: post
guid: http://blog.qmatteoq.com/?p=6616
permalink: /prism-for-xamarin-forms-handling-platform-specific-code-part-4/
categories:
  - Xamarin
tags:
  - MVVM
  - Prism
  - Xamarin
  - Xamarin Forms
---
Xamarin Forms is cool because it allows to share not just the business logic but also the user interface, unlike the traditional Xamarin approach, which requires to create multiple projects with its own user experience and features. However, at some point, even with Xamarin Forms you have to deal with platform specific code: unless your application leverages only plain and simple features that can be embedded into a PCL (like in the sample project we’ve created in the <a href="http://blog.qmatteoq.com/?p=6585" target="_blank">previous posts</a>, where we simply downloaded some data from a REST API and we displayed it to the user), sooner or later you will have to use a feature that requires to write code which is different based on the target platform. There are many samples of this scenario: one of the most common is geo localization. Android, iOS and Windows offer a way to geo localize the user, but every platform uses its own approach and APIs, so you can’t include them in the Portable Class Library, since these APIs aren’t shared among the platform.

Luckily, to make the developer’s life easier, Xamarin has created many plugins for the most common scenarios, like geo localization, file system access, network, etc. which are available on NuGet or GitHub ([https://github.com/xamarin/XamarinComponents](https://github.com/xamarin/XamarinComponents "https://github.com/xamarin/XamarinComponents"))

However, not all the scenarios are covered, so it may easily happen that you are required to implement your own plugin. Xamarin Forms includes an infrastructure to handle this requirement in an easy way, based on a class called **DependencyService**. From a high level, here is how it works:

  1. In your PCL you create an interface, which describes the methods you want to use in your shared code (for example, you could create an interface called **IGeolocatorService** which offers a **GetPosition()** method to retrieve the current location of the user). However, since it’s an interface, it describes just the operations you want to perform, without actually implementing them. 
      * In every platform specific project, you create a class that implement this interface, using the actual APIs provided by the platform. This means that, for example, your Android project will contain an **AndroidGeolocatorService**, which will contain the actual implementation of the **GetPosition()** method using the Android APIs. These classes have to be decorated with a special attribute provided by Xamarin Forms called **Dependency** with, as parameter, a reference to the type of the class itself: </ol> 

```csharp
[assembly: Dependency(typeof(AndroidGeolocatorService))]
namespace InfoSeries.Droid.Services
{
    public class AndroidGeolocatorService: IGeolocatorService
    {
        public async Coordinates GetPosition()
        {
            //implementation using Android APIs
        }
    }
}
```
    
Thanks to this attribute, you are now able to use the **DependencyService** class and the **Get<T>()** method (where **T** is the interface type) in your PCL to get the proper implementation of the service, based on the platform where the app is running. Consequently, let’s say that in your PCL you have, at some point, the following code:
    
```csharp
IGeolocatorService service = DependencyService.Get&lt;IGeolocatorService&gt;();
var result = service.GetPosition();
```
    
When the Xamarin Forms app is launched on an Android device, the **Get<IGeolocatorService>()** method will return an instance of the **AndroidGeolocationService** class; vice versa, if the app is launched on a Windows 10 device, the method will return an instance of the **UwpGeolocationService** class, which would have been created into the specific UWP project of the solution.
    
So far, so good. However, to reach this goal in our MVVM application, we would need to use the **DependencyService** class in a ViewModel, which isn’t a great approach: using a static property in a ViewModel makes things harder to test. Additionally, this approach would require us to use two different approaches based on the service type: 
    
1. If it’s a non platform specific service (like the **TsApiService** class we created in the previous posts to interact with the TrackSeries APIs), we register it into the Prism container and we let it automatically be injected into the ViewModel’s constructor.  
2. If it’s a platform specific service (like the previous **AndroidGeolocatorService** class), we have to access to it 
through the **DependencyService** class and not through the standard Prims container.</ol> 

With other MVVM frameworks, a typical workaround is to use the **DependencyService** in the framework’s bootstrapper to get the platform specific instance and then register it into the dependency container we’re using. This way, we can continue to leverage the usual approach to simply add a parameter in the ViewModel’s constructor and to have its implementation automatically injected. This is made possible by the fact that basically all the dependency containers allow to register not only a generic type (which means that, in this case, the container will create a new instance of the class when it’s requested) but also to assign a specific implementation to an interface.
        
Here is a sample code of how it would work:
        
```csharp
protected override void RegisterTypes()
{
    Container.RegisterTypeForNavigation&lt;NavigationPage&gt;();
    Container.RegisterTypeForNavigation&lt;MainPage&gt;();
    Container.RegisterTypeForNavigation&lt;DetailPage&gt;();
    Container.RegisterType&lt;ITsApiService, TsApiService&gt;();

    var geoService = DependencyService.Get&lt;IGeolocationService&gt;();
    Container.RegisterInstance&lt;IGeolocationService&gt;(geoService);

}
```
        
However, Prism is smarter than that and it’s automatically able to register into its container every class he finds in each platform specific project decorated with the **Dependency** attribute. You can find a real example in the InfoSeries app on my GitHub repository: [https://github.com/qmatteoq/XamarinForms-Prism/tree/master/InfoSeries](https://github.com/qmatteoq/XamarinForms-Prism/tree/master/InfoSeries "https://github.com/qmatteoq/XamarinForms-Prism/tree/master/InfoSeries")
        
I’ve decided to implement a feature that allows a user to share, from within the detail page of a TV Show, the poster image of the show. This is something that needs to be implemented in a different way for each platform (since each of them has its own unique APIs to share content) and we can’t leverage an existing plugin (since the one available in the Xamarin repository supports only text sharing). As such, I’ve created in the Xamarin Forms Portable Class Library the following interface:
        
```csharp
public interface IShareService
{
    Task SharePoster(string title, string image);
}
```
        
The interface simply describes an asynchronous method, called **ShareShirt()**, which accepts as parameter the title of the show and the path of the image to share. The next step is to implement this interface in every platform specific project. I won’t go into the details, because it would be out of scope for the post but, for example, here is how it looks the implementation in the Android project:
        
```csharp
using System;
using System.IO;
using System.Net.Http;
using System.Threading.Tasks;
using Android.App;
using Android.Content;
using InfoSeries.Droid.Services;
using InfoSeries.Services;
using Xamarin.Forms;
using Environment = Android.OS.Environment;

[assembly: Dependency(typeof(AndroidShareService))]
namespace InfoSeries.Droid.Services
{
    public class AndroidShareService: IShareService
    {
        public async Task SharePoster(string title, string image)
        {
            var intent = new Intent(Intent.ActionSend);
            intent.SetType("image/png");
            Guid guid = Guid.NewGuid();
            var path = Environment.GetExternalStoragePublicDirectory(Environment.DataDirectory
                                                                + Java.IO.File.Separator + guid + ".png");

            HttpClient client = new HttpClient();
            var httpResponse = await client.GetAsync(image);
            byte[] imageBuffer = await httpResponse.Content.ReadAsByteArrayAsync();

            if (File.Exists(path.AbsolutePath))
            {
                File.Delete(path.AbsolutePath);
            }

            using (var os = new System.IO.FileStream(path.AbsolutePath, System.IO.FileMode.Create))
            {
                await os.WriteAsync(imageBuffer, 0, imageBuffer.Length);
            }

            intent.PutExtra(Intent.ExtraStream, Android.Net.Uri.FromFile(path));

            var intentChooser = Intent.CreateChooser(intent, "Share via");

            Activity activity = Forms.Context as Activity;
            activity.StartActivityForResult(intentChooser, 100);
        }
    }
}
```
        
As you can see, the class implements the **ShareShirt()** method by downloading the poster image in the local storage of the app and then, by using specific Android APIs (like **Activity** or **Intent**), it starts the sharing operation. As you can see, the class is decorated with the **Dependency** attribute with, as parameter, the type of the class itself.
        
Thanks to Prism, that’s all we need to do. Now simply add, in the **DetailPageViewModel,** a parameter of type **IShareService** in the class constructor, so that you can use it the command that will be invoked when the user will press the button to share the image. Here is how our updated **DetailPageViewModel** looks like:
        
```public class DetailPageViewModel : BindableBase, INavigationAware
{
    private readonly IShareService _shareService;
    private SerieFollowersVM _selectedShow;

    public SerieFollowersVM SelectedShow
    {
        get { return _selectedShow; }
        set { SetProperty(ref _selectedShow, value); }
    }

    public DetailPageViewModel(IShareService shareService)
    {
        _shareService = shareService;
    }

    public void OnNavigatedFrom(NavigationParameters parameters)
    {

    }

    public void OnNavigatedTo(NavigationParameters parameters)
    {
        SelectedShow = parameters["show"] as SerieFollowersVM;
    }

    private DelegateCommand _shareItemCommand;

    public DelegateCommand ShareItemCommand
    {
        get
        {
            if (_shareItemCommand == null)
            {
                _shareItemCommand = new DelegateCommand(async () =&gt;
                {
                    string image = SelectedShow.Images.Poster;
                    await _shareService.SharePoster(SelectedShow.Name, image);
                });
            }

            return _shareItemCommand;
        }
    }
}
```
        
We have added an **IShareService** parameter in the constructor and we have defined a **DelegateCommand** called **ShareItemCommand**: when it’s invoked, it will simply call the **ShareShirt()** method exposed by the **IShareService** interface, passing as parameter the name of the show and the URL of the poster image (which can be both retrieved from the **SelectedShow** property).
        
In the end, in the **DetailPage.xaml** we have added a **ToolbarItem** control, which adds a new button in the UI’s toolbar (which is rendered in a different way based on the platform, for example on UWP it’s rendered as a button in the **CommandBar**):
        
```xml
<ContentPage.ToolbarItems>
  <ToolbarItem Text="Share" Command="{Binding Path=ShareItemCommand}" Order="Secondary"/>
</ContentPage.ToolbarItems>
```
        
<img title="Screenshot_2016-08-22-14-46-47" src="https://i2.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/08/Screenshot_2016-08-22-14-46-47_thumb.png?resize=290%2C516"/>

<img title="Screenshot_2016-08-22-14-46-38" src="https://i2.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/08/Screenshot_2016-08-22-14-46-38_thumb.png?resize=290%2C516" />
        
Nothing special here: the control has a text and it’s connected through binding to the **ShareItemCommand** we have previously defined in the ViewModel. Now launch the application on an Android device or emulator, choose one TV Show and use the new share option we’ve just added: automagically, Android will show us the share chooser, which will allow the user to choose which target application will receive the image. As you can see, we didn’t have to register anything related to the **IShareService** in the **App** class: Prism did everything on its own. Of course, if we want to achieve the same result also when the app is running on iOS or on Windows 10, we need to create also an implementation of the **IShareService** interface in the other platform specific projects.
        
Pretty cool, isn’t it?
                
### Wrapping up
        
We have concluded our journey in learning how the most recent version of the Prism framework has been greatly improved, specifically when it comes to support Xamarin Forms. Thanks to Prism and Xamarin, it will be much easier to create great cross platform application using the MVVM pattern and leveraging the same skills you have learned as a developer by creating WPF, Silverlight, Windows Store or UWP apps. As a final reminder, don’t forger that all the samples used in this series of posts are available on my GitHub account: [https://github.com/qmatteoq/XamarinForms-Prism](https://github.com/qmatteoq/XamarinForms-Prism "https://github.com/qmatteoq/XamarinForms-Prism")
        
Happy coding!