---
id: 1192
title: 'Maps in Windows Phone 8 and Phone toolkit: a winning team &ndash; Part 1'
date: 2013-01-17T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=1192
permalink: /maps-in-windows-phone-8-and-phone-toolkit-a-winning-team-part-1/
categories:
  - Windows Phone
tags:
  - Maps
  - Nokia
  - Windows Phone
---
Maps in Windows Phone 8 is probably the biggest evidence of the great partnership between Nokia and Microsoft: in Windows Phone 8 the maps application installed on every device, even the ones made by other manufacturers, is powered by Nokia technologies.

The most interesting news for developers is that also the Maps control has been updated and it introduced many improvements: first of all, performance improvements. Bing Maps control wasn’t good in this and, especially if you tried to add layers or pushpin, you could have experienced really poor performances.

Second, the new control now offers lot of new options: you can enable 3D landmarks, you can switch between day and night colors, you can change the heading and the pitch, you can easily calculate routes and so on.

Adding the map control is very easy: in the XAML you need to add a reference to the **Microsoft.Phone.Maps** namespace:

_xmlns:maps=&#8221;clr-namespace:Microsoft.Phone.Maps.Controls;assembly=Microsoft.Phone.Maps&#8221;_

After that you can insert the **Map** control simply by declaring it in the XAML:

<pre class="brush: csharp;">&lt;maps:Map
    x:Name="myMap"
    Center="30.712474,-132.32691"
    ZoomLevel="17"
    Heading="45"
    Pitch="25"
    CartographicMode="Road"
    ColorMode="Dark"
    PedestrianFeaturesEnabled="True"
    LandmarksEnabled="True"
    /&gt;
</pre>

