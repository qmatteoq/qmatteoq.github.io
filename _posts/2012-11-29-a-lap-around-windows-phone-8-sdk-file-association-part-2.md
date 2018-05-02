---
id: 200
title: 'A lap around Windows Phone 8 SDK: file association &#8211; Part 2'
date: 2012-11-29T14:07:13+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=200
permalink: /a-lap-around-windows-phone-8-sdk-file-association-part-2/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows Phone
tags:
  - Windows Phone
---
In the previous post we started to take a deep look to one of the most interesting new features in Windows Phone 8: file association. The example we&#8217;re using in this tutorial is made by two applications: in the last post we&#8217;ve created the &#8220;launcher&#8221; app, that generates a text file and tries to open it. In this post, instead, we&#8217;re going to create the &#8220;reader&#8221; app, that will receive the file from the launcher app and will open and display it.

### The &#8220;reader&#8221; app: how to register for a file extensions

Let&#8217;s start to create the reader application: create a new Windows Phone 8 project (you can just use the basic **Windows Phone App** template) and let&#8217;s edit the manifest file. Unlucky, as I&#8217;ve already mentioned in the post I wrote <a href="http://wp.qmatteoq.com/?p=167" target="_blank">with a summary of all the new Windows Phone 8 features</a>, the new visual editor is not perfect and it doesn&#8217;t support all the scenarios: to register the file extensions we&#8217;ll need to manually edit the file, so you have to right click on the **WMAppManifest.xml** file into the **Properties** folder and choose **View code.**

File and protocol associations are registered in the **Extensions** section: in case it&#8217;s missing, you have to manually add it below the **Tokens** section. Here is how we register the **.log** extension:

<pre class="brush:xml">&lt;FileTypeAssociation Name="LogFile" TaskID="_default" NavUriFragment="fileToken=%s"&gt;
        &lt;SupportedFileTypes&gt;
          &lt;FileType ContentType="text/plain"&gt;.log&lt;/FileType&gt;
        &lt;/SupportedFileTypes&gt;
      &lt;/FileTypeAssociation&gt;</pre>

Every file type association is identified by a **FileTypeAssociation** node, with a unique **Name** property. The attributes **TaskID** and **NavUriFragment** defines how the file is passed: these two values are fixed and can&#8217;t be changed, they should always be exactly like you see in the code example.

Inside this node you can specify the file types you&#8217;re going to support, by adding a **FileType** node with the correct content type and extension. You have also the option to include a logo that identifies the custom file type, that is displayed by the operating system when needed (for example, near the file name of an attachment inside the Mail application). In this case, you have to create three different images (with size 33&#215;33, 69&#215;69 and 176&#215;176), add them to your project and include them in the **FileTypeAssociation** definition, like in the following example:

<pre class="brush:xml">&lt;FileTypeAssociation Name="LogFile" TaskID="_default" NavUriFragment="fileToken=%s"&gt;
&lt;Logos&gt;
           &lt;Logo Size="Small"&gt;log-33x33.png&lt;/Logo&gt;
           &lt;Logo Size="Medium"&gt;log-69x69.png&lt;/Logo&gt;
           &lt;Logo Size="Large"&gt;log-176x176.png&lt;/Logo&gt;
       &lt;/Logos&gt;
        &lt;SupportedFileTypes&gt;
          &lt;FileType ContentType="text/plain"&gt;.log&lt;/FileType&gt;
        &lt;/SupportedFileTypes&gt;
      &lt;/FileTypeAssociation&gt;</pre>

Once you&#8217;ve registered your application it&#8217;s time to write some code. There are two steps here: to define a **UriMapper** and to create a page that will handle the received file.

### The UriMapper

Before talking about the UriMapper, it&#8217;s better to explain how the OS handles file association. When an application is launched as a consequence of a file request (in our example, another application is trying to open a .log file), it&#8217;s opened using a special Uri that has the following structure:

<pre class="brush:xml">/FileTypeAssociation?fileToken=89819279-4fe0-4531-9f57-d633f0949a19</pre>

