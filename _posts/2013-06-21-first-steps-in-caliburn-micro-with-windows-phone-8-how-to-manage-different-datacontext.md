---
id: 3732
title: 'First steps in Caliburn Micro with Windows Phone 8 &ndash; How to manage different DataContext'
date: 2013-06-21T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=3732
permalink: /first-steps-in-caliburn-micro-with-windows-phone-8-how-to-manage-different-datacontext/
categories:
  - Windows Phone
tags:
  - Caliburn
  - MVVM
  - Windows Phone
---
After a while I’m back to publish a post about Caliburn Micro and Windows Phone. The reason is that, during the development of my <a href="http://www.windowsphone.com/s?appid=a476ab62-44b4-4cca-9763-e7e064a1cc26" target="_blank">Movies Tracker</a> app, I’ve found my self in a corner case I haven’t managed before: a **ContextMenu**. We’re talking about a control available in the <a href="http://phone.codeplex.com/" target="_blank">Phone Toolkit</a>, that can be used to add a contextual menu to a control. Windows Phone itself uses it widely: for example, when you press and hold your finger on an icon in the application list; or when you’re the People hub and you want to do some operations on a contact.

Usually **ContextMenu** is used in combination with a control that is used to display a collection, like a **ListBox** or **LongListSelector**: the user sees a list of items and can tap and hold on one of them to do some additional actions.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="MoviesTracker1" alt="MoviesTracker1" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/06/MoviesTracker1_thumb.png?resize=218%2C363" width="218" height="363" border="0" data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/06/MoviesTracker1.png)

Which is the problem in a MVVM application? That, in this case, we’re working with two different DataContext at the same time. Let’s see why:

  * The **ListBox** is placed inside a view, that will be connected to a specific ViewModel, that will take care of populating the collections, get the item selected by the user and stuff like that;
  * The **ContextMenu** is placed inside the **ItemTemplate** of the **ListBox,** since it’s applied to every single item of the list. This means that the DataContext of the **ContextMenu** control is different: inside an **ItemTemplate** the **DataContext** is represented by the object that is inside the collection (if you have a **List<Person>,** for example, the **DataContext** of the single item will be the **Person** class).

This means that, probably, you’ll define the actions to execute when the user taps on an item of the **ContextMenu** in the ViewModel of the page but the control won’t be able to see them, since it will expect to find them inside the **Person** object. And this is not good: defining methods inside a class that represents an entity is not a good idea; plus, probably the actions we’re going to execute (for example, deleting the selected item), will require us to interact with other properties of the ViewModel (like the collection), that can’t be accessed from the **Person** class.

Let’s see how to manage this scenario using Caliburn Micro.

### Let’s start!

We’re going to use a simple scenario: an application to display a list of persons using a **ListBox**. By using a **ContextMenu**, we’ll give to the user the ability to delete a specific item. Let’s start with a standard Caliburn project: if you don’t know how to setup it, please refer to <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-complete-series/" target="_blank">the other posts of the series</a>. Our goal is to have one View with its specific ViewModel, connected together with <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-first-project/" target="_blank">the naming convention</a> every Caliburn developer should be already familiar with.

First, we’re going to add to our project, using <a href="http://nuget.org/packages/WPtoolkit/" target="_blank">NuGet</a>, the Phone Toolkit: simply right click on your project, choose **Manage NuGet packages,** search for and install the package called “phone toolkit”. Then, we add a new **Person** class, that we’re going to use to store the information about the persons. Since it’s a sample, it’s really simple:

<pre class="brush: csharp;">public class Person
{
    public string Name { get; set; }
    public string Surname { get; set; }
}</pre>

Now we can define the View, inside the standard page **MainPage.xaml**: we’re going to include the **ListBox** control, with a simple **ItemTemplate.** We’ll simply display, in fact, name and surname of the person. Plus, we’re going to add the **ContextMenu** control to manage the contextual options:

