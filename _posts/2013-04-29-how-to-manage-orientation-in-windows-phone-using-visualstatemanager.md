---
id: 3022
title: How to manage orientation in Windows Phone using VisualStateManager
date: 2013-04-29T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=3022
permalink: /how-to-manage-orientation-in-windows-phone-using-visualstatemanager/
categories:
  - Windows Phone
tags:
  - Windows Phone
---
When I started to play with Windows Store apps development for Windows 8, one of the things I liked most was the base **LayoutAwarePage** class, from which every page inherits from. This class introduces a built in support for managing the different visual states of the page using the **VisualStateManager**: every time the page state changes (Portrait, Landscape, Snapped or Filled), a specific state is triggered so, as a developer, it’s really easy to apply animation and changes the layout of the page.

Unfortunately, this built in helper is missing in Windows Phone, so you have to manually manage the orientation from portrait to landscape or vice versa: Windows Phone is able to, automatically, arrange the controls when the orientation is changed, but often isn’t enough because you want to deeply change the layout of the page according to the orientation.

So I googled a little bit and I’ve found that a talented developer that has a blog called [Tomahawk’s blog](http://atomaras.wordpress.com/) (I wasn’t able to find the real developer name, if you contact me I’ll be more than happy to update the post) has developed a really nice extension, that can be used to recreate the same approach I’ve described in Windows 8 in both Windows Phone 7 and 8 applications. 

Let’s see how to use it. Unfortunately, the library isn’t available on NuGet, so you’ll have to download the source project and compile it: you’ll get, in the end, different DLLs, one for each supported platform. For your convenience, in the sample project attached at the end of this post you’ll find the already precompiled libraries.

After you’re created your project, the first modify to do to start supporting orientation is to change the **SupportedOrientations** property in the XAML: you’ll find it as a property of the **PhoneApplicationPage** object in XAML, which is the main control that identifies the page. This property tells to the page which orientations are you going to support and, by default, it’s set to **Portrait**, so the page doesn’t react when the device is rotated in landscape. To enable it, you’ll have to change the value in **PortraitOrLandscape.**

The second thing is to register, always in the **PhoneApplicationPage** control, the library**:** you’ll have to register the namespace of the library (which, for a Windows Phone 8 application is _xmlns:orientationHelper=&#8221;clr-namespace:OrientationHelper;assembly=OrientationHelper.PhoneRT&#8221;_ while for a Windows Phone 7 one is&nbsp; _xmlns:orientationHelper=&#8221;clr-namespace:OrientationHelper;assembly=OrientationHelper.WP7&#8243;_) and set the property **OrientationHelper.IsActive** to **true**.

Here is how it will look like, in the end, the **PhoneApplicationHelper** declaration of your page:

<pre class="brush: xml;">&lt;phone:PhoneApplicationPage
    x:Class="Orientation.MainPage"
    SupportedOrientations="PortraitOrLandscape" Orientation="Portrait"
    orientationHelper:OrientationHelper.IsActive="True"
    &gt;</pre>

Once you’ve completed this steps, automatically the application will start to apply different visual states every time the orientation changes. So, you’ll need to define them in your page, inside the main Grid of the application (the one that is called **LayoutRoot** in the standard template).

Here is a sample:

<pre class="brush: xml;">&lt;VisualStateManager.VisualStateGroups&gt;
    &lt;VisualStateGroup x:Name="PageOrientationStates"&gt;
        &lt;VisualState x:Name="Landscape" /&gt;
        &lt;VisualState x:Name="LandscapeLeft" /&gt;
        &lt;VisualState x:Name="LandscapeRight" /&gt;

        &lt;VisualState x:Name="Portrait" /&gt;
        &lt;VisualState x:Name="PortraitUp" /&gt;
        &lt;VisualState x:Name="PortraitDown" /&gt;
    &lt;/VisualStateGroup&gt;
&lt;/VisualStateManager.VisualStateGroups&gt;</pre>

As you can see, we’ve added a **VisualStateGroup** which name is **PageOrientationStates** and, inside it, there’s a specific **VisualState** for every possible orientation that can be intercepted by the application. This XAML is basically useless: if you use an empty **VisualState** tag, like in this sample, without specifying anything inside, no changes will be applied and the standard layout of the controls will be used.

What we’re going to do is, inside every **VisualState**, specifying one or more animations, that will tell to our controls how they should behave or look in portrait or landscape mode: we can change any property of any control, so we can hide them, move them, change alignment or orientation, etc.

### Changing value of a property

In this sample we’re going to see a simple animation, that it’s often used because it can be used to simply change the value of a property: it’s perfect if you need to change visibility, alignment, orientation, etc.

We’ll start from a very simple XAML:

<pre class="brush: xml;">&lt;StackPanel x:Name="MainPanel"&gt;
    &lt;TextBlock Text="Text 1" /&gt;
    &lt;TextBlock Text="Text 2" /&gt;
    &lt;TextBlock Text="Text 3" /&gt;
    &lt;TextBlock Text="Text 4" /&gt;
&lt;/StackPanel&gt;</pre>

We have four **TextBlock** controls, inside a **StackPanel**, so they are displayed one below the other. We want that, in landscape mode, the **Orientation** property of the**&nbsp;**panel is changed to **Horizontal** and that the content is aligned at the center of the page.

The first thing to do is to assign a name to every control we want to change during the animation: in this sample, we’ve assigned the name **MainPanel** to the **StackPanel** control, since we want to manipulate its properties.

Here is the **VisualStateManager** definition to accomplish our task: 

<pre class="brush: xml;">&lt;VisualStateManager.VisualStateGroups&gt;
    &lt;VisualStateGroup x:Name="PageOrientationStates"&gt;
        &lt;VisualState x:Name="Landscape"&gt;
            &lt;Storyboard&gt;
                &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="MainPanel" Storyboard.TargetProperty="Orientation"&gt;
                    &lt;DiscreteObjectKeyFrame KeyTime="0" Value="Horizontal" /&gt;
                &lt;/ObjectAnimationUsingKeyFrames&gt;
                &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="MainPanel" Storyboard.TargetProperty="HorizontalAlignment"&gt;
                    &lt;DiscreteObjectKeyFrame KeyTime="0" Value="Center" /&gt;
                &lt;/ObjectAnimationUsingKeyFrames&gt;
            &lt;/Storyboard&gt;
        &lt;/VisualState&gt;
        &lt;VisualState x:Name="LandscapeLeft"&gt;
            &lt;Storyboard&gt;
                &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="MainPanel" Storyboard.TargetProperty="Orientation"&gt;
                    &lt;DiscreteObjectKeyFrame KeyTime="0" Value="Horizontal" /&gt;
                &lt;/ObjectAnimationUsingKeyFrames&gt;
                &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="MainPanel" Storyboard.TargetProperty="HorizontalAlignment"&gt;
                    &lt;DiscreteObjectKeyFrame KeyTime="0" Value="Center" /&gt;
                &lt;/ObjectAnimationUsingKeyFrames&gt;
            &lt;/Storyboard&gt;
        &lt;/VisualState&gt;
        &lt;VisualState x:Name="LandscapeRight"&gt;
            &lt;Storyboard&gt;
                &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="MainPanel" Storyboard.TargetProperty="Orientation"&gt;
                    &lt;DiscreteObjectKeyFrame KeyTime="0" Value="Horizontal" /&gt;
                &lt;/ObjectAnimationUsingKeyFrames&gt;
                &lt;ObjectAnimationUsingKeyFrames Storyboard.TargetName="MainPanel" Storyboard.TargetProperty="HorizontalAlignment"&gt;
                    &lt;DiscreteObjectKeyFrame KeyTime="0" Value="Center" /&gt;
                &lt;/ObjectAnimationUsingKeyFrames&gt;
            &lt;/Storyboard&gt;
        &lt;/VisualState&gt;

        &lt;VisualState x:Name="Portrait" /&gt;
        &lt;VisualState x:Name="PortraitUp" /&gt;
        &lt;VisualState x:Name="PortraitDown" /&gt;
    &lt;/VisualStateGroup&gt;
&lt;/VisualStateManager.VisualStateGroups&gt;</pre>

The animation we’re going to use is called **ObjectAnimationUsingKeyFrames**, that can be used to change the control’s properties and to specify the exact time of the animation, so that we can control it. In our case, we don’t want a real animation, but just to change the value. For this reason, every animation will have just one frame (identified by the **ObjectAnimationUsingKeyFrames** object) with the property **KeyTime** set to 0 (so that the animation will trigger immediately). The **ObjectAnimationUsingKeyFrames** object is placed inside the object that identifies the real animation, which is called **ObjectAnimationUsingKeyFrames**: using two attached properties we’re going to set which is the control that we want to animate (**Storyboard.TargetName**) and which is the property we want to change (**Storyboard.TargetProperty**). In this sample we’re going to apply two animations: the first one will change the **Orientation** property, the second one the **HorizontalAlignment** one. The new value that should be assigned is set as **Value** property of the **DiscreteObjectKeyFrame** object. In case the device is rotated in landscape mode, we change the **StackPanel’s Orientation** to **Horizontal** and the **HorizontalAlignment** to **Center**.

We repeat this for every **VisualState** that identifies a landscape orientation: **Landscape, LandscapeLeft** and **LandscapeRight**. We leave, instead, the portrait **VisualStates** empty: this way, when the device is rotated back in portrait, the standard layout is restored. This is a good shortcut to avoid creating other animations simply to put back the controls in the original state.

This kind of animation can be used also if you want to totally change the layout when the orientation is changed: since, in these cases, moving, hiding or aligning too many controls can be complex, it’s easier to create two different layouts (inside, for example, two different **Grid** or **StackPanel** controls) and, by changing the **Visibility** property of the panel, hiding or displaying the proper one. This approach works better if you’re developing the application using the MVVM pattern, since you can’t assign the same **x:Name** property value to two controls. Using binding, instead, you’re able to assign the same property to multiple controls without problems.

[<img title="object1" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px 10px 0px 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="object1" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/object1_thumb.png?resize=200%2C333" width="200" height="333"  data-recalc-dims="1" />](https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/object1.png)[<img title="object2" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="object2" src="https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/object2_thumb.png?resize=300%2C180" width="300" height="180"  data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/object2.png)

### Moving objects with animations

Another common situation is to move controls with an animation, to change the layout. In this case we can use a **DoubleAnimation**, which can be used to change one of the properties that has a numeric value, like the width, the height or the font size. To move a control in the page, we need to use a transformation, that are a way to change the layout of the control without having to redefine every property. For example, if we want to move an object in the page we can use a **TranslateTransform,** that supports changing the X and Y coordinates of the control. We’ll start from a situation similar to the previous sample: we&#8217; have a **StackPanel** with some **TextBlock** controls inside.

The first thing is to apply a **TranslateTrasform** to the **StackPanel**, so that we can use it in the animation to move the control.

<pre class="brush: xml;">&lt;StackPanel&gt;
    &lt;StackPanel.RenderTransform&gt;
        &lt;TranslateTransform x:Name="PanelTranslate"/&gt;
    &lt;/StackPanel.RenderTransform&gt;
    &lt;TextBlock Text="Text 1" /&gt;
    &lt;TextBlock Text="Text 2" /&gt;
&lt;/StackPanel&gt;</pre>

In this case, we don’t give a value to the X and Y property of the **TranslateTransform** object, since we want to keep it in the original position when it’s loaded. We give it just a name, because we’ll need to interact with it when the device is rotated in landscape mode. Here is how we define our **VisualStateManager**:

<pre class="brush: xml;">&lt;VisualStateManager.VisualStateGroups&gt;
    &lt;VisualStateGroup x:Name="PageOrientationStates"&gt;
        &lt;VisualState x:Name="Landscape"&gt;
            &lt;Storyboard&gt;
                &lt;DoubleAnimation Storyboard.TargetName="PanelTranslate" Storyboard.TargetProperty="X" To="400" /&gt;
            &lt;/Storyboard&gt;
        &lt;/VisualState&gt;
        &lt;VisualState x:Name="LandscapeLeft"&gt;
            &lt;Storyboard&gt;
                &lt;DoubleAnimation Storyboard.TargetName="PanelTranslate" Storyboard.TargetProperty="X" To="400" /&gt;
            &lt;/Storyboard&gt;
        &lt;/VisualState&gt;
        &lt;VisualState x:Name="LandscapeRight"&gt;
            &lt;Storyboard&gt;
                &lt;DoubleAnimation Storyboard.TargetName="PanelTranslate" Storyboard.TargetProperty="X" To="400" /&gt;
            &lt;/Storyboard&gt;
        &lt;/VisualState&gt;

        &lt;VisualState x:Name="Portrait" /&gt;
        &lt;VisualState x:Name="PortraitUp" /&gt;
        &lt;VisualState x:Name="PortraitDown" /&gt;
    &lt;/VisualStateGroup&gt;
&lt;/VisualStateManager.VisualStateGroups&gt;</pre>

For every **VisualState** that refers to a landscape position, we set a **DoubleAnimation**. Also in this case we need to set which is the control we want to animate (the **TranslateTransform** object) and which is the property we want to change (the **X** position). Plus, we define the ending value of the animation using the **To** property: in this sample, we’re telling that the control should move from the starting position to the coordinate 400 of the X axe.

This kind of approach can be useful when you have some controls which position needs to be rearranged when the device is rotated.

[<img title="translation1" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px 10px 0px 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="translation1" src="https://i1.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/translation1_thumb.png?resize=200%2C333" width="200" height="333"  data-recalc-dims="1" />](https://i0.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/translation1.png)[<img title="translation2" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="translation2" src="https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/translation2_thumb.png?resize=300%2C180" width="300" height="180"  data-recalc-dims="1" />](https://i2.wp.com/wp.qmatteoq.com/wp-content/uploads/2013/04/translation2.png)

### 

### In the end

This is just one of the available approaches to manage orientations. Another good solution is to use behaviors, like the MVP Andr� s Velv� rt explains in [his blog post](http://dotneteers.net/blogs/vbandi/archive/2011/03/08/handling-wp7-orientation-changes-via-visual-states.aspx). Personally I like more the approach I’ve described in this post because is similar to the one offered by Windows 8 and Windows Store apps, but it’s up to you to choose what you like best!

<div id="scid:fb3a1972-4489-4e52-abe7-25a00bb07fdf:4693fc9e-1952-41ee-b21c-0c3b57946b43" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <p>
    <a href="http://wp.qmatteoq.com/wp-content/uploads/2013/04/Orientation.zip" target="_blank">Download the sample project</a>
  </p>
</div>