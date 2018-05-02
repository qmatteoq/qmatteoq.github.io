---
id: 1372
title: 'First steps with Caliburn Micro in Windows Phone 8 &ndash; The theory'
date: 2013-02-04T15:00:00+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=1372
permalink: /first-steps-with-caliburn-micro-in-windows-phone-8-the-theory/
categories:
  - Windows Phone
tags:
  - Caliburn
  - MVVM
  - Windows Phone
---
If you’re a Windows Phone developer you’ll probably have heard of Model-View-ViewModel (MVVM for friends): it’s the most widely approach used for development with technologies based on XAML, like WPF, Silverlight and, more recently, Windows Phone and Windows Store apps. I won’t go deep about MVVM in this post, this is not its purpose and probably it would require 10 or 15 posts to cover all the aspects.****

Let’s just say that it&#8217;s a way to split your code into three main concepts, in order to separate the layers that are responsible of managing the data of your application from the one that are responsible to interact with the user interface.

I’ve always been a huge fan of the [MVVM Light toolkit](http://mvvmlight.codeplex.com/) by Laurent Bugnion, but I’m also a curious developer so I started to give it a try to another famous MVVM toolkit, which is Caliburn Micro. Why am I talking about toolkits and not libraries? Because MVVM i**s a pattern**, that is a way to define the architecture of an application. For this reason, these toolkits **don’t implement MVVM**, but they offer a set of helpers and utilities to help the developer in implementing the pattern.

MVVM Light and Caliburn.Micro have two different approaches. Let’s see pros and cons.

### MVVM Light

MVVM Light is very lightweight and simple and leaves the developer in control of everything. For this reason, the infrastructure provided by the toolkit is very simple. Basically, it offers some helpers to implement the basic requirements of a MVVM application, like:

  * A class, from which a ViewModel can inherit, in order to provide basic functionalities like INotifyPropertyChanged support
  * A command implementation, to manage operations that are executed when a user interacts with a control
  * A messenger implementation, in order to exchange messages between different view models or between a view model and a view
  * A built in simple dependency injection container, to automatically resolve view models and their dependencies

Which are the pros of using MVVM Light? Since its implementation is very simple, it gives a lot of power to the developer to customize it and to adapt the MVVM architecture to fit its needs. Plus, since MVVM Light uses the standard concepts that should be familiar to every XAML developer, like binding and the INotifyPropertyChanged implementation, it’s very simple to understand for other developers that need to work on your project.

The cons is basically just one: since the implementation is very simple, the developer has to do a lot of work to implement everything and to manage correctly scenarios that are very common in a Windows Phone application, like tombstoning or navigation, but that are hard to manage in a MVVM application due to the separation between the view and the view model.

### Caliburn Micro

Caliburn Micro, on the other side, has a more complex configuration than MVVM Light, that allows the developer to be more productive and to hide all the mechanism that under the hood are used to implement the MVVM pattern.

Caliburn Micro is heavily based on conventions to connect the different part of the application: for example, you don’t have to use a ViewModelLocator class to dispatch the view models, because it’s enough to give to a model the same name of the view plus the **ViewModel** suffix (for example, MainView and MainViewModel) to connect the two of them and have the ViewModel automatically bound with the DataContext of the view.

Or, for example, you just have to give to a control (for example, a TextBlock) the same name of a property defined in the ViewModel to connect the two of them in binding, so that if the property changes also the control is refreshed to display the updated value.

Which are the pros of using Caliburn Micro? The most important one is that most of the overhead that you have to add in your application to support MVVM is hidden by the toolkit. This approach minimizes the efforts needed for the developer, that usually needs to do many repetitive steps every time he needs to start a new project and to configure the environment. Plus, Caliburn Micro comes with many helpers to specifically resolve common Windows Phone scenarios, like managing the tombstone, interacting with the application lifecycle or using launchers.

The cons are that, in my opinion, the initial setup is a bit more complex than in MVVM Light, due to the infrastructure needed to support all the features. Plus, being heavily based on conventions is very helpful for the developer, but makes the project hard to understand for people that don’t know Caliburn Micro and that can’t understand all the operations that are made under the hood without an explanation. Just to make an example, since there isn’t a ViewModelLocator or a central dispatcher of the view models, it’s almost impossible to understand how Views and ViewModels are connected together if you don’t know the naming convention I’ve explained you before. However, one nice thing about Caliburn Micro is that it supports conventions, but **you don’t have to use them if you don’t want**: as we’ll see in the next posts, exactly like with every other MVVM toolkit, since the ViewModel is the DataContext of a View, so you can also use standard bindings to connect things together.

### The practice… coming soon

In the next post we’ll start to make some practice and we’ll start to set up a Windows Phone project that uses Caliburn Micro. Stay tuned!

**The Caliburn Micro posts series**

  1. The theory
  2. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-the-first-project/" target="_blank">The first project</a>
  3. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-actions/" target="_blank">Actions</a>
  4. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-collections-and-navigation/" target="_blank">Collections and navigation</a>
  5. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-tombstoning/" target="_blank">Tombstoning</a>
  6. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-advanced-navigation-and-deep-links/" target="_blank">Advanced navigation and deep links</a>
  7. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-messaging/" target="_blank">Messaging</a>
  8. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-using-launchers-and-choosers/" target="_blank">Using launchers and choosers</a>
  9. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-use-your-own-services-and-how-to-pass-data-between-different-pages/" target="_blank">Use your own services </a>
 10. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-the-application-bar/" target="_blank">The Application Bar</a>
 11. <a href="http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-pivot/" target="_blank">Pivot</a>
 12. [Lazy loading with pivot](http://wp.qmatteoq.com/first-steps-with-caliburn-micro-in-windows-phone-8-lazy-loading-with-pivot/)