---
id: 251
title: 'Having fun with Azure Mobile Services &#8211; The Windows Phone application'
date: 2012-10-09T10:00:59+00:00
author: qmatteoq
layout: post
guid: http://wp.qmatteoq.com/?p=251
permalink: /having-fun-with-azure-mobile-services-the-windows-phone-application/
aktt_notify_twitter:
  - 'no'
categories:
  - Windows Phone
tags:
  - Azure
  - Windows 8
  - Windows Phone
---
Did you have fun using your Azure Mobile Service with your Windows 8 app? Good, because now the fun will continue! We will do the same operations using a Windows Phone application. As I already anticipated in the previous posts, actually there isn’t a Windows Phone SDK and the reason is very simple: Microsoft is probably waiting for the Windows Phone 8 SDK to be released, in order to proper support both its old and new mobile platform.

But we don’t have to worry: our service is a REST service that supports the OData protocol, so we can interact with it starting from now by using simple HTTP requests and by parsing the HTTP response.

To help us in our work we’ll use two popular libraries:

  * **RestSharp**, that is a wrapper around the HttpWebRequest class that simplifies the code needed to communicate with a REST service. Sometimes RestSharp tries to be “too smart” for my taste, as we’ll see later, but it’s still a perfect candidate for our scenario.
  * **JSON.NET,** that is a library that provides many useful features when you work with JSON data, like serialization, deserialization, plus a manipulation language based on LINQ called LINQ to JSON.

&nbsp;

Let’s start! First you have to open Visual Studio 2010 (the 2012 release isn’t supported yet by the current Windows Phone SDK) and create a new Windows Phone project. Then, using NuGet, we’re going to install the two library: right click on the project, choose **Manage NuGet Packages** and look for the two needed libraries using the keywords **RestSharp** and **JSON.NET**.

The UI of the application will be very similar to the one we’ve used for the Windows 8 app, I’ve just replaced the **ListView** with a **ListBox**.

<pre class="brush: xml;">&lt;StackPanel&gt;
    &lt;Button Content="Insert data" Click="OnAddNewComicButtonClicked" /&gt;
    &lt;Button Content="Show data" Click="OnGetItemButtonClicked" /&gt;
    &lt;ListBox x:Name="ComicsList"&gt;
        &lt;ListBox.ItemTemplate&gt;
            &lt;DataTemplate&gt;
                &lt;StackPanel Margin="0, 20, 0, 0"&gt;
                    &lt;TextBlock Text="{Binding Title}" /&gt;
                    &lt;TextBlock Text="{Binding Author}" /&gt;
                &lt;/StackPanel&gt;
            &lt;/DataTemplate&gt;
        &lt;/ListBox.ItemTemplate&gt;
    &lt;/ListBox&gt;
&lt;/StackPanel&gt;</pre>

### 

### Insert some data

To insert the data we’ll have to send a HTTP request to the service, using the POST method; the body of the request will be the JSON that represent our **Comic** object. Let’s see the code first:

<pre class="brush: csharp;">private void OnAddNewComicButtonClicked(object sender, RoutedEventArgs e)
{

    RestRequest request = new RestRequest("https://myService.azure-mobile.net/tables/Comics");

    request.Method = Method.POST;
    request.AddHeader("X-ZUMO-APPLICATION", "your-application-key");

    Comic comic = new Comic
                       {
                           Title = "300",
                           Author = "Frank Miller"
                       };

    string jsonComic = JsonConvert.SerializeObject(comic, new JsonSerializerSettings
                                                              {
                                                                  NullValueHandling = NullValueHandling.Ignore
                                                              });

    request.AddParameter("application/json", jsonComic, ParameterType.RequestBody);

    RestClient client = new RestClient();
    client.ExecuteAsync(request, response =&gt;
                                     {
                                         MessageBox.Show(response.StatusCode.ToString());
                                     });

}</pre>

The first thing we do is to create a **RestRequest**, which is the RestSharp class that represents a web request: the URL of the request (which is passed as parameter of the constructor) is the URL of our service.

Then we set the HTTP method we’re going to use (POST, in this case, since we’re going to add some data do the table) and we set the application’s secret key: thanks to Fiddler, by intercepting the traffic of our previous Windows 8 application, I’ve found that the key that in our Windows Store app was passed as a parameter of the constructor of the **MobileService** class is added as a request’s header, which name is **X-ZUMO-APPLICATION.** We do the same by using the **AddHeader** method provided by RestSharp.

