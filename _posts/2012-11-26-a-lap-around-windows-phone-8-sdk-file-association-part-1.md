---
id: 198
title: 'A lap around Windows Phone 8 SDK: file association &#8211; Part 1'
date: 2012-11-26T14:51:27+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=198
permalink: /a-lap-around-windows-phone-8-sdk-file-association-part-1/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows Phone
tags:
  - Windows Phone 8
---
One of the biggest limitations for Windows Phone developers was the impossibility to share data between applications. This wasn&#8217;t limited to third party apps, but also file types that are supported by native applications (just think about Office files) couldn&#8217;t be opened directly: the workaround was to open the file using the browser, this way if the file type was supported by the OS it was opened with the correct application. The biggest issue of this workaround is that it worked only if the file was available online: it didn&#8217;t work with files that were stored in the isolated storage.

It&#8217;s with a big applause that we welcome protocol and file association in Windows Phone 8, a feature that comes directly from Windows 8 and that allows your application to register for a specific protocol or file extension; this way other applications can simply launch a request to open a specific file or URL and the OS will take care of redirecting the operation to one of the available apps that has registered for that file type.

There are two ways to share data: by registering for a **file association** and for a **protocol**. The first feature allows an application to register for a specific file extension (for example, .**log**) : when someone tries to open a file with that extension, the file is passed to the application and copied to its local storage, so that he can interact with it. The second one, that we will cover in another blog post, can be used to register for a specific protocol (for example, **log:/**), so that an application can send text data to another application simply by calling that protocol.

What happens when a third party application tries to open a file? The OS scans for apps that, in the manifest file, has registered to support that file type: if no apps are found, the user is prompted to go to the Store to download an application that is able to manage that kind of file. If multiple apps that are able to manage the same extension are installed on the device, the user is asked which one he wants to use; instead, if there&#8217;s only one application available it&#8217;s directly opened.

In this tutorial we&#8217;ll focus on file association: we&#8217;ll deal with the protocol association in another post. In the example we&#8217;re going to use for this tutorial we&#8217;ll create two separate Windows Phone 8 apps: the first one will be called &#8220;the launcher&#8221; and it will be the app that will launch the file request, that will be processed by &#8220;the reader&#8221; app, that will be the app that will &#8220;listen&#8221; for a specific type extension and that will receive the files.� We&#8217;ll register for the **.log** extension and we&#8217;ll use it to share a file with some text inside between the two applications.

### The &#8220;launcher&#8221; app: how to open a file

In the launcher app we&#8217;re going to accomplish two tasks: the first one is to create a log file, which will be a simple text file. The second one is to &#8220;launch&#8221; that file, so that the reader application can use it. Let’s start creating the log file: for this step we’re going to use a class that adds two simple extension methods to work with strings. The first one is called **WriteToFile** and can be used to write a string into a file in the local storage; the second one is called **ReadFromFile** and does the opposite task: reads a text file from the local storage and return it as a string.

### 

Let’s start to create a class that will contain our extension methods: to do that just create a new class (by right clicking on your project and choosing **Add new file**) and mark it as static. In this post we’ll see the **WriteToFile** method, since it’s the only one we’re going to use in the launcher app.

<pre class="brush: csharp;">namespace ShareData.Launcher.Helpers
{
    public static class FileExtensions
    {   
        public static async Task WriteToFile(this string contents, string fileName, StorageFolder folder = null)
        {
            folder = folder ?? ApplicationData.Current.LocalFolder;
            var file = await folder.CreateFileAsync(
                fileName,
                CreationCollisionOption.ReplaceExisting);
            using (var fs = await file.OpenAsync(FileAccessMode.ReadWrite))
            {
                using (var outStream = fs.GetOutputStreamAt(0))
                {
                    using (var dataWriter = new DataWriter(outStream))
                    {
                        if (contents != null)
                            dataWriter.WriteString(contents);

                        await dataWriter.StoreAsync();
                        dataWriter.DetachStream();
                    }

                    await outStream.FlushAsync();
                }
            }
        }
    }
}</pre>

The method takes a string and saves it into a file in the local storage, which name is specified as parameter. The method accepts also another parameter, which type is **StorageFolder** (that identifies a folder in the local storage): we can use it in case we want to store the file in a specific folder. If you don’t set this parameter, the file will be created in the storage’s root.

This method uses the standard WinRT APIs to create the file in the storage and uses an **OutputStream** and the **DataWriter** class to write the content of the string in the class.

The next step is to install the **Microsoft.Bcl.Async** package from NuGet, that we’ve learned to use [in this post](http://wp.qmatteoq.com/async-targeting-pack-for-visual-studio-2012-why-its-useful-also-for-windows-phone-8-projects/ "Async pack for Visual Studio 2012: why it’s useful also for Windows Phone 8 projects"): the purpose is to use the async **WebClient** methods to download the content of a RSS feed and to save it into a file.

Now let’s work on the UI: we’re simply going to add two buttons in the **MainPage.xaml**, one to download the XML and write the file, the other one to “open” it.

<pre class="brush: xml;">&lt;Grid x:Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0"&gt;
    &lt;StackPanel&gt;
        &lt;Button Content="Create file" Click="OnCreateFileButtonClicked" /&gt;
        &lt;Button Content="Open file" Click="OnOpenFileButtonClicked" /&gt;
    &lt;/StackPanel&gt;
&lt;/Grid&gt;</pre>

Thanks to the extension method we’ve created and to the **WebClient** extension methods we’ve added by installing the **Microsoft.Bcl.Async** package it’s very easy to accomplish our task, that is downloading a XML and storing into a file with .log extension:

<pre class="brush: csharp;">private async void OnCreateFileButtonClicked(object sender, RoutedEventArgs e)
{
    WebClient client = new WebClient();
    string result = await client.DownloadStringTaskAsync("http://feeds.feedburner.com/qmatteoq_eng");

    await result.WriteToFile("rss.log");
}</pre>

We simply download the RSS using the **DownloadStringTaskAsync** method (in this example, we download the RSS feed of this blog) and we save it into a file called **rss.log**, that is placed in the root of the local storage.

Now it’s time to open the file:

<pre class="brush: csharp;">private async void OnOpenFileButtonClicked(object sender, RoutedEventArgs e)
{
    StorageFile file = await ApplicationData.Current.LocalFolder.GetFileAsync("rss.log");
    Windows.System.Launcher.LaunchFileAsync(file);
}</pre>

Also in this case the operation is really easy: we get a reference to the file (using the **GetFileAsync** method) and then we open it using the **LaunchFileAsync** method of the **Launcher** class. What’s going to happen? At this stage of the tutorial, you will be prompted with a message that will tell you that you don’t have any� app that is able to open **.log** files and you will be asked if you want to look for it in the Store. For the moment, just ignore the message: in the following post we’re going to create the reader application, that will register the **.log** extension and that will be able to open our file.

If you want to give a try to this feature until I publish the next post, try to rename the file saved by the application to **rss.txt**, by simply changing the name that is passed both to the **result.WriteToFile()** and **LocalFolder.GetFileAsync()** methods. Since **.txt** is an extension registered by the operating system, the file will be opened using Word and you’ll be able to read the content of the XML file.

### To be continued

In the next post we’re going to developer our own reader app, that will be able to intercept and open the **file.log** file we’ve created in this post. Stay tuned!