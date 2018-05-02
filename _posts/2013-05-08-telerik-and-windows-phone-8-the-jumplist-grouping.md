---
id: 3102
title: 'Telerik and Windows Phone 8 &ndash; The JumpList (Grouping)'
date: 2013-05-08T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=3102
permalink: /telerik-and-windows-phone-8-the-jumplist-grouping/
categories:
  - Windows Phone
tags:
  - MVVM
  - Telerik
  - Windows Phone
---
[Telerik](http://www.telerik.com/) is a very well known company by Microsoft developers, since it develops probably the best components and controls for .NET technologies.� Its relationship with Microsoft developers started a long time ago, since, in their catalogue, you can find controls for “old” technologies like Windows Forms.

It’s a commercial company: their controls suites aren’t free, even if Telerik is a company that has a strong relationships with communities, so it’s not rare to receive free licenses during events and conferences. But, recently, [their Windows Phone suite](http://www.telerik.com/products/windows-phone.aspx) has started to gain a lot of interest among Windows Phone developers for many reasons:

  * The quality of the provided controls is very high: they can make your life easier and they almost fully support binding (even when standard controls doesn’t), so they are a good companion for MVVM apps.
  * Telerik suites, usually, have a high price, because they include all the available controls developed by the company. But, since Windows Phone is gaining a lot of interest also from hobbyst and students, Telerik has decided to sell at 99$ just the Windows Phone controls, without having to buy the full suite: for this reason, the price is affordable also if you’re not a professional.
  * [As I’ve already mentioned in this blog](http://wp.qmatteoq.com/nokia-premium-developer-program/), Nokia has created a new program called [Nokia Premium Developer](https://www.developer.nokia.com/Developer_Programs/Lumia_developer_program.xhtml) which, at the same price of the standard Microsoft Store registration (99 $), it includes a token to create or renew a Store account, a certain number of API calls for the Buddy’s API services and, guess what, a Telerik license!

For these reasons, and due to my recent experience with the suite (I used it in my latest projects), I’ve decided to write some blog posts to show you how to use some of the controls that are part of the toolkit and how they can improve your work.

Le’t start with the JumpList!

### The JumpList control

Lists are one of the key part of every Windows Phone application: it’s almost impossible to find an application that doesn’t need to display a collection of data. Originally Windows Phone 7 included just the **ListBox** control for this scenario, that had many limitations and performance problems. Things are changed now, since Windows Phone 8 includes a control called **LongListSelector** (that, previously, was part of the Phone Toolkit), that has much better performances and that supports features like grouping and sorting.

But things are still not perfect: the **LongListSelector** doesn’t support features that are, instead, widely used by the platform itself, like sticky headers or multiple selection. Plus, the approach to group your data is not very simple: it requires to create a separate class to manage them and to change your collection, like [it’s described in this MSDN article.](http://msdn.microsoft.com/en-us/library/windowsphone/develop/jj244365(v=vs.105).aspx)

For this reason Telerik has included in the RadControls for Windows Phone suite a control called **RadJumpList**, which goal is to overcome the limitations of the original **LongListSelector** control. Let’s see how to implement some scenarios with the **RadJumpList** control.

### 

### Grouping data

Grouping is one of the features that is widely used also by the native operating system: just think at the **People** hub, where contacts are grouped by their initial, so that the user can easily jump from a letter to another. The **RadJumpList** control makes very easy to group your data in the same way and, most important, it doesn’t require you to change your original collection: with the LongListSelector control, in fact, instead of just setting your collection as **ItemsSource** property of the control, you have first to group your data. With **RadJumpList,** instead, you can keep using your original collection, where items are not grouped: grouping will be added in a second step. Let’s start with a simple example: we have a collection of people, that we want to display in the application. Every person is mapped with a specific class:

### 

<pre class="brush: csharp;">public class Person
{
    public string Name { get; set; }
    public string Surname { get; set; }
    public string City { get; set; }
}</pre>

Let’s start by adding the control in the XAML (you’ll need to declare the following namespace _xmlns:telerikDataControls=&#8221;clr-namespace:Telerik.Windows.Controls;assembly=Telerik.Windows.Controls.Data&#8221;_ in the **PhoneApplicationPage** node in the XAML).

<pre class="brush: xml;">&lt;telerikDataControls:RadJumpList x:Name="People"&gt;
    &lt;telerikDataControls:RadJumpList.ItemTemplate&gt;
        &lt;DataTemplate&gt;
            &lt;StackPanel&gt;
                &lt;TextBlock Text="{Binding Path=Name}" /&gt;
                &lt;TextBlock Text="{Binding Path=Surname}" /&gt;
            &lt;/StackPanel&gt;
        &lt;/DataTemplate&gt;
    &lt;/telerikDataControls:RadJumpList.ItemTemplate&gt;
&lt;/telerikDataControls:RadJumpList&gt;</pre>

As you can see, for the basic use there are no differences with a standard **ListBox** or **LongListSelector**: we define an **ItemTemplate**, which is the template that will be used to show every item of the collection. In this sample, we simply display the **Name** and **Surname** properties of the person.

In the code behind, we’re going to load some data as we’ve already used to do with a ListBox:

<pre class="brush: csharp;">public MainPage()
{
    InitializeComponent();

    // Sample code to localize the ApplicationBar
    //BuildLocalizedApplicationBar();

    ObservableCollection&lt;Person&gt; people = new ObservableCollection&lt;Person&gt;
        {
            new Person
                {
                    Name = "Matteo",
                    Surname = "Pagani",
                    City = "Como"
                },
            new Person
                {
                    Name = "Ugo",
                    Surname = "Lattanzi",
                    City = "Milan"
                },
            new Person
                {
                    Name = "Mario",
                    Surname = "Rossi",
                    City = "Milan"
                }
        };

    People.ItemsSource = people;
}</pre>

Nothing special here: if you launch the application, you’ll see the list of people displayed in the **RadJumpList** control.

But if we want grouping? We need to interact with the **GroupDescriptors** property of the **RadJumpList** control, that can be used to set how we want to group our data. Let’s say we want to group our contacts by city. Here is the code we need to add before setting the **ItemsSource** property of the **RadJumpList** control.

<pre class="brush: csharp;">GenericGroupDescriptor&lt;Person, string&gt; group = new GenericGroupDescriptor&lt;Person, string&gt;();
group.KeySelector = (person) =&gt;
    {
        return person.City;
    };

People.GroupDescriptors.Add(group);</pre>

Easy, isn’t it? We just need to define a new **GenericGropDescriptor<T, Y>** object, where **T** is the base class of our collection (in this case, **Person**), while Y is the property’s type we want to use for grouping (in our sample, **City,** that is a string). Then we assign a function to the **KeySelector** property, that is the way we want to group our data. This function features, as parameter, the object’s type of our collection (in our case, **Person**), so we simply need to return, as result, the field we want to use for grouping (**City**). In the end, we add the **GenericGroupDescriptor** object to the **GroupDescriptors** collection of the **RadJumpList** control. And that’s it! Now launch the application and you’ll see your data grouped by city: plus, the group headers will be interactive, so that the user can tap on them and jump directly to the selected group.

[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="jumplist1" alt="jumplist1" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/jumplist1_thumb.png?resize=200%2C333" width="200" height="333" border="0" data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/jumplist1.png)[<img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px 0px 0px 10px; display: inline; padding-right: 0px; border-width: 0px;" title="jumplist2" alt="jumplist2" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/jumplist2_thumb.png?resize=200%2C333" width="200" height="333" border="0" data-recalc-dims="1" />](https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/jumplist2.png)

&nbsp;

### Sorting

Sorting your data is really easy and **RadJumpList** supports two way of sorting: by group and by item. The two things can be combined, so that you can have your groups ordered with a criteria and the items inside that group ordered for another one. Sorting the groups is really easy: the **GenericGroupDescriptor** class we’ve just seen supports a property called **SortMode**, that accepts one of the values of the **ListSortMode** enumerator: **Ascending, Descending** and **None.**

<pre class="brush: csharp;">GenericGroupDescriptor&lt;Person, string&gt; group = new GenericGroupDescriptor&lt;Person, string&gt;();
group.SortMode = ListSortMode.Ascending;
group.KeySelector = (person) =&gt;
    {
        return person.City;
    };

People.GroupDescriptors.Add(group);</pre>

This type of sorting applies just to the order in which groups are displayed in the list and it doesn’t affect the elements inside every group: if you try to change, for example, the **SortMode** to **ListSortMode.Descending**, you’ll see that the group labeled **Milan** will be displayed before the one labeled **Como**, but the order of the contacts inside every group won’t change.

If you want to apply a specific order also for the contacts, you’ll have to use a **GenericSortDescriptor<T, Y>** object, that works exactly like the **GenericGroupDescriptor** one. The only difference is that, in this case, as value of the **KeySelector** property we will pass a function that returns the property we want to use to order the items.

<pre class="brush: csharp;">GenericSortDescriptor&lt;Person, string&gt; sort = new GenericSortDescriptor&lt;Person, string&gt;();
sort.SortMode = ListSortMode.Ascending;
sort.KeySelector = person =&gt;
    {
        return person.Name;
    };

People.SortDescriptors.Add(sort);</pre>

In this sample we want to order the contacts by name: after we’ve set up our **GenericSortDescriptor** object, we add it to another special collection exposed by the **RadJumpList** control, that is called **SortDescriptors**. This sorting is independant from the grouping one: if you try to change the **SortMode** to **ListSortMode.Descending** you’ll notice that the groups order won’t change, but the order of the contacts inside every group will be different.

### Grouping and sorting with MVVM

Using grouping and sorting in an application built with the MVVM pattern it’s easy, since the **RadJumpList** control exposes two properties called **GroupDescriptorsSource** and **SortDescriptorsSource,** that support binding. They accept a collection as input, so you’ll have to create in your ViewModel two collections to hold the **GenericGroupDescriptor** or **GenericSortDescriptor** criterias you want to apply (even if you’re going to add just one criteria).

Here is a sample of a ViewModel:

<pre class="brush: csharp;">public class MainViewModel: INotifyPropertyChanged
{
    private ObservableCollection&lt;GenericGroupDescriptor&lt;Person, string&gt;&gt; groupDescriptors;

    public ObservableCollection&lt;GenericGroupDescriptor&lt;Person, string&gt;&gt; GroupDescriptors
    {
        get { return groupDescriptors; }
        set
        {
            groupDescriptors = value;
            OnPropertyChanged("GroupDescriptors");
        }
    }

    private ObservableCollection&lt;GenericSortDescriptor&lt;Person, string&gt;&gt; sortDescriptors;

    public ObservableCollection&lt;GenericSortDescriptor&lt;Person, string&gt;&gt; SortDescriptors
    {
        get { return sortDescriptors; }
        set
        {
            sortDescriptors = value;
            OnPropertyChanged("SortDescriptors");
        }
    }

    public MainViewModel()
    {
        GroupDescriptors = new ObservableCollection&lt;GenericGroupDescriptor&lt;Person, string&gt;&gt;();
        SortDescriptors = new ObservableCollection&lt;GenericSortDescriptor&lt;Person, string&gt;&gt;();

        GenericGroupDescriptor&lt;Person, string&gt; group = new GenericGroupDescriptor&lt;Person, string&gt;();
        group.SortMode = ListSortMode.Ascending;
        group.KeySelector = (person) =&gt;
            {
                return person.City;
            };

        GroupDescriptors.Add(group);

        GenericSortDescriptor&lt;Person, string&gt; sort = new GenericSortDescriptor&lt;Person, string&gt;();
        sort.SortMode = ListSortMode.Ascending;
        sort.KeySelector = person =&gt;
            {
                return person.Name;
            };

        SortDescriptors.Add(sort);
    }

    public event PropertyChangedEventHandler PropertyChanged;

    [NotifyPropertyChangedInvocator]
    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChangedEventHandler handler = PropertyChanged;
        if (handler != null) handler(this, new PropertyChangedEventArgs(propertyName));
    }
}</pre>

As you can see, we simply use the same code we’ve seen before but, instead of adding the generic descriptors directly to the **RadJumpList** control, we add them to two collections exposed by the ViewModel. Then, in the XAML, we use binding to connect them to the **RadJumpList** control:

<pre class="brush: xml;">&lt;telerikDataControls:RadJumpList x:Name="People" GroupDescriptorsSource="{Binding Path=GroupDescriptors}" SortDescriptorsSource="{Binding Path=SortDescriptors}"&gt;
    &lt;telerikDataControls:RadJumpList.ItemTemplate&gt;
        &lt;DataTemplate&gt;
            &lt;StackPanel&gt;
                &lt;TextBlock Text="{Binding Path=Name}" /&gt;
                &lt;TextBlock Text="{Binding Path=Surname}" /&gt;
            &lt;/StackPanel&gt;
        &lt;/DataTemplate&gt;
    &lt;/telerikDataControls:RadJumpList.ItemTemplate&gt;
&lt;/telerikDataControls:RadJumpList&gt;</pre>

### 

### In the next posts

In the next posts we’ll see some other features of the **RadJumpList** control: how to use sticky headers, multiple selection and how to customize grouping. Meanwhile, you can play with the sample project (obviously, the Telerik libraries are missing, since they can’t be redistributed, you’ll need to manually add your own ones).

<div class="wlWriterEditableSmartContent" id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:664be08d-76ea-4dfa-a611-294081b553a1" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/04/Telerik-JumpList.zip" target="_blank">Download the sample project</a>
  </p>
</div>