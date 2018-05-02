---
id: 6560
title: Integrating Facebook in a Universal Windows app
date: 2016-02-12T16:00:00+00:00
author: qmatteoq
layout: post
guid: http://blog.qmatteoq.com/?p=6560
permalink: /integrating-facebook-in-a-universal-windows-app/
categories:
  - Universal Apps
  - UWP
  - wpdev
tags:
  - Facebook
  - Universal Windows Platform
  - Windows 10
---
Facebook is, without any doubt, one of the most popular social networks out there. There are many reasons why we would integrate their services into a mobile application: to make the login process easier, to allow sharing some content, to post something on the user’s timeline. <a href="http://blog.qmatteoq.com/integrating-facebook-in-your-windows-phone-app-using-the-facebook-app" target="_blank">I’ve already covered in the past on this blog</a> how to integrate Facebook in a Windows or Windows Phone application. In this post we’re going to see how to achieve this goal in a Universal Windows app for Windows 10 using a new Facebook SDK released <a href="https://github.com/microsoft/winsdkfb" target="_blank">by a Microsoft team on GitHub</a> a while ago. Specifically, we’re going to see how to retrieve the main information about the logged user and how to interact with the Graph APIs, the set of REST APIs provided by Facebook to access to their services. 

Let’s start! 

# 

# 

### Configuring the app