In this sample you can see an example of many of the properties that are available in the control:

  * **Center** is the location where the map is centered. It accepts a **GeoCoordinate** object, which is a complex object that store many geo location properties, like latitude, longitude, altitude and so on. 
      * **ZoomLevel** is the zoom level of the map, from 1 (very far) to 20 (very near). 
          * **Heading** is the rotation of the map. 
              * **Pitch** is the elevation of the map compared to the horizon. 
                  * **CartographicMode** is the type of map that is displayed, you can choose between **Aerial, Road, Terrain** or **Hybrid**. 
                      * **ColoreMode** can be used to improve the readability of the map during night (**Dark**) and day (**Light**). 
                          * **LandmarksEnabled** can be used to turn on the 3D representation of some important builidings in the map (like monuments, churches and so on). 
                              * **PedestrianFeaturesEnabled** can be used to turn elements that are useful for people that is using the map during a walking (like stairs). </ul> 
                            One of the coolest feature is that, with the new map control, is really easy to add a layer, that is a collection of elements that are displayed over the map, as you can see in this sample:
                            
                            <pre class="brush: csharp;">private void OnAddShapeClicked(object sender, RoutedEventArgs e)
{
    MapOverlay overlay = new MapOverlay
                             {
                                 GeoCoordinate = myMap.Center,
                                 Content = new Ellipse
                                               {
                                                   Fill = new SolidColorBrush(Colors.Red),
                                                   Width = 40,
                                                   Height = 40
                                               }
                             };
    MapLayer layer = new MapLayer();
    layer.Add(overlay);

    myMap.Layers.Add(layer);
}
</pre>
                            
                            &nbsp;
                            
                            For every element I want to display on the map I create a **MapOverlay** object, that has two main properties: **GeoCoordinate**, with the coordinates where the element should be placed, and **Content**, which is a generic object property so it accepts any XAML control. In this example, I’m creating a blue circle, using the **Ellipse** control, that will be placed in the same position where the map is centered.
                            
                            In the end I simply create a **MapLayer** and I add to it the **MapOverlay** object I’ve just created: I could have created many more **MapOverlay** object and added them to the same **MapLayer** (or also to a new one). The last step is to add the layer to the map, by simply adding it to the **Layers** collection, which is a property of the **Map** control.
                            
                            Here is the final result:
                            
                            [<img title="shape" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="shape" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/shape_thumb.png?resize=222%2C370" width="222" height="370"  data-recalc-dims="1" />](https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/shape.png)
                            
                            This feature is very powerful: since the **MapOverlay** object has a generic **Content** property, you can customize the overlay elements any way you want. But there’s a downside: there’s nothing ready to be used in the SDK for common scenarios like displaying a pushpin or the user’s location.
                            
                            ### 
                            
                            ### 
                            
                            So please say hello to the **Phone Toolkit**, that probably you already know since it’s one of the essentials toolkits for every Windows Phone developers, since it contains many useful controls, helpers and converters that are not available in the SDK, even some of the standard ones that are used in the native application. The Phone Toolkit is available on <a href="http://phone.codeplex.com/" target="_blank">Codeplex</a>, but the easiest way to add it to your project is using <a href="https://nuget.org/packages/WPtoolkit" target="_blank">NuGet</a>. In the latest release Microsoft has added, other than some new cool controls and transitions effects, some map utilities. In our scenario we can make use of two nice controls: **UserLocationMarker** and **Pushpin**. They both work in the same way: they are overlay elements that are placed on the map over a specific position and they offer a way to interact with them (so that, for example, the user can tap on a pushpin and do something). They only **look** **different**: the **Pushpin** (in the left image) has the same look & feel of the ones that were available in the old Bing Maps and can be used to highlights point of interests in the map; **UserLocationMarker** (in the right image) is used to identify the position of the user and it has the same look & feel of the one used in the native app.
                            
                            [<img title="pushpin" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="pushpin" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/pushpin_thumb.png?resize=222%2C370" width="222" height="370"  data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/pushpin.png)[<img title="user" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px 0px 0px 15px;border-left: 0px;padding-right: 0px" border="0" alt="user" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/user_thumb.png?resize=222%2C370" width="222" height="370"  data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/user.png)
                            
                            ### How to use pushpins
                            
                            First let’s see the simplest way to add a single **Pushpin** or a **UserLocationMarker** to the map. First you have to declare in the XAML the following namespace:
                            
                            _xmlns:toolkit=&#8221;clr-namespace:Microsoft.Phone.Maps.Toolkit;assembly=Microsoft.Phone.Controls.Toolkit&#8221;_
                            
                            Then here is a sample XAML:
                            
                            <pre class="brush: xml;">&lt;maps:Map x:Name="myMap"&gt;
    &lt;toolkit:MapExtensions.Children&gt;
        &lt;toolkit:UserLocationMarker x:Name="UserLocationMarker" /&gt;
        &lt;toolkit:Pushpin x:Name="MyPushpin" Content="My Position"&gt;&lt;/toolkit:Pushpin&gt;
    &lt;/toolkit:MapExtensions.Children&gt;
&lt;/maps:Map&gt;</pre>
                            
                            The first thing is to use the **MapExtensions**, one of the extensions available in the toolkit. This extension has a **Children** property, that is the collection of the elements that are displayed over the map. In this code you can see an example of both controls: **UserLocationMarker** and **Pushpin**. The only difference is that, for the pushpin, I’ve set the **Content** property, which is the label displayed on the mark. 
                            
                            If we want to manage them in the code, in order to set the position in the map, we can use the following code: 
                            
                            <pre class="brush: csharp;">UserLocationMarker marker = (UserLocationMarker)this.FindName("UserLocationMarker");
marker.GeoCoordinate = myMap.Center;

Pushpin pushpin = (Pushpin)this.FindName("MyPushpin");
pushpin.GeoCoordinate = new GeoCoordinate(30.712474, -132.32691);

</pre>
                            
                            First we retrieve the control by using the **FindName** method and passing as parameter the value of the **x:Name** property of the control. Then, once we have a reference, we change the value of the **GeoCoordinate** property. Both controls also have a **Tap** event, that is triggered when user taps on the pushpin. We can use it to interact with the user, like in the following example:
                            
                            <pre class="brush: xml;">&lt;maps:Map x:Name="myMap" Height="400"&gt;
    &lt;toolkit:MapExtensions.Children&gt;
        &lt;toolkit:Pushpin x:Name="MyPushpin" Content="My Position" Tap="MyPushpin_OnTap" /&gt;
    &lt;/toolkit:MapExtensions.Children&gt;
&lt;/maps:Map&gt;
</pre>
                            
                            <pre class="brush: csharp;">private void MyPushpin_OnTap(object sender, GestureEventArgs e)
{
    Pushpin pushpin = sender as Pushpin;
    MessageBox.Show(pushpin.Content.ToString());
}</pre>
                            
                            In the XAML we have declared an event handler to manage the **Tap** event; in the code we get the pushpin that has been tapped (using the **sender** object that is passed as parameter) ****and we show the label on the screen using a MessageBox.
                            
                            ### 
                            
                            ### A useful converter
                            
                            If you’ve already played with the geo localization services available in the new Windows Runtime APIs (the same that are available in Windows 8), you should have noticed a weird thing: the class used by the geo localization APIs to store location information is called **Geolocalization**, while the one used by the map is called **GeoLocalization**. Unlucky, it’s not a typo, they are two different classes: the result is that you can’t take the information returned by the geo localization APIs and assign it to the map, but you need a conversion. Luckily, the Phone Toolkit contains an extension method to do that, you simply have to add the **Microsoft.Phone.Maps.Toolkit** to the code and than you can do something like this:
                            
                            <pre class="brush: csharp;">Geolocator geolocator = new Geolocator();
Geoposition geoposition = await geolocator.GetGeopositionAsync();
myMap.Center = geoposition.Coordinate.ToGeoCoordinate();
</pre>
                            
                            The **ToGeoCoordinate()** extension method takes care of converting the **Coordinate** object returned by the **Geolocator** class (which type is **Geocoordinate**) in the correct one required by the map control.
                            
                            ### 
                            
                            ### In the next post
                            
                            In this post we’ve covered some basic scenarios: they are useful to understand how the phone toolkit can be helpful in developing an application that uses maps, but they are not so useful in a real scenario. For example, a real application would have displayed more than one pushpin in the map, in order to display, for example, some point of interests around the user, like restaurants or pubs. In the next post we’ll see how to do that using some concepts every Windows Phone developer should be familiar with: templates and binding.