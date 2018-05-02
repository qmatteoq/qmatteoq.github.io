---
id: 6361
title: 'Xamarin Forms for Windows Phone devs &ndash; Navigation and application lifecycle'
date: 2015-01-16T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=6361
permalink: /xamarin-forms-for-windows-store-app-devs-navigation-and-application-lifecycle/
categories:
  - wpdev
  - Xamarin
tags:
  - Windows Phone
  - Xamarin
---
One of the core concepts when you create a mobile application is navigation: it’s unlikely that an app is made by just one page, so you need to provide a way to the user to move from one page to the other. We’ve already seen, <a href="http://wp.qmatteoq.com/xamarin-forms-for-windows-store-app-devs-the-first-project/" target="_blank">in the previous post</a>, that the base navigation infrastructure for Xamarin Forms is provided by the **NavigationPage** class: by encapsulating a basic **ContentPage** page into a **NavigationPage**, we’ll be able to navigate from one page to another using the **Navigation** property, regardless of the platform where the application is running.

### 

### 

### Navigate to another page

We’ve already seen that, to proper support navigation, we need to encapsulate our page into a **NavigationPage**. From a Windows Phone developer point of view, this operation is completely transparent to the user: a standard **ContentPage** and a **ContentPage** encapsulated into a **NavigationPage** will look exactly the same, since Windows Phone doesn’t provide a visual way to navigate through the different pages of the applications. The navigation is managed by the Back key, which is a physical button. In Android and iOS, instead, we can notice immediately if we’re using a **NavigationPage** or not: in the first case, in fact, the application will display an header with the name of the page and an arrow to go back to the previous page, if existing. The following image shows the same **ContentPage** on Android: the first one is directly displayed, the second one, instead, is embedded into a **NavigationPage** first. This is an important difference to understand if you’re coming to the Windows Phone world.