<pre class="brush: xml;">&lt;StackPanel&gt;
    &lt;ListBox x:Name="Persons"&gt;
        &lt;ListBox.ItemTemplate&gt;
            &lt;DataTemplate&gt;
                &lt;StackPanel&gt;
                    &lt;toolkit:ContextMenuService.ContextMenu&gt;
                        &lt;toolkit:ContextMenu&gt;
                            &lt;toolkit:MenuItem Header="delete" /&gt;                                                        
                        &lt;/toolkit:ContextMenu&gt;
                    &lt;/toolkit:ContextMenuService.ContextMenu&gt;
                    &lt;TextBlock Text="{Binding Path=Name}" /&gt;
                    &lt;TextBlock Text="{Binding Path=Surname}" /&gt;
                &lt;/StackPanel&gt;
            &lt;/DataTemplate&gt;
        &lt;/ListBox.ItemTemplate&gt;
    &lt;/ListBox&gt;
&lt;/StackPanel&gt;</pre>

To use the **ContextMenu** you’ll need to add the toolkit’s namespace in your page declarations in the XAML:

_xmlns:toolkit=&#8221;clr-namespace:Microsoft.Phone.Controls;assembly=Microsoft.Phone.Controls.Toolkit&#8221;_

First, we’re going to use the <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">collections naming convention</a>: since the name of the **ListBox** is **Persons**, we expect to have a collection with the same name in the ViewModel. This way, the data of the collection will be automatically displayed in the **ListBox**. The second thing to notice is that we’ve added the **ContextMenu** inside the **StackPanel**: this way the user will be able to tap and hold inside the item to display the menu. We’ve added just one action, using the **MenuItem** control: the deletion of the current item.

What we need to do when the user chooses the delete option?

  * We need to identify on which item the context menu has been activated;
  * We need to delete the item from the collection;

And here comes the problem I’ve described in the beginning of the post: we need to manage these operations in the ViewModel of the page (since it’s the one that contains the collection) but, since we’re inside the **ItemTemplate**, the application will look for them inside the **Person** object. Let’s introduce some Caliburn magic to solve our problem:

<pre class="brush: xml;">&lt;phone:PhoneApplicationPage
    x:Class="Caliburn.ContextMenu.Views.MainPageView"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:phone="clr-namespace:Microsoft.Phone.Controls;assembly=Microsoft.Phone"
    xmlns:shell="clr-namespace:Microsoft.Phone.Shell;assembly=Microsoft.Phone"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:toolkit="clr-namespace:Microsoft.Phone.Controls;assembly=Microsoft.Phone.Controls.Toolkit"
    xmlns:micro="clr-namespace:Caliburn.Micro;assembly=Caliburn.Micro"
    mc:Ignorable="d"
    FontFamily="{StaticResource PhoneFontFamilyNormal}"
    FontSize="{StaticResource PhoneFontSizeNormal}"
    Foreground="{StaticResource PhoneForegroundBrush}"
    SupportedOrientations="Portrait" Orientation="Portrait"
    shell:SystemTray.IsVisible="True"
    x:Name="Page"&gt;

    &lt;!--LayoutRoot is the root grid where all page content is placed--&gt;
    &lt;Grid x:Name="LayoutRoot" Background="Transparent"&gt;
        &lt;Grid.RowDefinitions&gt;
            &lt;RowDefinition Height="Auto"/&gt;
            &lt;RowDefinition Height="*"/&gt;
        &lt;/Grid.RowDefinitions&gt;

        &lt;!--TitlePanel contains the name of the application and page title--&gt;
        &lt;StackPanel x:Name="TitlePanel" Grid.Row="0" Margin="12,17,0,28"&gt;
            &lt;TextBlock Text="MY APPLICATION" Style="{StaticResource PhoneTextNormalStyle}" Margin="12,0"/&gt;
            &lt;TextBlock Text="page name" Margin="9,-7,0,0" Style="{StaticResource PhoneTextTitle1Style}"/&gt;
        &lt;/StackPanel&gt;

        &lt;!--ContentPanel - place additional content here--&gt;
        &lt;Grid x:Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0"&gt;
            &lt;StackPanel&gt;
                &lt;ListBox x:Name="Persons"&gt;
                    &lt;ListBox.ItemTemplate&gt;
                        &lt;DataTemplate&gt;
                            &lt;StackPanel&gt;
                                &lt;toolkit:ContextMenuService.ContextMenu&gt;
                                    &lt;toolkit:ContextMenu&gt;
                                        &lt;toolkit:MenuItem Header="delete" 
                                                          micro:Action.TargetWithoutContext="{Binding ElementName=Page, Path=DataContext}"
                                                          micro:Message.Attach="[Event Tap] = [Action Delete($datacontext)]" /&gt;
                                    &lt;/toolkit:ContextMenu&gt;
                                &lt;/toolkit:ContextMenuService.ContextMenu&gt;
                                &lt;TextBlock Text="{Binding Path=Name}" /&gt;
                                &lt;TextBlock Text="{Binding Path=Surname}" /&gt;
                            &lt;/StackPanel&gt;
                        &lt;/DataTemplate&gt;
                    &lt;/ListBox.ItemTemplate&gt;
                &lt;/ListBox&gt;
            &lt;/StackPanel&gt;
        &lt;/Grid&gt;
    &lt;/Grid&gt;
