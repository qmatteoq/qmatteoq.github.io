---
id: 3382
title: How to apply a background to a Panorama using binding
date: 2013-05-31T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=3382
permalink: /how-to-apply-a-background-to-a-panorama-using-binding/
categories:
  - Windows Phone
tags:
  - Panorama
  - Windows Phone
---
In the next few days I’m going to publish a new application called Movies Tracker, which can be used to keep track of the movies you’ve already seen or that you would like to see. The application is totally built with the MVVM pattern, using Caliburn Micro as framework. The application makes great use of the Panorama control: in the main page, I use it to display different movies categories; in the movie detail, I use it to display different information about the movie (the poster, the cast, the story, etc.). To give to the application a better look and feel, I’ve decided to use as a background of every Panorama an image taken from the movie, kindly provided by The Movie Database. And here comes the problem: since I’m using MVVM, the path of the image I want to display as background is available in the ViewModel, so I need to assign it as Panorama’s background using binding, like in the following sample:

<pre class="brush: xml;">&lt;phone:Panorama&gt;
    &lt;phone:Panorama.Background&gt;
        &lt;ImageBrush ImageSource="{Binding Path=BackgroundImage}" /&gt;
    &lt;/phone:Panorama.Background&gt;
           
    &lt;phone:PanoramaItem Header="item 1" /&gt;
    &lt;phone:PanoramaItem Header="item 1" /&gt;

&lt;/phone:Panorama&gt;</pre>

Nothing strange right? So, which is the problem? That the above code simply doesn’t work: when you assign, in your ViewModel, an image path to the **BackgroundImage** property ****nothing happens. Unfortunately, the Panorama controls has a bug, so binding an image path to its Background property simply doesn’t work.

And here comes attached properties to the rescue! Attached properties is a powerful way to extend an existing control, provided by the XAML infrastructure: basically, you can add your custom property to an already existing control and you can decide the logic to apply when that property is set with a value. The workaround to this bug is to create an attached property, where we’re going to set the path of the image. When this value is set, we’re going to manually set the **Background** property of the control with the image from code: this way, everything will work fine, because the bug happens only when you use binding.

But we’re going to do more and overcome, at the same time, a limitation of the **ImageBrush** control: the **ImageSource** property, which contains the image path, supports only remote paths or local paths inside the Visual Studio project, so it’s ok to do something like in the following samples:

<pre class="brush: xml;">&lt;phone:Panorama&gt;
    &lt;phone:Panorama.Background&gt;
        &lt;ImageBrush ImageSource="http://www.mywebsite.com/background.png" /&gt;
    &lt;/phone:Panorama.Background&gt;
           
    &lt;phone:PanoramaItem Header="item 1" /&gt;
    &lt;phone:PanoramaItem Header="item 1" /&gt;

&lt;/phone:Panorama&gt;

&lt;phone:Panorama&gt;
    &lt;phone:Panorama.Background&gt;
        &lt;ImageBrush ImageSource="/Assets/Images/Background.png" /&gt;
    &lt;/phone:Panorama.Background&gt;
           
    &lt;phone:PanoramaItem Header="item 1" /&gt;
    &lt;phone:PanoramaItem Header="item 1" /&gt;

&lt;/phone:Panorama&gt;</pre>

What if the image is stored in the Isolated Storage? You can’t assign it using binding, but you’ll have to manually load in the code using a **BitmapImage**, like in the following sample:

<pre class="brush: csharp;">using (IsolatedStorageFile store = IsolatedStorageFile.GetUserStoreForApplication())
{
    using (IsolatedStorageFileStream stream = store.OpenFile("Background.png", FileMode.Open))
    {
        BitmapImage image = new BitmapImage();
        image.SetSource(stream);
        ImageBrush brush = new ImageBrush
        {
            Opacity = 0.3,
            Stretch = Stretch.UniformToFill,
            ImageSource = image,

        };
        panorama.Background = brush;        
    }
}</pre>