The first step to start using the Facebook SDK is registering the app on the <a href="https://developers.facebook.com/" target="_blank">Facebook Developer Portal</a>: every mobile or web app which uses the Facebook APIs needs to be connected to an app registered on the Facebook portal. Without this step, Facebook won’t authorize us to perform any operation. Let’s point our browser to [https://developers.facebook.com/](https://developers.facebook.com/ "https://developers.facebook.com/") and choose to register a new app.

[<img title="newapp" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="newapp" src="https://i2.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/newapp_thumb.png?resize=471%2C257" width="471" height="257"  data-recalc-dims="1" />](https://i1.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/newapp.png)

As you can see, the Windows platform isn’t listed by default, so you have to choose to perform a **basic** **setup** with the link at the bottom.

[<img title="facebook2" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="facebook2" src="https://i0.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/facebook2_thumb.png?resize=475%2C267" width="475" height="267"  data-recalc-dims="1" />](https://i2.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/facebook2.png)

The next step will ask you to set a display name (typically, it’s the same name of your application), an optional namespace (which acts as unique identifier for your app) and the category. In the end, press the **Create App ID** button. At the end of this step, your Facebook app will be created and ready to be configured. Now we need to add support to the Windows platform: we do it in the **Settings** section, which we can find in the menu on the left. In this page we’ll find the **Add platform** link, as highlighted in the below image:

[<img title="facebook3" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="facebook3" src="https://i0.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/facebook3_thumb.png?resize=466%2C223" width="466" height="223"  data-recalc-dims="1" />](https://i0.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/facebook3.png)

When you click on the link, this time, you’ll find an option called **Windows App** in the list. After you’ve chosen it, the following section will appear in the Settings page:

[<img title="facebook4" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="facebook4" src="https://i1.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/facebook4_thumb.png?resize=502%2C134" width="502" height="134"  data-recalc-dims="1" />](https://i1.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/facebook4.png)

In our case, since it’s a Universal Windows app, we need to fill the **Windows Store SID** field**;** feel free to ignore the **Windows Phone Store SID [BETA]** field, since it applies only to old Silverlight apps for Windows Phone.

The SID is the unique identifier that the Windows Store assigns to our application when we associate it using the **Store** menu in Visual Studio. However, we don’t have to do it in advance to properly register the app on the Facebook portal. It’s enough to use the **WebAuthenticationBroker** class to retrieve this information: in case the app isn’t associated yet, we’ll retrieve a temporary value which is enough for our tests. Here is how to retrieve the SID in a Universal Windows app:

<pre class="brush: csharp;">string sid = WebAuthenticationBroker.GetCurrentApplicationCallbackUri().ToString(); 
</pre>

You have to take note of this value: the simplest approach is to use the **Debug.WriteLine()** method to print the value in the Visual Studio’s Output Window.

Here is how a SID looks like:

<pre class="brush: plain;">ms-app://s-1-15-2-2031376880-2039615047-2889735989-2649504141-752107641-3656655002-2863893494/ 
</pre>

This is the value we need to to put into the **Windows Store SID** field of the Facebook portal. However, we need to remove the **ms-app://** prefix and the ending **/**. So, based on the previous sample, the SID we would need to add in the developer’s portal is:

<pre class="brush: plain;">s-1-15-2-2031376880-2039615047-2889735989-2649504141-752107641-3656655002-2863893494
</pre>

The last thing to do is to take note of another information provided by Facebook: the **App ID**, which will need later during the login phase. You can find it in the main dashboard of your app, as you can see in the following image:

[<img title="clip_image002" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="clip_image002" src="https://i2.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/clip_image002_thumb.png?resize=604%2C234" width="604" height="234"  data-recalc-dims="1" />](https://i2.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/clip_image002.png) 

&nbsp;

Now we’ready to move on and write some code.

### 

### Integrating the library

The next step is to integrate the library into your app. Unfortunately, it isn’t available on NuGet, so you’ll need to download the whole project and compile it together with your app. The SDK is available on <a href="https://github.com/microsoft/winsdkfb" target="_blank">GitHub</a>, so there are two ways to get it:

  1. Using the **Download ZIP** button, to download a zipped file with the whole content of the repository. 
      * Cloning the repository on your computer using GIT.</ol> 
    Personally, I suggest using the second approach: this way, you’ll be able to quickly get any update released by the team and keep your app always up-to-date. There are many ways to do it: you can choose the one you prefer, based on your experience with Git. The easiest one is to use the tools integrated in Visual Studio. Let’s see how:
    
      1. Open Visual Studio. 
          * Open the **Team Explorer** window. If it isn’t already available in your Visual Studio configuration, look for it in the **View** menu. 
              * You will find a section called **Local Git repositories.** Press the **Clone** button and set: 
                  1. In the first field, the URL of the GitHub repository, which is [https://github.com/microsoft/winsdkfb](https://github.com/microsoft/winsdkfb "https://github.com/microsoft/winsdkfb") 
                      * In the second field, the local path on your computer where you want to save the project.</ol> 
                      * Now press the **Clone** button: Visual Studio will download all the files in your local folder. Any time, by using Solution Explorer, you’ll be able to access to this repository and, by using the **Sync** button, to download the latest version of the source code from GitHub.</ol> 
                    [<img title="git" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="git" src="https://i2.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/git_thumb.png?resize=444%2C313" width="444" height="313"  data-recalc-dims="1" />](https://i0.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/git.png)
                    
                    Now that you have the library, you can add it to your Visual Studio solution that contains your Universal Windows app, right click on it, choose **Add existing project** and look for one of the projects of the Facebook SDK you’ve just cloned on your computer:
                    
                      1. If it’s a Universal Windows app for Windows 10 (so based on UWP), you need to add the project located at _FBWinSDK\FBSDK-UWP\FBSDK-UWP\FBSDK-UWP.vcxproj_ 
                          * If it’s a Windows Store app for Windows 8.1, you need to add the project located at _FBWinSDK\FBWinSDK\FBWinSDK.Windows\FBWinSDK.Windows.vcxproj_ 
                              * If it’s a Windows Store app for Windows Phone 8.1, you need to add the project located _FBWinSDK\FBWinSDK\FBWinSDK.WindowsPhone\FBWinSDK.WindowsPhone._vcxproj </ol> 
                            The last step is to right click on your app’s project, choose **Add reference** and, from the **Projects** section, look for the Facebook project you’ve just added. Now you’re finally ready to write some code.
                            
                            ### The login phase
                            
                            The authentication to Facebook services is handled, as for many others similar services, using the oAuth protocol: the user will never share his credentials directly in the app, but inside a Web View which content is provided directly by the service’s owner (Facebook, in this case). If the credentials are accepted, Facebook wil return to the app a token, which we’ll need to perform any operation against the Facebook APIs. This approach improves the user’s security: a malicious developer won’t be able to access, in any case, to the user’s credentials.
                            
                            The Facebook SDK we’ve added to our project makes easier to handle the login process: the Web View and the auhtentication will be handled directly by the library, which will return us automatically all the info about the logged user and a set of APIs to interact with the Graph APIs.
                            
                            Here is the full login procedure:
                            
                            <pre class="brush: plain;">private async void OnLogin(object sender, RoutedEventArgs e)
{
    FBSession session = FBSession.ActiveSession;
    session.WinAppId = Sid;
    session.FBAppId = AppId;

    List&lt;string&gt; permissionList = new List&lt;string&gt;();
    permissionList.Add("public_profile");
    permissionList.Add("email");

    FBPermissions permissions = new FBPermissions(permissionList);
    var result = await session.LoginAsync(permissions);
    if (result.Succeeded)
    {
        //do something
    }
}

</pre>
                            
                            The first step is to retrieve the current session, by using the static property **FBSession.ActiveSession**. If we haven’t completed the login procedure yet, this property will be empty: in this case, we need to move on and continue the login procedure.
                            
                            The first important properties to set are **WinAppId** and **FBAppId**. The first one is the SID of the application, the one we have previously retrieved using the **GetCurrentApplicationCallbackUri()** method of the **WebAuthenticationBroker** class. **FBAppId**, instead, is the App Id that we have noted before from the Facebook’s developer portal. The next step is to define which kind of operations we want to do with the Facebook APIs, by using the permissions mechanism. You can find the complete list of available permissions in the documentation <https://developers.facebook.com/docs/facebook-login/permissions>
                            
                            It’s imortant to highlight how Facebook doesn’t allow to get access to all the permissions by default. Only the basic ones, like **public_profile** or **email**, are automatically allowed. The most advanced ones (like **publish_actions** that allows to publish a post on behalf of the user) require that the Facebook app passes a review process, where the team will double check the reasons and the use cases for which you’re requesting such permissions. Don’t confuse it with the certification process done by Microsoft when you submit the app on the Store: this review is completely handled by Facebook.
                            
                            Permissions are defined into a collection of strings, which is passed as parameter when you initialize the **FBPermissions** object. The last step is to call the **LoginAsync()** method, passing as parameter the **FBPermissions** object you’ve just defined. When you call it, if the user has never logged in with your app, the SDK will display a popup, which will embed the Facebook login page, as you can see in the following image:
                            
                            [<img title="clip_image004" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="clip_image004" src="https://i2.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/clip_image004_thumb.jpg?resize=476%2C369" width="476" height="369"  data-recalc-dims="1" />](https://i1.wp.com/blog.qmatteoq.com/wp-content/uploads/2016/02/clip_image004.jpg) 
                            
                            The user will have to fill his credentials and give to our app the authorization to access to the permissions we’ve required. If everything goes well, the method will return us a **FBResult** object with the outcome of the operation. The first property we can leverage is called **Succeded**, which will tell us if the login operation went well or not. In case the answer is yes, we can leverage the **User** property of the **FBSession** class to get access to all the public information of the user. The following code stores into a variable the full name of the user:
                            
                            <pre class="brush: csharp;">var result = await session.LoginAsync(permissions);
if (result.Succeeded)
{
    string name = session.User.Name;
}

</pre>
                            
                            ### Interact with the Graph APIs
                            
                            As mentioned in the beginning of the post, Graph APIs are a set of REST APIs offered by Facebook that we can use to perform many operations, like publishing a post, liking a comment, etc. Let’s see how the SDK can help us to interact with these APIs with a real sample: retrieving the e-mail address of the user. It isn’t one of the information exposed by the public profile, so you won’t find an **Email** property in the **FBUser** class, but you’ll need to use the Graph APIs to get it.
                            
                            As every REST API, an operation is defined by:
                            
                              1. An endpoint, which is the URL we need to invoke to interact with the service. In our scenario, we need to use the ****<https://graph.facebook.com/v2.5/me> endpoint, followed by the query string parameter **fields** with the list of information we need. In our case, the full endpoint will be <https://graph.facebook.com/v2.5/me?fields=email>
                              2. A method exposed by the HTTP protocol, to be used in combination with the endpoint. Typically, when we’re retrieving some data we use the GET method; writing operations (like posting a status on Facebook), instead, are performed with the POST method. Our scenario falls in the first category, so we will perform a GET.
                            
                            The first thing we need is a class which maps the response that we’re going to receive from the API. For this purpose we can use the Graph API Explorer, a tool which we can use to simulate the operations with the Graph APIs and that is available at <https://developers.facebook.com/tools/explorer> 
                            
                            The tool features an address bar, where we can specify the endpoint and the HTTP method we want to use. After pressing the **Submit** button, we’ll see in the main windows the JSON response returned by the API. If, for example, we perform a test with the endpoint we’re interested into (**/me?fields=email**) this is the response we’ll get:
                            
                            <pre class="brush: plain;">{
  "email": "mymail@mail.com",
  "id": "10203404048039813"
} 
</pre>
                            
                            Visual Studio offers a built-in option to convert a JSON data into a class. We just need to add a new class to the project and, once we have opened the file, using the option **Paste Special –> Paste JSON as classes** which is available in the **Edit** menu. Visual Studio will generate the following class:
                            
                            <pre class="brush: csharp;">public class UserProfile
{
    public string id { get; set; }

    public string email { get; set; }
}

</pre>
                            
                            However, to properly use the Facebook SDK, we need also another element in the class: a static method which is able to deserialize the JSON returned by the service to convert it into a **UserProfile** object. We create it with the help of the popular JSON.NET library, which we need to install in the project using NuGet (<https://www.nuget.org/packages/Newtonsoft.Json/>).
                            
                            This is the complete definition of the class:
                            
                            <pre class="brush: csharp;">public class UserProfile
{
    public string id { get; set; }

    public string email { get; set; }

    public static UserProfile FromJson(string jsonText)
    {
        UserProfile profile = JsonConvert.DeserializeObject&lt;UserProfile&gt;(jsonText);
        return profile;
    }
}

</pre>
                            
                            The **FromJson()** method uses the **JsonConvert** class to take as input the JSON returned by Facebook and to return, as output, a **UserProfile** object. To understand why we have created such a method, we need to analyze the code required to interact with the Graph APIs:
                            
                            <pre class="brush: csharp;">private async void OnGet(object sender, RoutedEventArgs e)
{
    string endpoint = "/me";

    PropertySet parameters = new PropertySet();
    parameters.Add("fields", "email");

    FBSingleValue value = new FBSingleValue(endpoint, parameters, UserProfile.FromJson);
    FBResult graphResult = await value.GetAsync();

    if (graphResult.Succeeded)
    {
        UserProfile profile = graphResult.Object as UserProfile;
        string email = profile?.email;
        MessageDialog dialog = new MessageDialog(email);
        await dialog.ShowAsync();
    }
}

</pre>
                            
                            The first step is to define the endpoint. It’s important to highlight that the query string parameters can’t be added directly in the URL, but separately using the **PropertySet** collection, otherwise we will get an exception at runtime. This happens because the SDK, under the hood, automatically sets the base URL for the APis and adds the access token retrieved during the login phase. You can notice it from the fact that we have just set the value **/me** as endpoint; we didn’t have to specify the full URL with the [https://graph.facebook.com/v2.5](https://graph.facebook.com/v2.5/) prefix.
                            
                            Using the **PropertySet** property is quite simple: it’s a collection, where we can add key-value pairs composed by a key (the name of the property, in our case **fields**) and a value (the property’s value, in our case **email**). In the next line of code you’ll finally understand why we had to create a **FromJson()** method inside the **UserProfile** class: it’s one of the paramters required when we initialize the **FBSingleValue** object, which is the class exposed by the SDK to interact with the Graph APIs. The other two parameters are the endpoind and the **PropertySet** collection with the list of parameters.
                            
                            Now we are ready to perform the request. The **FBSingleValue** class offers many options, based on the HTTP method we need to use. In our case, since we need to perform a GET operation, we use the **GetAsync()** method, which will return us a **FBResult** object. It’s the same type of result we received during the login operation: this means that, also in this case, we can leverage the **Succeded** property to understand if the operation has completed succesfully or not. The difference is that, this time, we have also a **Result** property, which contains the result returned by the Graph API. It’s a generic **object**, since the Graph APIs don’t return a fixed structure; it’s our job to convert it into the type we expect, in this case a **UserProfile** object.
                            
                            The rest is up to us and depends by our app’s logic: in the previous sample code, we just show to the user a dialog with the retrieved mail address.
                            
                            This approach works for every interaction supported by the Graph APIs. What changes between one operation and the other is:
                            
                              1. The endpoint
                              2. The values to add to the **PropertySet** collection, based on the parameters required by the API.
                              3. The class to map the API’s response.
                              4. The method of the **FBSingleValue** class to call to perform the operation.
                            
                            For example, if instead of retrieving the user’s mail address we would have wanted to publish a post on the user’s timeline, we would have used the following code:
                            
                            <pre class="brush: csharp;">private async void OnPost(object sender, RoutedEventArgs e)
{
    PropertySet parameters = new PropertySet();
    parameters.Add("title", "Microsoft");
    parameters.Add("link", "https://www.microsoft.com/en-us/default.aspx");
    parameters.Add("description", "Microsoft home page");
    parameters.Add("message", "Posting from my Universal Windows app.");

    string path = "/me/feed";

    FBSingleValue sval = new FBSingleValue(path, parameters, FBReturnObject.FromJson);
    FBResult fbresult = await sval.PostAsync();

    if (fbresult.Succeeded)
    {
        // Posting succeeded
    }
    else
    {
        // Posting failed
    }
} 

</pre>
                            
                            The main difference here, other than the endpoint and the parameters, is that we’re dealing with a “write” operation. Consequently, we need to use the POST method of the HTTP protocol, which is translated into the **PostAsync()** method of the **FBSingleValue** class. As a reminder, remember that this method won’t work as it is: you’ll need to request the **publish_actions** permission, which is granted by Facebook only after the app has passed their review.
                            
                            ### 
                            
                            ### Wrapping up
                            
                            In this post we’ve learned how to leverage the new Facebook SDK in a Universal Windows app to interact with the services offered by the popular social network. If you want to learn more, you can refer to the GitHub project (<https://github.com/microsoft/winsdkfb>) and o the official documentation (<http://microsoft.github.io/winsdkfb/>). Happy coding!