&lt;/phone:PhoneApplicationPage&gt;</pre>

I’ve included, this time, the whole XAML of the page for just one reason: please notice that I’ve assigned a name (using the **x:Name** property) to the page. Thanks to this name, I’m able to use a special attached property provided by Caliburn, called **Action.TargetWithoutContext.** It’s purpose is to change the **DataContext** of the action that we’re going to assign to the interaction: instead of using the default one (in this case, it would be the **Person** object) we can override it and specify a new one.� In our sample, we’re going to set the same DataContext of the page, that is our ViewModel: this is why we need to give to the entire page a name, so that we can refer to it to access to the **DataContext** property.

<p align="left">
  Now that the item is connected to the correct DataContext, we can define the action in the same way <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-actions/" target="_blank">we’ve learned before</a>: by using the <strong>Message.Attach</strong> attached property and setting first the event we want to manage (<strong>Tap</strong>) and then the method defined in the ViewModel to execute (<strong>Delete</strong>). You can notice something new: that we’re passing a parameter called <strong>$datacontext</strong> to the method. This way, we’re going to pass to the method the current <strong>DataContext, </strong>that is the selected <strong>Person</strong> object. This wasn’t needed when we simply had to manage the selected item in the list, thanks to the <strong>SelectedItem</strong> property of the <strong>ListBox</strong> control. This time, we can’t rely on it because, when we interact with another control in the <strong>ItemTemplate</strong> (like our <strong>ContextMenu</strong> or a <strong>Button</strong>) the <strong>SelectedItem</strong> doesn’t get a proper value, so we wouldn’t have a way to know which is the item the user is interacting with.
</p>

<p align="left">
  Thanks to this syntax, instead, we can pass the selected <strong>Person</strong> object to the method and work with it. To use both attached properties, you’ll need to declared the following namespace in your XAML’s page:
</p>

<p align="left">
  <em>xmlns:micro=&#8221;clr-namespace:Caliburn.Micro;assembly=Caliburn.Micro&#8221;</em>
</p>

<p align="left">
  And here is how it looks like our ViewModel:
</p>

<pre class="brush: csharp;">public class MainPageViewModel : Screen
{
    private ObservableCollection&lt;Person&gt; persons;

    public ObservableCollection&lt;Person&gt; Persons
    {
        get { return persons; }
        set
        {
            persons = value;
            NotifyOfPropertyChange(() =&gt; Persons);
        }
    }

    public MainPageViewModel()
    {
        Persons = new ObservableCollection&lt;Person&gt;
            {
                new Person
                    {
                        Name = "Matteo",
                        Surname = "Pagani"
                    },
                new Person
                    {
                        Name = "John",
                        Surname = "Doe"
                    },
                new Person
                    {
                        Name = "Mark",
                        Surname = "White"
                    }
            };
    }

    public void Delete(Person person)
    {
        Persons.Remove(person);
    }

}</pre>

As you can see, the only difference with a standard action is that the **Delete** method accepts a parameter in input, which is the selected **Person** object: this way, we can simply remove it from the collection by calling the **Remove** method.