[<img title="navigation" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="navigation" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/navigation_thumb1.png?resize=500%2C379" width="500" height="379"  data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/navigation1.png)

&nbsp;

In the previous post we’ve seen the basic way to navigate from one page to the other: by using the **PushAsync()** method of the **Navigation** property, which requires as parameter the new page where we want to redirect the user.

<pre class="brush: csharp;">private async void OnNavigateClicked(object sender, EventArgs e)
{
    await Navigation.PushAsync(new DetailView());
}
</pre>

This code is the equivalent of calling the **Navigate()** method of the **Frame** class in a Windows Store app or the **NavigationService** class in a Silverlight app.

This sample assumes that our project has a page called **DetailView**: by passing a new instance to the **PushAsync()** method, we’ll redirect the user to this page. Again, in Windows Phone you’ll just be redirected to a new page and, to go back, you’ll just have to use the Back button; on Android and iOS, instead, the header will change and it will display, other than the page title, a virtual back button to redirect the user to the previous page. The following image will help you to understand the difference:

[<img title="detail" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="detail" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/detail_thumb1.png?resize=500%2C378" width="500" height="378"  data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/detail1.png)

In case you want to provide in your page an additional button to redirect the user to previous page, you can use the **PopAsync()** method of the **Navigation** class, like in the following sample:

<pre class="brush: csharp;">private async void OnNavigateBackClicked(object sender, EventArgs e)
{
    await Navigation.PopAsync();
}

</pre>

### 

The previous code is the equivalent of the **GoBack()** method exposed by the **Frame** class in a Windows Store app or the **NavigationService** class in a Silverlight app.

### Displaying a modal view

Android and iOS has the concept of “modal view”, which is a full screen page that is displayed on top of navigation stack. The difference with a standard navigation is that the navigation bar will be hidden and the page will use the whole screen: the only way to go back to the previous page is by using the Back button (in Android) or by adding a dedicated button in the page, that will invoke a specific method offered by the **Navigation** class. To display a page in modal view, you’ll have to use the **PushModalAsync()** method, like in the following sample:

<pre class="brush: csharp;">private async void OnModalViewClicked(object sender, EventArgs e)
{
    await Navigation.PushModalAsync(new DetailView());
}
</pre>

The following image shows the difference between a navigation to the **DetailView** page using the **PushAsync()** method (on the left) and the **PushModalAsync()** method (on the right): as you can see, the right page doesn’t have the navigation bar at the top.

[<img title="modal" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="modal" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/modal_thumb1.png?resize=500%2C381" width="500" height="381"  data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/modal1.png)

On Windows Phone, the **PushModalAsync()** method and the **PushAsync()** one has the same effect: since Windows Phone doesn’t have the concept of modal view, the **PushModalAsync()** method will simply redirect the user to the new page, allowing him to go back to the previous one simply by pressing the Back button. If you want to manually dismiss a modal view and redirect the user to the previous page, you can use the **PopModalAsync()** method:

<pre class="brush: csharp;">private async void OnBackClicked(object sender, EventArgs e)
{
    await Navigation.PopModalAsync();
}
</pre>

### 

### Combining different page’s types

In the previous post we’ve seen that Xamarin Forms offers different types of pages, like **TabbedPage** or **CarouselPage**, which were set directly as starting page of our application by assigning them to the **MainPage** property of the **App** class. However, we can also combine different page types, to create more complex navigation patterns. For example, we could have a **NavigationPage** that redirects the user to a **TabbedPage**, like in the following example:

<pre class="brush: csharp;">private async void OnTabbedPageClicked(object sender, EventArgs e)
{
    TabbedPage tabbedPage = new TabbedPage();
    tabbedPage.Children.Add(new DetailView());
    tabbedPage.Children.Add(new AnotherView());
    await Navigation.PushModalAsync(tabbedPage);
}
</pre>

With this code, the user will be redirected from the current page to another page that will display two different sections: one connected to the **DetailView** page, the other one to the **AnotherView** page. Vice versa, we could have a **TabbedPage** that contains, as children, a **NavigationPage**, so that we can perform additional navigations from that page:

<pre class="brush: csharp;">public class App : Application
{
    public App()
    {
        TabbedPage tabbedPage = new TabbedPage();
        NavigationPage navigationPage = new NavigationPage(new MainView());
        tabbedPage.Children.Add(navigationPage);
        tabbedPage.Children.Add(new AnotherView());

        this.MainPage = tabbedPage;
    }
}
</pre>

This way, the first section of the tab control (which is a **MainView** page) will allow the user to navigate to another page of the application (like a detail page). The following image shows a **NavigationPage** displayed inside a tab:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/image_thumb2.png?resize=362%2C603" width="362" height="603"  data-recalc-dims="1" />](https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/image2.png)

Again, you won’t notice instead any difference in the Windows Phone application: you will just see a **Pivot** control with two items, one for the **MainView** and one for the **AnotherView** pages.

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/image_thumb3.png?resize=362%2C648" width="362" height="648"  data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2015/01/image3.png)

The only thing to keep in mind, in this scenario, is that the **TabbedPage** takes, as label for the tab, the title of the page: in this case, since we’re not directly setting as tab content the **ContentPage**, but we have wrapped it into a **NavigationPage,** we have also to set its **Title** property, otherwise the tab label will be empty. This is the proper code:

<pre class="brush: csharp;">public class App : Application
{
    public App()
    {
        TabbedPage tabbedPage = new TabbedPage();
        NavigationPage navigationPage = new NavigationPage(new MainView()) { Title = "Main page"};
        tabbedPage.Children.Add(navigationPage);
        tabbedPage.Children.Add(new AnotherView());

        this.MainPage = tabbedPage;
    }
}
</pre>

### Page navigation

One common requirement in a mobile application is to detect is moving towards the current page or he’s navigating away to go to another page. This approach is very well known to Windows Phone developers: Windows Store apps, in fact, offer the **OnNavigatedTo()** and **OnNavigatedFrom()** events, which are often used to load data or to unsubscribe to event handlers. The **OnNavigatedTo()** event is especially useful to load the data to display in the current page: since it’s an event handler, we can use it also to invoke asynchronous method with the async / await pattern (which, instead, is not possible in the page’s constructor).

These two events are mapped in Xamarin Forms with the **OnAppearing()** method (that is invoked when the page is being displayed) and the **OnDisappearing**() method (that is invoked when the user is moving away from the current page):

<pre class="brush: csharp;">public partial class MainView
{
    public MainView()
    {
        InitializeComponent();
    }

    protected override void OnAppearing()
    {
        Debug.WriteLine("Page is appearing");
    }

    protected override void OnDisappearing()
    {
        Debug.WriteLine("Page is disappearing");
    }
}
</pre>

### The application lifecycle

If you’re a Windows Phone developer, you should be familiar with the concept of application’s lifecycle. To satisfy the typical requirement of a mobile environment (performance, battery consumption, etc.) all the mobile platform applies constraints to the operations that an application can perform in background. On all the platforms, typically, when an application isn’t in foreground anymore is suspended and it doesn’t have the chance to perform operation: this scenario is managed using other approaches, like background tasks in Windows Phone.

Xamarin Forms has added three methods in the shared **App** class to properly manage the application’s lifecycle, in case you need to perform additional operations when the app is launched, suspended or resumed. The following sample shows how to declare and use these events:

<pre class="brush: csharp;">public class App : Application
{

   protected override void OnStart()
   {
       Debug.WriteLine("Application started");
   }

   protected override void OnSleep()
   {
       Debug.WriteLine("Application suspended");
   }

   protected override void OnResume()
   {
       Debug.WriteLine("Application resumed");
   }
}
</pre>

If you are a Windows Phone developer, these events should be familiar to you:

  * **OnStart()** is called when the application is launched from scratch. 
      * **OnSleep()** is called when the application is suspended. 
          * **OnResume()** is called when the application is resumed. It’s important to highlight that this method is invoked only when a previous instance of the app was kept in memory and it’s restored: for example, on Windows Phone, in case of tombstoning (so the process was terminated by the operating system due to low resources) you’ll trigger the **OnStart()** method at startup and not the **OnResume()** one. </ul> 
        ### Wrapping
        
        In this posts we’ve seen the basic concept of navigation and application’s lifecycle management and how they compare to the Windows Phone approach. In the next posts, we’ll start to see how to create a real application using the MVVM pattern. As usual, you can find the samples used in this post on GitHub: [https://github.com/qmatteoq/XamarinFormsSamples](https://github.com/qmatteoq/XamarinFormsSamples "https://github.com/qmatteoq/XamarinFormsSamples")</font>
        
        </font></font>