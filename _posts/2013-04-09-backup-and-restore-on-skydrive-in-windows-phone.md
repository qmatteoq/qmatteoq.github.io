---
id: 2612
title: Backup and restore on Skydrive in Windows Phone 7
date: 2013-04-09T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=2612
permalink: /backup-and-restore-on-skydrive-in-windows-phone/
categories:
  - Windows Phone
tags:
  - Microsoft
  - Skydrive
  - Windows Phone
---
I’m working on a big update for my Speaker Timer application and one of the features I’m implementing is Skydrive’s support, to create a backup of the data and restore it in case the user buys a new phone or he has to reset it. There are many tutorials on the web about this scenario, but I think that all of them are missing some crucial information: some tutorials only explain how to upload files on Skydrive and not how to download them; the official documentation explains how to execute a download, but it mentions stuff like “file id” without explaining in details how to get it.

For this reason� I’ve decided to write a tutorial on my own: I hope you’ll find it useful for your applications. Please keep in mind that this tutorial is based on Windows Phone 7: on Windows Phone 8 the approach is really similar, but the Live SDK supports async methods instead of using the callback pattern.

### Prepare the environment

The first thing to do is to add the **Live SDK** to your project, which is the SDK provided by Microsoft to interact with the Live services. You can use it to identify the user, to get some information about it, to access to his remote pictures and so on. One of the supported feature is Skydrive’s access: you can read and write files from the user’s cloud storage. You can download the Live SDK for Windows Phone from <a href="http://msdn.microsoft.com/en-us/live/ff621310.aspx" target="_blank">the official website</a> or you can simply add it using <a href="http://nuget.org/packages/LiveSDK/" target="_blank">NuGet</a>.

The second step is to register your application in the Live Developer Portal: this step is required to get access to the **Client Id**, which is a unique identifier needed by the Live services to identify your application. Once you’ve logged in with your Microsoft Account <a href="http://msdn.microsoft.com/en-us/live" target="_blank">on the portal</a> click on the **My apps** section and choose the option **Create application**.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/03/image_thumb2.png?resize=531%2C157" width="531" height="157" border="0" data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/03/image2.png)

In the first step you have to specify a unique identifier (in the **Application Name** field) and the primary language of your application. Once you’re ready, click on the **I accept button.** In the second and final step the only option you do is to specify that you’re developing a mobile application, by choosing **Yes** in the **Mobile client app** option. Remember to press **Save** to confirm.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/03/image_thumb3.png?resize=501%2C284" width="501" height="284" border="0" data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/03/image3.png)

In this page you’ll also get the **Client ID**, that we’re going to use soon to configure our application.

### Sign in with the Microsoft Account

The first operation to do in your application is to add sign in support: the user will have to login to his Microsoft Account before being able to interact with Skydrive. This operation is really simple, since the Live SDK includes a built in control that takes care of all the process. The first thing is to add, in the PhoneApplicationPage declaration in the XAML, the namespace that contains the control:

_xmlns:my=&#8221;clr-namespace:Microsoft.Live.Controls;assembly=Microsoft.Live.Controls&#8221;_

Now you are able to add the control in the page, like in the following sample:

<pre class="brush: xml;">&lt;my:SignInButton ClientId="your_client_id" 
    Scopes="wl.signin wl.skydrive_update" 
    Branding="Skydrive" 
    TextType="SignIn" 
    SessionChanged="SignInButton_OnSessionChanged" 
    VerticalAlignment="Top"
/&gt;</pre>

The control is simply a button, that will take care of the all operations needed to do the login: when the user taps on it, a web view is opened, asking for the Microsoft account credentials. Once the login process is completed, in the application you’ll get the information if the login operation is successful or not.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/03/image_thumb4.png?resize=250%2C400" width="250" height="400" border="0" data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/03/image4.png)[<img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px 0px 0px 10px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/03/image_thumb5.png?resize=254%2C421" width="254" height="421" border="0" data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/03/image5.png)

The name of the control is **SignInButton** and it offers many properties to customize it. The most important one is **ClientId**, which contains the unique identifier of the application that you received after registering your application on the developer portal. One another important option is **Scopes**, that can be used to specify which features of the Live platform you’re going to use (you can find a list of all the available scopes <a href="http://msdn.microsoft.com/en-us/library/live/hh243646.aspx" target="_blank">in this page</a>): in this case we just need **wl.signin** (that is the basic one, needed to support authentication) and **wl.skydrive_update** (which is needed to get read and write access to Skydrive). If you want to customize the look & feel of the button you can use the **Branding** and **TextType** options: the first one is used to choose the logo that will be displayed on the button (since we’re going to interact with Skydrive, we use the **Skydrive** option), the second one to choose the login / logout texts that will be used as label (you can also customize them using the **SignInText** and **SignOutText** properties).

