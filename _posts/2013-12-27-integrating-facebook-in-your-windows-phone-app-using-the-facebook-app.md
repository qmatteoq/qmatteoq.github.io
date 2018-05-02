---
id: 5202
title: Integrating Facebook in your Windows Phone app using the Facebook app
date: 2013-12-27T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=5202
permalink: /integrating-facebook-in-your-windows-phone-app-using-the-facebook-app/
categories:
  - Windows Phone
tags:
  - Windows Phone
---
If you’re a Windows Phone user, you’ll probably know that Microsoft offers two Facebook applications: an <a href="http://www.windowsphone.com/s?appid=82a23635-5bd9-df11-a844-00237de2db9e" target="_blank">official one</a> and a <a href="http://www.windowsphone.com/s?appid=93da5d29-daf0-4783-9ed5-a87b33247ec6" target="_blank">beta one</a>. The second one is simply a “test field” for all the new features that, from time to time, are added to the application by the team: after a reasonable time, the features are considered stable and are “promoted” and added also to the official application. Some months ago Microsoft has added to the beta application a really interesting feature for developers: a way to use it as a “bridge” to integrate Facebook into our applications. This feature still relies on the existing <a href="http://facebooksdk.net/" target="_blank">Facebook SDK for .NET</a> (a library available on NuGet to easily integrate Facebook in our applications) but, instead of asking to devs to create their own login page in their apps, everything is made by the Facebook app. Here is the flow if you start using this new feature:

  1. Our application starts the login procedure.
  2. The Facebook app is opened: the user will have to eventually login and to confirm the permissions.
  3. The user is returned to our application but, this time, we will receive the Access Token that is needed by the Facebook SDK to perform any operation (like posting a message or getting information about the logged user).

The biggest pro of this approach is that we need to write less code and that, in most of the cases, the user will just have to grant the permissions to the app: since the user, probably, regularly uses the Facebook app, he’s already logged in, so he won’t have to insert his credentials every time. And if he doesn’t have the app? No problem: since the method is based on the protocol sharing feature that has been added in Windows Phone 8, the user will simply be asked if he wants to download the app from the Store.

Since a few days ago, this feature was available just for testing scenarios: most of the people use the official version, not the beta one. Plus, since the beta application is hidden in the Store (you can reach it just with the deep link), in case the user doesn’t have it the user will still be prompted to search for the app on the Store, but he won’t be able to find it. However, now the official application has been updated with all the latest beta improvements, including the authentication support, so there’s no reason why you shouldn’t use it!

This feature <a href="http://blogs.windows.com/windows_phone/b/wpdev/archive/2013/11/14/sign-into-windows-phone-8-apps-with-facebook-login.aspx" target="_blank">was detailed in a blog post</a> made by the team, which I used as a “training ground” to write this post. The official post is really good, but it can be hard to understand if you aren’t an experienced Windows Phone developer, since it assumes that the reader already knows many concepts like Uri associations, the UriMapper class, etc.

So, let’s see how to do it in a more detailed way.

### Publish your app

Probably you’re wondering why publishing your app, which should be the last step, is reported at the beginning of this post. To properly set up the authentication process, you’ll need to include in your application your real Application Id. The problem is that, when you create a new application with Visual Studio, a random id is generated and assigned to the app: when the app is submitted on the store, the real application id is generated and included in the manifest. The first step is to ask to the <a href="http://dev.windowsphone.com" target="_blank">Dev Center</a> to generate an application id for us: to do it, you just have to start the submission process and to upload the XAP file (in Step 2). Then, simply abort the process by returning in the main page of your dashboard: you’ll find your app in the **In-progress submissions** list. Click on it and switch to the **Details** section: you’ll find the **App ID** section already populated with the real application id. Copy it: we’re going to use it later. **Important!** When your app is finished and you’re ready to publish it, you’ll have to resume this pending submission and not to start a new one: otherwise, the application will get a new application id, breaking the Facebook authentication process we’re going to set up in the next steps.

### Register the application on Facebook

