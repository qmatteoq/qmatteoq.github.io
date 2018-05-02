---
id: 1832
title: 'How to save a picture captured with the new camera&rsquo;s API in the camera roll in Windows Phone 8'
date: 2013-01-31T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=1832
permalink: /how-to-save-a-picture-captured-with-the-new-cameras-api-in-the-camera-roll-in-windows-phone-8/
categories:
  - Windows Phone
tags:
  - Windows Phone 8
---
Windows Phone 8 has introduced new APIs to interact with the native camera and to take pictures and record videos. These APIs are much more powerful than the ones that are available in Windows Phone 7: you have full control over all the camera settings, like exposure, focus, flash and so on.

There’s a good example <a href="http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj662940(v=vs.105).aspx" target="_blank">in the MSDN documentation</a> about how to use them: unlucky the article has some minor issues, so you can face some problems. Let’s see, in details, how to use the new APIs and how to apply a workaround to fix these issues.

Usually the first goal you want to achieve when you develop a camera application is to display the camera’s stream in your application: for this purpose we’re going to use a **VideoBrush**, which is one of the available brush in XAML that can be used to customize a control.

We’re going to insert a **VideoBrush** inside a **Canvas**, to display the camera’s stream inside it:

<pre class="brush: xml;">&lt;Canvas Height="400" Width="400"&gt;
    &lt;Canvas.Background&gt;
        &lt;VideoBrush x:Name="video" Stretch="UniformToFill"&gt;
            &lt;VideoBrush.RelativeTransform&gt;
                &lt;CompositeTransform x:Name="previewTransform" CenterX=".5" CenterY=".5" /&gt;
            &lt;/VideoBrush.RelativeTransform&gt;
        &lt;/VideoBrush&gt;
    &lt;/Canvas.Background&gt;
&lt;/Canvas&gt;
</pre>

You can notice that we apply also a **CompositeTransform** to the brush: as we’ll see soon, this is needed to rotate the stream according to the phone’s rotation.

Now we need to get access to the camera in our code: we’re going to use the new class **PhotoCaptureDevice**, that has replaced the **PhotoCamera** class that was available in Windows Phone 7. The old class is still there but, if you don’t need to keep the compatibility with both platforms, I suggest you to use the new one, since it’s much more powerful.

We’re going to initialize the camera in the **OnNavigatedTo** event:

<pre class="brush: csharp;">public partial class MainPage : PhoneApplicationPage
{
    public MainPage()
    {
        InitializeComponent();
    }

    private PhotoCaptureDevice camera;

    protected override async void OnNavigatedTo(NavigationEventArgs e)
    {
        base.OnNavigatedTo(e);
        Size resolution = PhotoCaptureDevice.GetAvailableCaptureResolutions(CameraSensorLocation.Back).First();
        camera = await PhotoCaptureDevice.OpenAsync(CameraSensorLocation.Back, resolution);
        video.SetSource(camera);
        previewTransform.Rotation = camera.SensorRotationInDegrees;
    }
}
</pre>

The first step is to define which is the resolution we’re going to use to take our photos: for this reason we use the **GetAvailableCaptureResolutions()** method of the **PhotoCaptureDevice** class, passing as parameter which camera we want to use: **CameraSensorLocation.Back** for the rear camera and **CameraSensorLocation.Front** for the front camera. The method returns a list of all the supported resolutions by the camera, starting from the highest to the lowest. In our example we take the picture using the maximum available resolution, so we take just the first of the list and we pass it, as parameter, to the **OpenAsync** method of the **PhotoCaptureDevice** class, together (again) with the camera we want to use. We use the **await** keyword (and, as a consequence, we mark the **OnNavigatedTo** method with the **async** keyword) since it’s asynchronous. 

The last step is to display the camera’s stream on the screen, by passing the **PhotoCaptureDevice** object to the **SetSource** method of the **VideoBrush**. We also set the rotation of the transformation applied to the brush according to the rotation of the camera, using the **SensorRotationInDegrees** property.

**Important!** Notice that, to do this operation, we need to add in our class the namespace **Microsoft.Device**: this is required to set the **PhotoCaptureDevice** object as a source for the **VideoBrush,** since it adds an override of the **SetSource** method that accepts as parameter an **ICameraCaptureDevice** object, that is the base interface from which every new camera API inherits from.

If you just launch the application you’ll see the image recorded by the camera on the screen: if you test the application on the emulator, it will be generated a fake stream, since the emulator isn’t able to use the computer’s webcam.

### Take the picture

The new Windows Phone 8 APIs have introduced the concept of “sequence of frames”, which is a set of pictures taken at the same time. In the current version the frames are limited to 1, so you can capture sequences composed by just one picture.

Here is a sample code that takes the picture and saves it into the camera roll of the phone:

<pre class="brush: csharp;">private async void OnTakePhotoClicked(object sender, RoutedEventArgs e)
{
    CameraCaptureSequence cameraCaptureSequence = camera.CreateCaptureSequence(1);

    MemoryStream stream = new MemoryStream();
    cameraCaptureSequence.Frames[0].CaptureStream = stream.AsOutputStream();

    await camera.PrepareCaptureSequenceAsync(cameraCaptureSequence);
    await cameraCaptureSequence.StartCaptureAsync();

    stream.Seek(0, SeekOrigin.Begin);

    MediaLibrary library = new MediaLibrary();
    library.SavePictureToCameraRoll("picture", stream);
}

</pre>

First we create a new **CameraCaptureSequence** object, by using the **CreateCaptureSequence** method of the **PhotoCaptureDevice** object. As parameter we need to pass the number of frames to capture: as I’ve just explained, due to a limit in the current APIs, we have to pass 1 as value; other values are not supported.

Then we create a new **MemoryStream** object, that will hold in memory the photo captured by the camera: we connect the output of the stream to output of the first frame of the sequence, that is the photo that will be captured. We do this by using the **CaptureStream** property of the first stream, that is available in the **Frames** collection of the sequence.

In the end, we take the picture: first we prepare the sequence (by calling the **PrepareCaptureSequenceAsync** method of the **PhotoCaptureDevice** object) an then we call the **StartCatureAsync** method on the sequence. Both methods are asynchronous, so we use the **await** keyword.

Now that we have the picture, we save it in the phone media library: and here is where need to apply&nbsp; a workaround, that is not mentioned in the MSDN article. We need to set the stream’s position of the **MemoryStream** object to the beginning, otherwise you’ll get a cryptic **InvalidOperationException** error with the **An unexpected error has occured** message, that isn’t helpful. The credit to this workaround goes to <a href="http://nostromo.aspitalia.com" target="_blank">Marco Leoncini</a>, a dear friend of mine and a member of the ASPItalia crew, that has helped me to understand why the sample provided in the MSDN article didn’t work.

We have two options to save the picture in the library: in the sample we use the **SavePictureToCameraRoll** method of the **MediaLibrary** object to save the picture in the camera roll. We have also another method, **SavePicture**, that can be used to save the image in the **Saved pictues** folder of the media library. It’s up to you choosing which is the best for your needs: if you’re developing a camera-replacement application (for example, a Lens app) probably you would prefer to save it in the camera roll, to be consistent with the experience provided by the native application.

Happy coding!

P.S. I would like to thank you also <a href="http://nokiawpdev.wordpress.com/" target="_blank">Lance Mc Carthy</a>, a Nokia ambassador that helped me to track the issue and that it’s been very kind to quickly reply to my questions on Twitter.

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:f8ce5213-56a8-4632-8399-add7d00ffbc5" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/01/CameraSample2.zip" target="_blank">Download the sample project</a>
  </p>
</div>