After the fixed keyword **FileTypeAssociation** there&#8217;s a unique parameter called **fileToken**, which is a GUID that identifies the file. As we&#8217;ll see later, Windows Phone 8 exposes an API to get the file using this token.

Now that you have understood how file association works under the hood, it should be easy to understand what is the UriMapper class: basically, it&#8217;s a central class that gets called when the application is started and it&#8217;s able to check if the app was called using a special Uri so that it can redirect the user to a specific page of the application.

In our case, we&#8217;re going to check if the app was opened since another application has requested to open a log file: if the answer is yes, we&#8217;re going to redirect the user to a specific detail page, where the log will be displayed. Here is the code:

<pre class="brush:csharp">public class UriMapper : UriMapperBase
    {
        private string tempUri;

        public override Uri MapUri(Uri uri)
        {
            tempUri = uri.ToString();

            // File association launch
            if (tempUri.Contains("/FileTypeAssociation"))
            {
                // Get the file ID (after "fileToken=").
                int fileIDIndex = tempUri.IndexOf("fileToken=") + 10;
                string fileID = tempUri.Substring(fileIDIndex);

                // Get the file name.
                string incomingFileName =
                    SharedStorageAccessManager.GetSharedFileName(fileID);

                // Get the file extension.
                int extensionIndex = incomingFileName.LastIndexOf(&#039;.&#039;) + 1;
                string incomingFileType =
                    incomingFileName.Substring(extensionIndex).ToLower();

                // Map the .log files to the appropriate pages.
                switch (incomingFileType)
                {
                    case "log":
                        return new Uri("/LogDetail.xaml?fileToken=" + fileID, UriKind.Relative);
                    default:
                        return new Uri("/MainPage.xaml", UriKind.Relative);
                }

            }
            // Otherwise perform normal launch.
            return uri;
        }
    }</pre>

First you have to create a new class in your project, that should inherit from the **UriMapperBase** class. This way you&#8217;ll have to implement the **MapUri** method, that is called when the application is initialized and that carries, as a parameter, the Url.

If the Url contains the **FileTypeAssociation** string the app is opened after that another application has requested to open a file: in this case we get the token of the file by simply playing with string properties (since **FileTypeAssociation** and **fileToken** are always fixed). After that, let&#8217;s welcome the **SharedStoreAccessManager**, which is the class that is able to handle operations with files that are opened this way: for this scenario we&#8217;ll use just the **GetSharedFileName** method that, passing the token, returns the name of the file as it was defined by the original application.

Using the name, and using some other strings voodoo magic, we finally get the information we need: the extension of the file. This way we&#8217;re able to identify the file type and we&#8217;re able to redirect the user to the page that is able to process that kind of file. In our example we manage just the .log extension, so the **switch** statement contains just two entries: the **log** extension and the default, which is a redirect to the **MainPage** of the application.

In case we receive a log file, we redirect the user to a specific page called **LogDetail.xaml** and we attach the token to the Uri: as we&#8217;ll see in a few moments, the token will be needed to get the real file.

The last thing we have to do is to tell the application that we have a **UriMapper,** that should be parsed every time a navigation is issued: to do this we have to go in the **App.xaml.cs** and, in the **InitializePhoneApplication()** method, right after the **RootFrame** has been initialized, set the **UriMapper** property to the class we have created, like in the following example:

<pre class="brush: csharp;">private void InitializePhoneApplication()
{
    if (phoneApplicationInitialized)
        return;

    // Create the frame but don&#039;t set it as RootVisual yet; this allows the splash
    // screen to remain active until the application is ready to render.
    RootFrame = new PhoneApplicationFrame();
    RootFrame.Navigated += CompleteInitializePhoneApplication;
    RootFrame.UriMapper = new Helpers.UriMapper();

    // Handle navigation failures
    RootFrame.NavigationFailed += RootFrame_NavigationFailed;

    // Handle reset requests for clearing the backstack
    RootFrame.Navigated += CheckForResetNavigation;

    // Ensure we don&#039;t initialize again
    phoneApplicationInitialized = true;
}</pre>

### Let&#8217;s get this file!

Now it&#8217;s time to create the **LogDetail.xaml** page, that we&#8217;re going to use to display the content of the file. Add a new empty Windows Phone page to the application and give it the name **LogDetail.xaml**: in the code behind we&#8217;re going to manage the **OnNavigatedTo** event, that is invoked when the user navigates towards this page.

In this event we&#8217;re going to get the file token and use it to get the real file that has been &#8220;sent&#8221; by the other application. Here is the code:

<pre class="brush:csharp">protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            base.OnNavigatedTo(e);
            if (NavigationContext.QueryString.ContainsKey("fileToken"))
            {
                await SharedStorageAccessManager.CopySharedFileAsync(ApplicationData.Current.LocalFolder, "rss.log",
                                                               NameCollisionOption.ReplaceExisting,
                                                               NavigationContext.QueryString["fileToken"]);

            }
        }</pre>

