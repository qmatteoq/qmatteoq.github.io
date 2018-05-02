---
id: 3272
title: 'Telerik and Windows Phone 8 &ndash; The JumpList (Multi selection)'
date: 2013-05-14T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=3272
permalink: /telerik-and-windows-phone-8-the-jumplist-multi-selection/
categories:
  - Windows Phone
tags:
  - Telerik
  - Windows Phone
---
We continue our journey to see how to use some of the most interesting features of the **RadJumpList**, one of the available controls in the RadControls for Windows Phone toolkit developed by Telerik.

### Multi selection

Multi selection is a simple but interesting feature that will allow the user to choose more than one element from the list, by checking the checkboxes that are displayed on the left of the item. By default, selection isn’t enabled: the **RadJumpList** control will behave like a standard list; when the user taps on an item, you can intercept it and implement your own logic (for example, you can redirect the user to a detail page to see the details of the selected item.

The first step to enable it is to set the **IsCheckModeEnabled** property to **True** and you can do it in the XAML or in the code. Once it’s enabled, you’ll notice that the left margin of the items will be greater, but the list will continue to work as usual.

You have two ways to effectively enable multi selection mode:

  * You can tap on the left of any item: multi selection will be activated and you’ll see the checkboxes appear. 
      * You can set the **IsCheckModeActive** property of the **RadJumpList** control to **True:** this way, you can also enable the multi select mode in the code if, for example, you want to provide a button for this purpose. The native Mail app shows an example of this behavior: you can enable multi select mode by tapping on the left of any item or by using the appropriate button in the Application Bar. </ul> 
    By default, if you’ve enable grouping like <a href="http://wp.qmatteoq.com/telerik-and-windows-phone-the-jumplist-part-1/" target="_blank">we’ve seen in the previous post</a>, user will be allowed also to select groups and not only single items. If you want to disable this behavior, you need to set the **GroupCheckMode** to **None:** this way, checkboxes will be displayed only next to the single items.
    
    Here is a sample of a **RadJumpList** control with multi selection enabled:
    
    <pre class="brush: xml;">&lt;telerikDataControls:RadJumpList x:Name="People" IsCheckModeEnabled="True" GroupCheckMode="None"&gt;
    &lt;telerikDataControls:RadJumpList.ItemTemplate&gt;
        &lt;DataTemplate&gt;
            &lt;StackPanel&gt;
                &lt;TextBlock Text="{Binding Path=Name}" /&gt;
                &lt;TextBlock Text="{Binding Path=Surname}" /&gt;
            &lt;/StackPanel&gt;
        &lt;/DataTemplate&gt;
    &lt;/telerikDataControls:RadJumpList.ItemTemplate&gt;
&lt;/telerikDataControls:RadJumpList&gt;</pre>
    
    Managing the selected items is really simple: the **RadJumpList** control exposes a property called **CheckedItems**, which contains the collection of items that have been selected. Here is a really sample code that can be used to show the number of items that have been selected by the user.
    
    <pre class="brush: csharp;">private void OnShowItemsClicked(object sender, RoutedEventArgs e)
{
    MessageBox.Show(People.CheckedItems.Count.ToString());
}</pre>
    
    ### Supporting multi selection with MVVM
    
    One issue of the **RadJumpList** control is that, at the moment, **CheckedItems** doesn’t support binding, so you can’t have a property in your ViewModel with the collection of the items that have been selected. The best workaround I’ve found and that I’m using in my Speaker Timer app is to use a custom behavior, that has been developed by my dear friend <a href="http://www.google.it/url?sa=t&rct=j&q=marco%20leoncini%20aspitalia&source=web&cd=1&cad=rja&ved=0CC4QFjAA&url=http%3A%2F%2Fblogs.aspitalia.com%2Fnostromo%2F&ei=SxF9UbXHOYbYswaawYCABQ&usg=AFQjCNGdo3wxIoyRoJf3OaNqr7HxaiSvrQ&bvm=bv.45645796,d.ZWU" target="_blank">Marco Leoncini</a>. The purpose of this behavior is to automatically store, in a property of the ViewModel, the list of selected items.
    
    First, you need to define in your ViewModel a collection, that will hold the items that will be checked by the user. For the following samples, I’m going to use the helpers provided by the **MVVM Light Toolkit** by Laurent Bugnion, that you can install using <a href="http://nuget.org/packages/MvvmLightLibs/" target="_blank">NuGet</a>. Here is a ViewModel sample:
    
    <pre class="brush: csharp;">public class MultiSelectViewModel : ViewModelBase
{
    private ObservableCollection&lt;Person&gt; people;

    public ObservableCollection&lt;Person&gt; People
    {
        get { return people; }
        set
        {
            people = value;
            RaisePropertyChanged("People");
        }
    }

    private ObservableCollection&lt;Person&gt; itemsToDelete;

    public ObservableCollection&lt;Person&gt; ItemsToDelete
    {
        get { return itemsToDelete; }
        set
        {
            itemsToDelete = value;
            RaisePropertyChanged("ItemsToDelete");
        }
    }


    public MultiSelectViewModel()
    {
        ItemsToDelete = new ObservableCollection&lt;Person&gt;();
    }
}</pre>
    
    In this sample, **ItemsToDelete** is the property that will store the items selected by the user (this sample is taken from my Speaker Timer app, where multi selection is enabled to delete one or more sessions). The second step is to create a new class, that will contain our behavior:
    
    <pre class="brush: csharp;">public class AddoToCheckedItemBehavior : Behavior&lt;RadDataBoundListBox&gt;
{
    public AddoToCheckedItemBehavior() { }

    protected override void OnAttached()
    {
        base.OnAttached();
        this.AssociatedObject.ItemCheckedStateChanged += AssociatedObject_ItemCheckedStateChanged;
    }

    void AssociatedObject_ItemCheckedStateChanged(object sender, ItemCheckedStateChangedEventArgs e)
    {
        var viewModel = ((FrameworkElement)sender).DataContext as MainPageViewModel;
        if (viewModel == null) return;
        if (e.IsChecked)
        {
            viewModel.ItemsToDelete.Add(((Telerik.Windows.Data.IDataSourceItem)e.Item).Value as Session);
        }
        else { viewModel.ItemsToDelete.Remove(((Telerik.Windows.Data.IDataSourceItem)e.Item).Value as Session); }
    }

    protected override void OnDetaching()
    {
        base.OnDetaching();
        this.AssociatedObject.ItemCheckedStateChanged -= AssociatedObject_ItemCheckedStateChanged;
    }
}</pre>
    
    The purpose of this behavior is very simple: it inherits from the **Behavior<RadJumpList>** class (so that it can be used in combination with a **RadJumpList** control) and it subscribes to the **ItemCheckedStateChanged** event, that is triggered every time the user checks or unchecks an item from the list. When this event is triggered, we get a reference to the current ViewModel and we add or remove the selected item from the **ItemsToDelete** collection. To do this, we use a property returned by the event handler’s parameter: the **e** object (which type is **ItemCheckedStateChangedEventArgs**) contains the properties **Item** (which is the selected item) and **IsChecked** (which is a boolean, that tells you if the item has been checked or unchecked).
    
    There’s space for improvements in this behavior: one limitation is that it works just for our specific ViewModel, since when we get the reference to the ViewModel associated to the current view (using the **DataContext** property) we cast it to the **MainViewModel** class. If we need to apply the same behavior to multiple **RadJumpList** controls in different page, we can create a base class which our ViewModels can inherit from. This base class will host the the **ItemsToDelete** property, so that every ViewModel will be able to use it. This way, we can change the cast from **MainViewModel** to our new base class and reuse the same behavior for different ViewModels.
    
    How to apply the behavior? First, we need to add a reference to the **System.Windows.Interactivity** library, that is available by clicking with the right click on your project and choosing **Add new reference:** you’ll find it in the **Assemblies, Extensions** sections.
    
    After that, you can add the behavior directly in the XAML, by declaring the following namespace (plus the namespace of the behavior’s class you’ve created):
    
    _xmlns:i=&#8221;clr-namespace:System.Windows.Interactivity;assembly=System.Windows.Interactivity&#8221;_
    
    <pre class="brush: xml;">&lt;telerikDataControls:RadJumpList IsCheckModeEnabled="True" GroupCheckMode="None" ItemsSource="{Binding Path=People}"&gt;
    &lt;telerikDataControls:RadJumpList.ItemTemplate&gt;
        &lt;DataTemplate&gt;
            &lt;StackPanel&gt;
                &lt;TextBlock Text="{Binding Path=Name}" /&gt;
                &lt;TextBlock Text="{Binding Path=Surname}" /&gt;
            &lt;/StackPanel&gt;
        &lt;/DataTemplate&gt;
    &lt;/telerikDataControls:RadJumpList.ItemTemplate&gt;
    &lt;i:Interaction.Behaviors&gt;
        &lt;behaviors:AddoToCheckedItemBehavior /&gt;
    &lt;/i:Interaction.Behaviors&gt;
&lt;/telerikDataControls:RadJumpList&gt;

</pre>
    
    Now in the ViewModel, automatically, the **ItemsToDelete** property will always contain the items that have been checked by the user, so that you can use them for your own needs. In the following sample, we define a command to simply show a message with the number of selected items:
    
    <pre class="brush: csharp;">private RelayCommand showCount;

 public RelayCommand ShowCount
 {
     get
     {
         if (showCount == null)
         {
             showCount = new RelayCommand(() =&gt;
                                              {
                                                  MessageBox.Show(ItemsToDelete.Count.ToString());
                                              });
         }
         return showCount;
     }
 }

</pre>
    
    Happy coding! As usual, remember that the sample project doesn’t contain the Telerik libraries, since we’re talking about a commercial suite.
    
    <div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:8ede1e39-9fe4-42ad-8795-cf44e0d86eac" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
      <p>
        <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/04/JumpList_MultiSelect.zip" target="_blank">Download the sample project</a>
      </p>
    </div>