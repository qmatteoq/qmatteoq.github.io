---
id: 1322
title: 'Maps in Windows Phone 8 and Phone toolkit: a winning team &ndash; Part 2'
date: 2013-01-21T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=1322
permalink: /maps-in-windows-phone-8-and-phone-toolkit-a-winning-team-part-2/
categories:
  - Windows Phone
tags:
  - Maps
  - Windows Phone
---
<a href="http://wp.qmatteoq.com/maps-in-windows-phone-8-and-phone-toolkit-a-winning-team-part-1/" target="_blank">In the last post</a> we introduced the helpers that are available in the Phone toolkit for the Map control in Windows Phone 8: the map control is much more powerful than the one that is available in Windows Phone 7, but it lacks some controls that were widely used, like pushpins. In the previous post we’ve seen how to add a pushpin and place it in a specific position: it was an interesting experiment, but not really useful, because most of the times we need to place many pushpins on the same layer.

Let’s see how to do it using some features that every Windows Phone developer should know: template and binding.

### Back to work

Let’s imagine that we’re going to display some restaurants that are near the user’s location. For this reason, the first thing we need is a class to map this information:

<pre class="brush: csharp;">public class Restaurant
{
    public GeoCoordinate Coordinate { get; set; }
    public string Address { get; set; }
    public string Name { get; set; }
}
</pre>

After that, we need a way to define a template to display a collection of Restaurant’s object over the map: we’re going to use one of the helpers available in the toolkit.

<pre class="brush: xml;">&lt;maps:Map x:Name="myMap"&gt;
    &lt;toolkit:MapExtensions.Children&gt;
        &lt;toolkit:MapItemsControl Name="RestaurantItems"&gt;
            &lt;toolkit:MapItemsControl.ItemTemplate&gt;
                &lt;DataTemplate&gt;
                    &lt;toolkit:Pushpin GeoCoordinate="{Binding Coordinate}" Content="{Binding Name}" /&gt;
                &lt;/DataTemplate&gt;
            &lt;/toolkit:MapItemsControl.ItemTemplate&gt;
        &lt;/toolkit:MapItemsControl&gt;
    &lt;/toolkit:MapExtensions.Children&gt;
&lt;/maps:Map&gt;
</pre>

We use again the **MapExtensions.Children** that we’ve learned to use in the previous post. Then we introduce the **MapItemsControl**, which is a control that is able to display a collection of objects over the map. Think of it like a **ListBox** control, but in this case the elements that are part of the collection are displayed over the map, as a new layer. The most interesting feature of this control is that, like other controls that are used to display collections, it supports templating: we can define the template (the **ItemTemplate** property) of a single object, that will be used to render every object that is part of the collection. In this sample, we define as a template a **Pushpin** object, which **Content** and **GeoCoordinate** properties are in binding with the properties of the **Restaurant** class. This means that, in the code, we’re going to assign to the **ItemSource** property of the **MapItemsControl** a collection of restaurants: the control is going to render a **Pushpin** for every restaurant in the collection and will place it on the map, in the position defined by the **GeoCoordinate** property.

Here is how we do it:

<pre class="brush: csharp;">public MainPage()
{
    InitializeComponent();
    Loaded+=MainPage_Loaded;
}

void MainPage_Loaded(object sender, RoutedEventArgs e)
{
    ObservableCollection&lt;Restaurant&gt; restaurants = new ObservableCollection&lt;Restaurant&gt;() 
    {
        new Restaurant { Coordinate = new GeoCoordinate(47.6050338745117, -122.334243774414), Address = "Ristorante 1" },
        new Restaurant() { Coordinate = new GeoCoordinate(47.6045697927475, -122.329885661602), Address = "Ristorante 2" },
        new Restaurant() { Coordinate = new GeoCoordinate(47.605712890625, -122.330268859863), Address = "Ristorante 3" },
        new Restaurant() { Coordinate = new GeoCoordinate(47.6015319824219, -122.335113525391), Address = "Ristorante 4" },
        new Restaurant() { Coordinate = new GeoCoordinate(47.6056594848633, -122.334243774414), Address = "Ristorante 5" }
    };

    ObservableCollection&lt;DependencyObject&gt; children = MapExtensions.GetChildren(myMap);
    var obj =
        children.FirstOrDefault(x =&gt; x.GetType() == typeof(MapItemsControl)) as MapItemsControl;

    obj.ItemsSource = restaurants;
    myMap.SetView(new GeoCoordinate(47.6050338745117, -122.334243774414), 16);
}
</pre>

The first thing to highlight is that we’re going to use this code in the **Loaded** event of the page. This is required because, if we do it in the **MainPage** constructor, we risk that the code is executed before that every control in the page is rendered, so we may not have access yet to the **MapItemsControl** object.

First we create some fake restaurants to use for our test and we add it to a collection, which type is **ObservableCollection<Restaurant>.** Then we need to get a reference to the **MapItemsControl** object we’ve declared in the XAML, the one called **RestaurantItems.** Unlucky, due to the particular architecture of the **MapExtensions** helper provided with the toolkit, we can’t access to the control directly using the Name property, but we need to look for it in the collection of controls inside the **Children** property of the **MapExtensions** controls.

To do this, we use the **MapExtensions.GetChildren** method of the toolkit, passing as parameter our map control. This method returns an **ObservableCollection<DependencyObject>**, which is a collection of all the controls that have been added. In this case, by using ****a LINQ query and the **GetType()** method, we get a reference to the **MapItemsControl** object. We can use the **FirstOrDefault** statement because we’ve added just one **MapItemsControl** inside the **MapExtensions.Children** collection: once we have a reference, we cast it to the **MapItemsControl** (otherwise it would be a generic object, so we wouldn’t have access to the specific properties of the **MapItemsControl** class). Finally, we are able to access to the **ItemsSource** property, which holds the collection that is going to be displayed over the map using the **ItemTemplate** we’ve defined in the XAML: we simply assign to this property the fake **ObservableCollection<Restaurant>** we’ve created before.

For sake of simplicity, we also set the center of the map to the coordinates of the first restaurant: this way will be easier for us to test the code and see if every pushpin is displayed correctly.

And here is the final result:

[<img title="map" style="border-left-width: 0px;border-right-width: 0px;border-bottom-width: 0px;padding-top: 0px;padding-left: 0px;padding-right: 0px;border-top-width: 0px" border="0" alt="map" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/map_thumb.png?resize=269%2C448" width="269" height="448"  data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/01/map.png)

### 

### In the end

As you can see, the mechanism is very simple: it’s the usual one that we already use every time we have a collection that we wants to display. We define a collection of objects (usually a **ObservableCollection<T>,** to have INotifyPropertyChanged support out of the box), we define an **ItemTemplate** in the XAML and we assign the collection to the **ItemsSource** property of the control. The only trivial thing, in this case, is to get a reference to the **MapItemsControl** object, since it can’t be referenced just by using the name as for every other control. 

Here is the link to download a sample that recap most of the things we’ve seen in the latest two posts:

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:d90aa57b-cdc8-4019-bae4-473499e47cf5" class="wlWriterEditableSmartContent" style="float: none;padding-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/01/NokiaMaps.zip" target="_blank">Download the sample project</a>
  </p>
</div>