The second step is to register the application on Facebook: every application that needs to interact with the Facebook APIs has to be registered in the Facebook developer’s portal. This way, you’ll get some information that will be important to use the Facebook SDK (like the App Id assigned by Facebook). To do this, go to [https://developers.facebook.com](https://developers.facebook.com "https://developers.facebook.com"), click on the **Apps** link in the upper menu and choose **Create new app.** To interact with the Windows Phone application there are only two fields that are required:

  * The name of the application, which has to be set in the **Display Name** field.
  * How the application integrates with Facebook: choose **Windows App** from the list and, in the **Windows Phone Store ID [BETA]** section, paste the Application Id you previously copied from the Dev Center (you’ll have to remove the hypens so if, for example, your id is b94ca943-21f1-4ef7-9895-b44a32dc1230, you’ll have to past b94ca94321f14ef79895b44a32dc1230 as value).

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/12/image_thumb.png?resize=640%2C125" width="640" height="125" border="0" data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/12/image.png)

### 

### Register the protocol in the manifest

<a href="http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj206987(v=vs.105).aspx" target="_blank">Protocol sharing</a> is one of the new features introduced in Windows Phone 8 that allows an app to register a specific protocol (like foo:/), so that other apps can interact with your application by using that protocol. This technique is used to properly manage the Facebook login: the Facebook application, after the login, will call back your application using a specific Uri. This way, you’ll be able to intercept the call and to get the result of the login process. To register a protocol, you have to edit the manifest file: you’ll have to do it manually, since this scenario is not supported by the visual editor. Just right click on the manifest file (the one called **WMAppManifest.xml** inside the **Properties** folder) and choose **View code**: you’ll get access to the XML version of the manifest.

Under the section called **Tokens** you’ll have to add a new section called **Extensions** in the following way:

<pre class="brush: csharp;">&lt;Extensions&gt;
  &lt;Protocol Name="msft-b94ca94321f14ef79895b44a32dc1230" NavUriFragment="encodedLaunchUri=%s" TaskID="_default" /&gt;
&lt;/Extensions&gt;</pre>

The only thing you’ll need to change is the value in the **Name** attribute, which should be **msft-** followed by the same application id you’ve retrieved from the Dev Center and that you’ve pasted in the Facebook Developers Center (as you can notice, also in this case you’ll have to remove the hyphens).

### Start coding

Now you’re ready to write some code and to effectively start the login process. The first step is to install two libraries from NuGet, which will provide the needed APIs to interact with Facebook: the first one is the <a href="http://www.nuget.org/packages/Facebook/" target="_blank">Facebook for .NET SDK</a> (which we already mentioned), while the second one is the <a href="http://www.nuget.org/packages/Facebook.Client/" target="_blank">Facebook Client for .NET SDK</a> (which adds a set of libraries that are specific for Windows Phone and Windows Store apps). Please note that, to find the second package in NuGet, you have to extend the search also to pre-release packages.

The second step, which is not required but it will make things easier, is to use a class developed by Microsoft as a sample called **SessionStorage**: it’s a simple class that acts as a wrapper of the **IsolatedStorageSettings** class, but that is able to encrypt the data. This way, we’re going to save in safety the access token returned by Facebook. To create this helper simply creates a new class, called it **SessionStorage** and paste the following code:

<pre class="brush: csharp;">public class SessionStorage
{
    /// &lt;summary&gt;
    /// Key used to store access token in app settings
    /// &lt;/summary&gt;
    private const string AccessTokenSettingsKeyName = "fb_access_token";

    /// &lt;summary&gt;
    /// Key used to store access token expiry in app settings
    /// &lt;/summary&gt;
    private const string AccessTokenExpirySettingsKeyName = "fb_access_token_expiry";

    /// &lt;summary&gt;
    /// Key used to state in app settings
    /// &lt;/summary&gt;
    private const string StateSettingsKeyName = "fb_login_state";

    /// &lt;summary&gt;
    /// Tries to retrieve a session
    /// &lt;/summary&gt;
    /// &lt;returns&gt;
    /// A valid login response with access token and expiry, or null (including if token already expired)
    /// &lt;/returns&gt;
    public static FacebookSession Load()
    {
        // read access token
        string accessTokenValue = LoadEncryptedSettingValue(AccessTokenSettingsKeyName);

        // read expiry
        DateTime expiryValue = DateTime.MinValue;
        string expiryTicks = LoadEncryptedSettingValue(AccessTokenExpirySettingsKeyName);
        if (!string.IsNullOrWhiteSpace(expiryTicks))
        {
            long expiryTicksValue = 0;
            if (long.TryParse(expiryTicks, out expiryTicksValue))
            {
                expiryValue = new DateTime(expiryTicksValue);
            }
        }

        // read state
        string stateValue = LoadEncryptedSettingValue(StateSettingsKeyName);

        // return true only if both values retrieved and token not stale
        if (!string.IsNullOrWhiteSpace(accessTokenValue) && expiryValue &gt; DateTime.UtcNow)
        {
            return new FacebookSession()
            {
                AccessToken = accessTokenValue,
                Expires = expiryValue,
                State = stateValue
            };
        }
        else
        {
            return null;
        }
    }

    /// &lt;summary&gt;
    /// Saves an access token an access token and its expiry
    /// &lt;/summary&gt;
    /// &lt;param name="session"&gt;A valid login response with access token and expiry&lt;/param&gt;
    public static void Save(FacebookSession session)
    {
        SaveEncryptedSettingValue(AccessTokenSettingsKeyName, session.AccessToken);
        SaveEncryptedSettingValue(AccessTokenExpirySettingsKeyName, session.Expires.Ticks.ToString());
        SaveEncryptedSettingValue(StateSettingsKeyName, session.State);
    }

    /// &lt;summary&gt;
    /// Removes saved values for access token and expiry
    /// &lt;/summary&gt;
    public static void Remove()
    {
        RemoveEncryptedSettingValue(AccessTokenSettingsKeyName);
        RemoveEncryptedSettingValue(AccessTokenExpirySettingsKeyName);
        RemoveEncryptedSettingValue(StateSettingsKeyName);
    }

    /// &lt;summary&gt;
    /// Removes an encrypted setting value
    /// &lt;/summary&gt;
    /// &lt;param name="key"&gt;Key to remove&lt;/param&gt;
    private static void RemoveEncryptedSettingValue(string key)
    {
        if (IsolatedStorageSettings.ApplicationSettings.Contains(key))
        {
            IsolatedStorageSettings.ApplicationSettings.Remove(key);
            IsolatedStorageSettings.ApplicationSettings.Save();
        }
    }

    /// &lt;summary&gt;
    /// Loads an encrypted setting value for a given key
    /// &lt;/summary&gt;
    /// &lt;param name="key"&gt;The key to load&lt;/param&gt;
    /// &lt;returns&gt;
    /// The value of the key
    /// &lt;/returns&gt;
    /// &lt;exception cref="KeyNotFoundException"&gt;The given key was not found&lt;/exception&gt;
    private static string LoadEncryptedSettingValue(string key)
    {
        string value = null;
        if (IsolatedStorageSettings.ApplicationSettings.Contains(key))
        {
            var protectedBytes = IsolatedStorageSettings.ApplicationSettings[key] as byte[];
            if (protectedBytes != null)
            {
                byte[] valueBytes = ProtectedData.Unprotect(protectedBytes, null);
                value = Encoding.UTF8.GetString(valueBytes, 0, valueBytes.Length);
            }
        }

        return value;
    }

    /// &lt;summary&gt;
    /// Saves a setting value against a given key, encrypted
    /// &lt;/summary&gt;
    /// &lt;param name="key"&gt;The key to save against&lt;/param&gt;
    /// &lt;param name="value"&gt;The value to save against&lt;/param&gt;
    /// &lt;exception cref="System.ArgumentOutOfRangeException"&gt;The key or value provided is unexpected&lt;/exception&gt;
    private static void SaveEncryptedSettingValue(string key, string value)
    {
        if (!string.IsNullOrWhiteSpace(key) && !string.IsNullOrWhiteSpace(value))
        {
            byte[] valueBytes = Encoding.UTF8.GetBytes(value);

            // Encrypt the value by using the Protect() method.
            byte[] protectedBytes = ProtectedData.Protect(valueBytes, null);
            if (IsolatedStorageSettings.ApplicationSettings.Contains(key))
            {
                IsolatedStorageSettings.ApplicationSettings[key] = protectedBytes;
            }
            else
            {
                IsolatedStorageSettings.ApplicationSettings.Add(key, protectedBytes);
            }

            IsolatedStorageSettings.ApplicationSettings.Save();
        }
        else
        {
            throw new ArgumentOutOfRangeException();
        }
    }
}</pre>

Now it’s time to start the real login procedure, which is done using the **FacebookSessionClient** class: to create a new instance you’ll have to pass as parameter the Facebook application id, which you can find on the Facebook Developers portal page (see the following screenshot).

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/12/image_thumb1.png?resize=620%2C155" width="620" height="155" border="0" data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/12/image1.png)

Here is a full login sample:

<pre class="brush: csharp;">private void OnLoginClicked(object sender, GestureEventArgs e)
{
    FacebookSessionClient fb = new FacebookSessionClient("563659977051507");
    fb.LoginWithApp("basic_info,publish_actions,read_stream", "custom_state_string");
}</pre>

To effectively perform the login you have to use the **LoginWithApp** method, which requires two parameters:

  * The first one is the list of permissions you want to use. You can find all the available ones <a href="https://developers.facebook.com/docs/reference/login/" target="_blank">here</a>; the previous code is used to grant access to the user profile, to his timeline and to allow the app to post on behalf of the user.
  * The second parameter is a custom string that is returned to the application once the login procedure is completed: we’re going to use it to implement our own logic when the user is redirected back to our application (for example, to perform an action or to redirect the user to a specific page).

As we’ve previously described, this new login method is based on the Windows Phone 8 protocol sharing feature: this means that, when the Facebook app has completed the login, it calls back our application with a special Uri. We need to intercept it and to retrieve the query string parameters, that are needed to understand if the operation completed with success or not. The best way to achieve this task is to use a **UriMapper** class: you should be already familiar with this approach if you’ve already worked with scenarios like protocol sharing or speech APIs. Basically, this class acts as a middle man between every navigation: every time the user moves from a page to another, this class intercepts the navigation and he’s able to analyze the calling Uri and, eventually, to change it or to redirect the user to another page. This behavior is extended also to navigation from and towards the Start screen or another application: when the Facebook application has completed the login, the UriMapper will intercept it and will be able to extract the parameters we need.

To create the UriMapper, simply just add a new empty class to your project: you’ll have to inherit from the **UriMapperBase** class, which will force you to implement the **MapUri()** method. Here is the full code:

<pre class="brush: csharp;">public class FacebookUriMapper: UriMapperBase
{
    private bool facebookLoginHandled;

    public override Uri MapUri(Uri uri)
    {
        if (AppAuthenticationHelper.IsFacebookLoginResponse(uri))
        {
            FacebookSession session = new FacebookSession();
            try
            {
                session.ParseQueryString(HttpUtility.UrlDecode(uri.ToString()));

                // Handle success case
                // do something with the custom state parameter
                if (session.State != "custom_state_string")
                {
                    MessageBox.Show("Unexpected state: " + session.State);
                }
                else
                {
                    // save the token and continue (token is retrieved and used when the app
                    // is launched)
                    SessionStorage.Save(session);
                }
            }
            catch (Facebook.FacebookOAuthException exc)
            {
                if (!this.facebookLoginHandled)
                {
                    // Handle error case
                    MessageBox.Show("Not signed in: " + exc.Message);
                    this.facebookLoginHandled = true;
                }
            }
            return new Uri("/MainPage.xaml", UriKind.Relative);
        }
        // by default, navigate to the requested uri
        return uri;
    }
}</pre>

This code checks if the Uri used to open the app the one used by the Facebook app: in this case, we’re going to create a Facebook session by using the information we’ve got in return in the Uri. If it’s a valid Facebook session (do you remember the custom string we’ve passed to the **LoginWithApp()** method?), we use the **SessionStorage** class we’ve created before to store the session data.

The last step to enable the UriMapper is to register in the in the application’s frame: open the **App.xaml.cs** file and look for a method called **InitializePhoneApplication()**� (which, by default, is hidden in a collapsed region called **Phone application initialization**). Right after the **RootFrame** object has been initalized simply create a new instance of your UriMapper class and assign it to the **UriMapper** property, like in the following sample:

<pre class="brush: csharp;">RootFrame = new PhoneApplicationFrame();
RootFrame.Navigated += CompleteInitializePhoneApplication;
RootFrame.UriMapper = new FacebookUriMapper();</pre>

### Interacting with Facebook: getting data

Now you have a valid Facebook access token, which you can use to perform one of the operations offered by the **FacebookClient** class provided by the Facebook C# SDK. This class acts as a wrapper of the Facebook REST APIs and offers many methods to interact with Facebook by wrapping the standard HTTP commands (for example, GET to retrieve data or POST to submit changes).

The following sample shows how to retrieve the logged user’s name:

<pre class="brush: csharp;">private async void OnShowNameClicked(object sender, GestureEventArgs e)
{
    FacebookSession session = SessionStorage.Load();
    FacebookClient client = new FacebookClient(session.AccessToken);

    dynamic result = await client.GetTaskAsync("me");
    string name = result.name;
    MessageBox.Show(name);
}</pre>

The first step is to load the session we’ve previously stored after the login process by using the **Load()** method of the **SessionStorage** class. Now that we have a **FacebookSession** object we can use it to get the information we need: specifically, we need the value of the **AccessToken** property, which has to be passed as parameter when you create a new instance of the **FacebookClient** class.

Since, to retrieve information from Facebook, we have to issue a GET operation, we need to use the **GetTaskAsync()** asynchronous method, which is based on the <a href="https://developers.facebook.com/docs/opengraph/" target="_blank">Facebook’s Open Graph APIs.</a> As parameter it needs the Open Graph’s path: **me** is the path used to get information about the logged user.

The method returns a dictionary, which is a collection of data identified by a key: in this case, every key identifies one of the user’s information (for example, the key called **name** contains the user’s name). In the sample code you can see a different way to manage this scenario: by using the **dynamic** keyword, we are able to work with the value returned by the **GetTaskAsync()** method like if it’s a dynamic object (in a similar way when you work with dynamic languages like Javascript). This way, we are able to simply get the name of the user by asking for the value of the **name** property of the result. We could have used another property like **surname** or **birthday** to get other information about the user: you can see the full list <a href="https://developers.facebook.com/docs/graph-api/reference/user/" target="_blank">here</a>.

### Integrating with Facebook: posting data

Another common scenario when you want to integrate Facebook in your application is posting messages. Here is a sample code to post a message on the user’s timeline:

<pre class="brush: csharp;">private async void OnPostMessageClicked(object sender, GestureEventArgs e)
{
    FacebookSession session = SessionStorage.Load();
    FacebookClient client = new FacebookClient(session.AccessToken);

    var parameters = new Dictionary&lt;string, object&gt;();
    parameters.Add("message", "First test post using Facebook login");

    await client.PostTaskAsync("me/feed", parameters);
}</pre>

The first part of the code is the same we’ve seen before: by using the **SessionStorage** class we retrieve the access token to create a new instance of the **FacebookClient** class.

The biggest difference between this code and the previous one is that, in case of posting, we need to interact with the Open Graph APIs passing a parameter: we do it by creating a new set of parameters (which is a **Dictionary<string, object>** collection) and by adding a parameter called **message**. The value of the parameter is simply the text of the message we want to post.

In the end, we use the **PostTaskAsync()** method, since in this case we’re going to execute a POST request: other than the specifying the Open Graph APIs path (in this case, **me/feed**) we need to pass the input parameters we’ve just defined.

### Wrapping up

There are many other operations that you can do with the Facebook C# SDK: you can find many samples <a href="http://facebooksdk.net/docs/making-asynchronous-requests/" target="_blank">here</a>, just keep in mind that these samples are based on the old Facebook .NET SDK version, which was based on the callback approach for asynchronous method instead of the async / await one.

The biggest pro of this approach is that it saves you a lot of time working with Facebook login, which is probably the most complex scenario to implement when you have to deal with Facebook integration. In addition, it will offer a better experience to the user: since most of the time he’s already using the Facebook app, he won’t have to insert his credentials every time. The only downside, instead, is that it relies on a third party application: in case the user doesn’t have the official Facebook app (because, for example, he prefers to use another app or the mobile website) he will have to download it before using the Facebook integration in your app. This requirement may lead the user to change his mind and stop using your application.