This approach isn’t useful just when you use a **ContextMenu** control, but every time you need to add another type of interaction with a **ListBox** rather than just selecting the item. I have another example of that in my <a href="http://www.windowsphone.com/s?appid=a476ab62-44b4-4cca-9763-e7e064a1cc26" target="_blank">Movies Tracker</a> app: in the page used to search for a movie to add to your collection, I’ve included a button that works as “quick add”. The user can interact with a movie in the list in two ways: he can simply tap on one of them and being redirected to the detail page (and this is a standard interaction, made using the **SelectedItem** property of the list); or he can tap on the quick add button: in this case, I show a popup, where the user can set some options before saving it.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="MoviesTracker2" alt="MoviesTracker2" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/06/MoviesTracker2_thumb.png?resize=233%2C388" width="233" height="388" border="0" data-recalc-dims="1" />](https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/06/MoviesTracker2.png)

This second scenario has the same problems we’ve seen with **ContextMenu** control: when the user taps on the button, I need to know which is the selected movie; and when the saving action is triggered, I need to execute a method declared in the ViewModel of the page, not in the **Movie** class, since it’s missing all the properties and classes I need to perform the operation. Same scenario, same solution: here is how it looks like the **ItemTemplate** of my movie list.

<pre class="brush: xml;">&lt;DataTemplate x:Key="OnlineMoviesTemplate"&gt;
    &lt;Grid Margin="0,0,0,12"&gt;
        &lt;Grid.ColumnDefinitions&gt;
            &lt;ColumnDefinition Width="Auto"/&gt;
            &lt;ColumnDefinition /&gt;
            &lt;ColumnDefinition /&gt;
        &lt;/Grid.ColumnDefinitions&gt;
        &lt;Image Width="99" Height="99" &gt;
            &lt;Image.Source&gt;
                &lt;BitmapImage UriSource="{Binding PosterThumbnailUrl}" CreateOptions="BackgroundCreation, DelayCreation" /&gt;
            &lt;/Image.Source&gt;
        &lt;/Image&gt;
        &lt;StackPanel Margin="0,0,0,0" Grid.Column="1" VerticalAlignment="Top"&gt;
            &lt;TextBlock FontSize="20"
                       FontWeight="ExtraBold"
                       Text="{Binding Title, Converter={StaticResource UppercaseConverter}}" TextWrapping="Wrap" TextAlignment="Left" /&gt;
        &lt;/StackPanel&gt;
        &lt;telerikPrimitives:RadImageButton RestStateImageSource="/Assets/Icons/addmovie.png" Grid.Column="2" HorizontalAlignment="Right"
                                          micro:Action.TargetWithoutContext="{Binding ElementName=Page, Path=DataContext}" micro:Message.Attach="[Event Click] = [Action AddMovie($datacontext)]" 
                                          /&gt;
    &lt;/Grid&gt;
&lt;/DataTemplate&gt;</pre>

As you can see, the code is exactly the same with seen for the **ContextMenu**: I’ve defined the **Action.TargetWithoutContext** and **Message.Attach** attached properties on the **RadImageButton** control so that, when the button is pressed, I execute the **AddMovie** method on the ViewModel of the page that contains the list. Thanks to the **$datacontext** syntax, I’m able to get in my method the movie the user has interacted with, so that I know which one to save. In this case, I’m using a **RadImageButton** (that is a Telerik control that is simply a button that, instead of being rendered using text, it uses an image) but it would have worked with any other control the user can interact with: a **Button**, a **Checkbox**, a **RadioButton**, etc.

### In conclusion

The scenario described in this post isn’t so common: personally, I don’t suggest to abuse of the **ContextMenu** control, because it’s hard to discover for the user. But in case it happens (or you have to use another control, like happened to me with the button), these Caliburn Micro helpers will help you to save a lot of time and efforts.

<div class="wlWriterEditableSmartContent" id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:44b2dd91-669e-40b7-b57a-c9aa25271c62" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/06/Caliburn.ContextMenu.zip" target="_blank">Download the sample project</a>
  </p>
</div>