Now it’s time to write some code: the button offers an event called **SessionChanged,** that is invoked when the user is interacting with the control and he’s trying to login. Here is how to manage the event in the code:

<pre class="brush: csharp;">private void SignInButton_OnSessionChanged(object sender, Microsoft.Live.Controls.LiveConnectSessionChangedEventArgs e)
{
    if (e.Status == LiveConnectSessionStatus.Connected)
    {
        client = new LiveConnectClient(e.Session);
        MessageBox.Show("Connected!");
    }
}</pre>

The **e** parameter ****contains the status of the operation: with the **Status** property (which type is **LiveConnectSessionStatus**) we are able to know if the login operation completed successfully (**LiveConnectSessionStatus.Connected**). In this case, we can create a new instance of the **LiveConnectClient** class, which is the base class needed to perform all the operations with the Live services. As parameter you have to pass the identifier of the current session, which is stored in the **Session** property of the method’s parameters.

### Backing up the file

Now that the user is logged in and you have a valid **LiveConnectClient** object, you’re ready to backup your files on Skydrive. In this sample, we’re going to save a single file in the Skydrive’s root:

<pre class="brush: csharp;">private void OnBackupClicked(object sender, RoutedEventArgs e)
{
    client.UploadCompleted += client_UploadCompleted;
    using (IsolatedStorageFile storage = IsolatedStorageFile.GetUserStoreForApplication())
    {
        IsolatedStorageFileStream stream = storage.OpenFile("Sessions.xml", FileMode.Open);
        client.UploadAsync("me/skydrive", "SpeakerTimer.bak", stream, OverwriteOption.Overwrite);
    }
}

void client_UploadCompleted(object sender, LiveOperationCompletedEventArgs e)
{
    if (e.Error == null)
    {
        MessageBox.Show("Upload successfull!");
    }
}</pre>

The first thing we do is to subscribe to the **UploadCompleted** event exposed by the **LiveConnectClient** class, which is invoked when the upload operation is completed: this is needed because the operation is asynchronous. Then, using the storage APIs, we get the stream of the file we want to backup: in my sample, it’s a file called **Sessions.xml** stored in the isolated storage’s root. In the end we start the upload operation by calling the **UploadAsync()** method of the client, which accepts:

  * The path where to save the file on the Skydrive’s storage. With using the “**me/skydrive”** syntax we’re going to save the file in the Skydrive’s root;
  * The name of the file that will be saved on Skydrive;
  * The stream of the file to save: it’s the one that we’ve retrieved using the **OpenFile()** method of the **IsolatedStorageFile** class;
  * What to do in case the file on Skydrive already exists: in our sample we simply overwrite it by using the **OverwriteOption.Overwrite** value.

When the **UploadCompleted** event is raised we simply check if an error has occurred, by checking the value of the **Error** property of the **LiveOperationCompletedEventArgs** parameter: in case it’s null, we show a message with the notification that the upload has completed successfully. If we did everything correct, we’ll find the file in our Skydrive’s storage.

### Restoring the file

The backup operation was very simple to accomplish; the restore operation, instead, is a little bit trickier, because every file and folder on Skydrive is identified by a unique id and you can’t access to the file we’ve uploaded using its name. One way to get it is using the parameters returned by the **UploadCompleted** method, which contain, in the **Result** property, a dictionary with all the file’s properties, so by using the following code we are able to retrieve it.

<pre class="brush: csharp;">void client_UploadCompleted(object sender, LiveOperationCompletedEventArgs e)
{
    if (e.Error == null)
    {
        string fileId = e.Result["id"].ToString();
        MessageBox.Show("Upload successfull!");
    }
}</pre>

The problem with this approach is that, before getting the id of the file, we need to upload it. The most common scenario, instead, is that the user has made a reset or purchased a new phone and he wants to restore the backup after reinstalling the application: in this case no upload operations have been made, so we need to find another way to get the id of the file. The solution is to use the **GetAsync()** method of the client, that can be used to retrieve all the Skydrive files and folders and access to their properties. We’re going to get a list of all the Skydrive files and folders and, by exploring their properties, we’re going to find the id of the file which name is **SpeakerTimer.bak**.