The next step is to create the **Comic** object we’re going to insert in the table: for this task please welcome Json.net, that we’re going to use to **serialize** our **Comic** object, that means converting it into a plain JSON string. What does it mean? That if you put a breakpoint in this method and you take a look at the content of the **jsonComic** variable, you’ll find a plain text representation of our complex object, like the following one:

<pre class="brush: plain;">{ 
    "Author" : "Frank Miller",
    "Title" : "300"
}</pre>

We perform this task using the **SerializeObject** method of the **JsonConvert** class: other than passing the **Comic** object to the method, we also pass a **JsonSerializerSettings** object, that we can use to customize the serialization process. In this case, we’re telling to the the serializer not to include object’s properties that have **a null value**. This step is very important: do you remember that the **Comic** object has an **Id** property that acts as primary key and that is automatically generated every time we insert a new item in the table? Without this setting, the serializer would add the Id property to the Json with the value “0”. In this case, the request to the service would fail, since it isn’t a valid value: the **Id** property shouldn’t be specified since it’s the database that takes care of generating it.

Once we have the json we can add it to the body of our request: here comes a little trick, to avoid that RestSharp tries to be too smart for us. In fact, if you use the **AddBody** method of the **RestRequest** object you don’t have a way to specify which is the content type of the request. RestSharp will try to guess it and will apply it: the problem is that, sometimes, RestSharp fails to correctly recognize the content type and this is one of these cases. In my tests, every time I’ve tried to add a json in the request’s body, RestSharp set the content type to **text/xml**, that it’s not only wrong, but actively refused by the Azure mobile service, since the only accepted content type is **application/json**.

By using the **AddParameter** method and by manually specifying the content type, the body’s content and the parameter’s type (**ParameterType.RequestBody**) we are able to apply a workaround to this behavior. In the end we can execute the request by creating a new instance of the **RestClient** class and by calling the **ExecuteAsync** method, that accepts as parameters the request and the callback that is executed when we receive the response from our service. Just to trace what’s going on, in the callback we simply display using a **MessageBox** the status code: if everything we did is correct, the status code we receive should be **Created.**

To test it, simply go to the <a href="https://manage.windowsazure.com/" target="_blank">Azure management portal</a>, access to your service’s dashboard and go into the data tab: you should see the new item in the table.

### Play with the data

Getting back the data for display purposes is a little bit simpler: it’s just a GET request, with the same features of the POST request. The difference is that, this time, we’re going to use **deserialization**, which is the process to convert the JSON we receive from the service into C# objects.

<pre class="brush: csharp;">private void OnGetItemButtonClicked(object sender, RoutedEventArgs e)
{
    RestRequest request = new RestRequest("https://myService.azure-mobile.net/tables/Comics");
    request.AddHeader("X-ZUMO-APPLICATION", "my-application-key");
    request.Method = Method.GET;

    RestClient client = new RestClient();
    client.ExecuteAsync(request, result =&gt;
                                     {
                                         string json = result.Content;
                                         IEnumerable&lt;Comic&gt; comics = JsonConvert.DeserializeObject&lt;IEnumerable&lt;Comic&gt;&gt;(json);
                                         ComicList.ItemsSource = comics;
                                     });
}</pre>

The first part of the code should be easy to understand, since it’s the same we wrote to insert the data: we create a **RestRequest** object, we add the header with the application key and we execute asynchronously the request using the **RestClient** object.

This time in the response and, to be more precisely, in the **Content** property we get the JSON with the list of all the comics that are stored in our table.� It’s time to use Json.Net again: this time we’re going to use the **DeserializeObject<T>** method of the **JsonConvert** class, where **T** is the type of the object we expect to have back after the deserialization process. In this case, the return type is **IEnumerable<Comic>,** since our request returns a collection of all the comics stored in the table.

### Some advanced scenarios for playing with the data

In the next posts we’ll see some more operations we can do with the data: the mobile service we’ve created with Azure supports the OData protocol; this means that we can perform additional operations (like filtering the results) directly with the HTTP request, without having to get the whole content of the table first. We’ll see how to do it, in the meantime… happy coding!