The problem of this approach is that it works only in code behind, because you’ll need a reference (using the **x:Name** property) to the Panorama control: you can’t do that in a ViewModel. We’re going to support also this scenario in our attached property: if the image path is prefixed with the **isostore:/** protocol, we’re going to load it from the isolated storage.

Let’s see the code of the attached property first:

<pre class="brush: csharp;">public class BackgroundImageDownloader
{

    public static readonly DependencyProperty SourceProperty =
        DependencyProperty.RegisterAttached("Source", typeof(string), typeof(BitmapImage), new PropertyMetadata(null, callback));


    public static void SetSource(DependencyObject element, string value)
    {
        element.SetValue(SourceProperty, value);
    }

    public static string GetSource(DependencyObject element)
    {
        return (string)element.GetValue(SourceProperty);
    }

    private static async void callback(DependencyObject d, DependencyPropertyChangedEventArgs e)
    {
        Panorama panorama = d as Panorama;

        if (panorama != null)
        {
            var path = e.NewValue as string;
            {
                if (!string.IsNullOrEmpty(path))
                {
                    if (path.StartsWith("isostore:/"))
                    {
                        string localPath = path.Substring(10);
                        using (IsolatedStorageFile store = IsolatedStorageFile.GetUserStoreForApplication())
                        {
                            using (IsolatedStorageFileStream stream = store.OpenFile(localPath, FileMode.Open))
                            {
                                BitmapImage image = new BitmapImage();
                                image.SetSource(stream);
                                ImageBrush brush = new ImageBrush
                                {
                                    Opacity = 0.3,
                                    Stretch = Stretch.UniformToFill,
                                    ImageSource = image,

                                };
                                panorama.Background = brush;        
                            }
                        }
                    }
                    else
                    {
                        BitmapImage image = new BitmapImage(new Uri(path, UriKind.Absolute));
                        ImageBrush brush = new ImageBrush
                        {
                            Opacity = 0.3,
                            Stretch = Stretch.UniformToFill,
                            ImageSource = image,

                        };
                        panorama.Background = brush;
                    }
                }
            }
        }
    }
}</pre>

We’ve defined a new property called **Source**, which can be used with a Panorama control: when a value is assigned to the property, the method called **callback** is executed and it contains two important parameters; the first one, **d**, which type is **DependencyObject**, is the control which the property has been attached to (in our case, it will always be a Panorama control); the second one, which type is **DependencyPropertyChangedEventArgs**, contains the value assigned to the property.

With these information we’ll be able to get a reference to the Panorama control which we want to apply the background to and the path of the image to load. We retrieve the path using the **NewValue** property of the **DependencyPropertyChangedEventArgs&nbsp;** object and we check if it starts with **isostore:/** or not. If that’s the case, we load the image from the Isolated Storage using the Storage APIs: once we get the image’s stream, we can create a new **BitmapImage** object and assign to it using the **SetSource** property. In the end, we create a new **ImageBrush** object and we assign the image as **ImageSource**: this way, we have a proper **ImageBrush** that we can assign to the **Background** property of the Panorama.

In case the image doesn’t come from the isolated storage, we simply create a new **BitmapImage** passing the Url, then we repeat the same operation as before: we create a new **ImageBrush** object, we assign the image as **ImageSource** and we set it as value of the **Background** property of the **Panorama** control.

How to use this attached property? First, we need to define in the XAML the namespace that contains our property (in the sample, I’ve called it **panoramaBinding**), than we can simply use the following code:

<pre class="brush: xml;">&lt;phone:Panorama panoramaBinding:BackgroundImageDownloader.Source="{Binding BackgroundPath}"&gt;
    
    &lt;phone:PanoramaItem Header="item 1" /&gt;
    &lt;phone:PanoramaItem Header="item 2" /&gt;

&lt;/phone:Panorama&gt;</pre>

Thanks to our attached property, **BackgroundPath** can be either a remote url or a local url: we just need to remember to add the **isostore:/** prefix in the second case.

Here is a sample ViewModel where the **BackgroundPath** is set as a remote url:

<pre class="brush: csharp;">public class MainViewModel : ViewModelBase
{
    private string backgroundPath;

    public string BackgroundPath
    {
        get { return backgroundPath; }
        set
        {
            backgroundPath = value;
            RaisePropertyChanged(() =&gt; BackgroundPath);
        }
    }

    public MainViewModel()
    {
        BackgroundPath = "http://4.bp.blogspot.com/-m0CqTN988_U/USd9rxvPikI/AAAAAAAADDY/4JKRsm3cD8c/s1600/free-wallpaper-downloads.jpg";
    }
}</pre>

And here’s another sample where I’ve defined a command, that is triggered when the user presses a button in the page: using&nbsp; the new Windows Phone 8 APIs I copy some pictures in the isolated storage and then I set the **BackgroundPath** property using a local path:

<pre class="brush: csharp;">public class MainViewModel : ViewModelBase
{
    private string backgroundPath;

    public string BackgroundPath
    {
        get { return backgroundPath; }
        set
        {
            backgroundPath = value;
            RaisePropertyChanged(() =&gt; BackgroundPath);
        }
    }

    public RelayCommand LoadImages
    {
        get
        {
            return new RelayCommand(async () =&gt;
                {
                    StorageFolder folder = await Package.Current.InstalledLocation.GetFolderAsync("Assets\\Images\\");
                    IReadOnlyList&lt;StorageFile&gt; files = await folder.GetFilesAsync();
                    foreach (StorageFile file in files)
                    {
                        await file.CopyAsync(ApplicationData.Current.LocalFolder, file.Name, NameCollisionOption.ReplaceExisting);
                    }

                    BackgroundPath = "isostore://Balls.jpg";
                });
        }
    }
}
</pre>

In both cases the result will be what you expect: the image will be displayed as background of the Panorama control. However, be aware that loading too big images can lead to many performance issues, since they can consume a lot of memory. But we’ll talk about this in another post <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none; border-left-style: none; border-bottom-style: none; border-right-style: none" alt="Smile" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/05/wlEmoticon-smile.png?w=640" data-recalc-dims="1" />

In the meantime, you can play with the sample project!

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:3ecfd088-220b-42cc-9d93-d39050315f7f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/05/PanoramaBinding3.zip" target="_blank">Download the sample project</a>
  </p>
</div>