<pre class="brush: csharp;">private void OnRestoreClicked(object sender, RoutedEventArgs e)
{
    string id = string.Empty;
    client.GetCompleted += (obj, args) =&gt;
    {
        List&lt;object&gt; items = args.Result["data"] as List&lt;object&gt;;
        foreach (object item in items)
        {
            Dictionary&lt;string, object&gt; file = item as Dictionary&lt;string, object&gt;;
            if (file["name"].ToString() == "SpeakerTimer.bak")
            {
                id = file["id"].ToString();
            }
        }
    };

    client.GetAsync("me/skydrive/files");
}</pre>

The **GetAsync()** method of the client accepts as parameter the path we want to explore: by using the **me/skydrive/files** syntax we get the list of all the files and folders inside the root. Since the method is asynchronous, we subscribe to the **GetCompleted** event, which is raised when the operation is done. The **Result** property of the parameter contains a collection of all the available files and folders, inside the item identified by the **Data** collection (it’s a dictionary). Since it’s a collection, we need to do a cast to the **List<object>** type. Every object inside this collection is another **Dictionary<string, object>,** that contains all the properties of the file or folder. By using a foreach we iterate over all the files and folders and, for each one, we check the value of the **name**’s property: if the value is the one we’re expecting (**SpeakerTimer.bak**), we get the **id** property and we store it (it will be something like _file.8c8ce076ca27823f.8C8CE076CA27823F!129_).

Now we’re ready to execute the real download operation:

<pre class="brush: csharp;">client.DownloadCompleted += (o, a) =&gt;
                                {
                                    Stream stream = a.Result;
                                    using (IsolatedStorageFile storage = IsolatedStorageFile.GetUserStoreForApplication())
                                    {
                                        using (
                                            IsolatedStorageFileStream fileToSave = storage.OpenFile("Sessions.xml", FileMode.Create,
                                                                                                    FileAccess.ReadWrite))
                                        {
                                            stream.CopyTo(fileToSave);
                                            stream.Flush();
                                            stream.Close();
                                        }
                                    }
                                };

client.DownloadAsync(string.Format("{0}/content", id));</pre>

We use the **DownloadAsync()** method of the client, passing as parameter the id of the file we’ve just retrieved. **It’s really important** to add the **/content** suffix to the id: many tutorials are missing this information, but it’s crucial because, otherwise, you’ll get the JSON with all the file’s properties instead of the real content.

Since, as usual, the operation is asynchronous, we subscribe to the **DownloadCompleted** event, that is invoked when the download is completed and we have access to the downloaded file, which is stored, as a stream, in the **Result** property of the method’s parameters. By using, again, the storage APIs, we save the stream we’ve downloaded in the Isolated Storage, by creating a new file (using the **OpenFile()** method and passing **FileMode.Create** as option) and by copying the downloaded stream in the local stream.

Here is the full restore method, where the two operations are executed at the same time:

<pre class="brush: csharp;">private void OnRestoreClicked(object sender, RoutedEventArgs e)
{
    string id = string.Empty;
    client.GetCompleted += (obj, args) =&gt;
    {
        List&lt;object&gt; items = args.Result["data"] as List&lt;object&gt;;
        foreach (object item in items)
        {
            Dictionary&lt;string, object&gt; file = item as Dictionary&lt;string, object&gt;;
            if (file["name"].ToString() == "SpeakerTimer.bak")
            {
                id = file["id"].ToString();
            }
        }

        client.DownloadCompleted += (o, a) =&gt;
                                        {
                                            Stream stream = a.Result;
                                            using (IsolatedStorageFile storage = IsolatedStorageFile.GetUserStoreForApplication())
                                            {
                                                using (
                                                    IsolatedStorageFileStream fileToSave = storage.OpenFile("Sessions.xml", FileMode.Create,
                                                                                                            FileAccess.ReadWrite))
                                                {
                                                    stream.CopyTo(fileToSave);
                                                    stream.Flush();
                                                    stream.Close();
                                                }
                                            }
                                        };

        client.DownloadAsync(string.Format("{0}/content", id));
    };

    client.GetAsync("me/skydrive/files");
}</pre>

### Conclusion

This is a simple solution to backup and restore your data in your application: there’s room for improvement (for example, you can encrypt the file before uploading it), but this tutorial covers pretty much all of the basic steps. Have fun!