The mechanism should be familiar if you&#8217;re a Windows Phone developer, since it&#8217;s similar to the one adopted by the OS for other scenarios (like navigation from a secondary tile): if the **NavigationContext** contains a query string parameter which name is **fileToke****n** we&#8217;re going to use again the **SharedStorageAccessManager** class and, this time, specifically the **CopySharedFileAsync.**

This method simply translates the token into a real file and copies it into the local storage of the current application. The requested parameter are:

  * The folder in the storage where to save the file, identified by a **StorageFolder** object. In the example we simply pass the **LocalFolder**object: this way the file is copied into the root of the storage.
  * The name of the file to save.
  * What to do in case the file already exists (in the example, we overwrite it).
  * The file token (that we retrieve from the query string parameter)

Once we have the file in our storage, we can simply do whatever we want. For example, we can read it as a string and display it in a TextBlock using the following **ReadFromFile** extension method:

<pre class="brush: csharp;">public static class FileExtensions
{
    public static async Task&lt;string&gt; ReadFromFile(string fileName,StorageFolder folder = null)
    {
        folder = folder ?? ApplicationData.Current.LocalFolder;
        var file = await folder.GetFileAsync(fileName);

        using (var fs = await file.OpenAsync(FileAccessMode.Read))
        {
            using (var inStream = fs.GetInputStreamAt(0))
            {
                using (var reader = new DataReader(inStream))
                {
                    await reader.LoadAsync((uint)fs.Size);
                    string data = reader.ReadString((uint)fs.Size);
                    reader.DetachStream();
                    return data;
                }
            }
        }
    }
}</pre>

Now, thanks to this extension method, in the **OnNavigatedTo** event of the **LogDetail** page we can do something like this:

<pre class="brush: csharp;">protected override async void OnNavigatedTo(NavigationEventArgs e)
{
    base.OnNavigatedTo(e);
    if (NavigationContext.QueryString.ContainsKey("fileToken"))
    {
        SharedStorageAccessManager.CopySharedFileAsync(ApplicationData.Current.LocalFolder, "rss.log",
                                                       NameCollisionOption.ReplaceExisting,
                                                       NavigationContext.QueryString["fileToken"]);

        string content = await FileExtensions.ReadFromFile("rss.log");
        log.Text = content;
    }
}</pre>

### It’s debugging time!

Debugging this scenario is very easy: just deploy both applications in the emulator or in the device (by right clicking on the two projects and choosing **Deploy** from the menu). Then execute the launcher app we’ve developed in the previous post, create the log file and then launch it using the **Open file** button. If you did everything correctly, you’ll see the reader application open up directly in the **LogDetail**, with the content of the RSS that has been downloaded by the launcher app.

How cool is that? <img class="wlEmoticon wlEmoticon-smile" style="border-style: none;" src="https://i1.wp.com/qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/11/wlEmoticon-smile.png?w=640" alt="Smile" data-recalc-dims="1" />

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:655a9806-e2a5-4396-8581-17948387864e" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <p>
    You can download a sample project <a href="http://qmatteoq-en.azurewebsites.net/wp-content/uploads/2012/11/SharaDataSample6.zip" target="_blank">from here.</a>
  </p>
</div>