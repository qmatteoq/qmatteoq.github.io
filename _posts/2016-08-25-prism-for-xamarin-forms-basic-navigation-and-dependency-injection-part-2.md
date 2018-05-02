---
id: 6585
title: 'Prism for Xamarin Forms &ndash; Basic navigation and dependency injection (Part 2)'
date: 2016-08-25T16:00:00+00:00
author: qmatteoq
layout: post
guid: http://blog.qmatteoq.com/?p=6585
permalink: /prism-for-xamarin-forms-basic-navigation-and-dependency-injection-part-2/
categories:
  - Xamarin
tags:
  - MVVM
  - Prism
  - Xamarin
  - Xamarin Forms
---
<a href="http://blog.qmatteoq.com/?p=6582" target="_blank">In the previous post</a> we’ve started to see the basic concepts on how to leverage the new version of Prism (6.2) to implement the MVVM pattern in a Xamarin Forms app. So far, we haven’t seen nothing special that we couldn’t do also with another framework: we have just created a View, a ViewModel and we connected them through binding. In this post, we’re going to see how Prism can be helpful to handle a very common scenario which can be hard to handle in a MVVM app: navigation and page’s lifecycle.

As we’ve mentioned in the previous post, we’re going to create a simple client for <a href="https://www.trackseries.tv/" target="_blank">TrackSeries</a>, a website which offers many information about TV Shows. The app will display the current top series and will allow the user to discover more about them. To achieve this goal, we can use a set of REST services provided by the website, which are very simple to use and which follow the standard best practices of dealing with REST services: you invoke a URL using a HTTP command and you receive back a JSON response with the result.

For example, if you want to know which are the top series at the moment, you can just perform a HTTP GET request to the following URL: [https://api.trackseries.tv/v1/Stats/TopSeries](https://api.trackseries.tv/v1/Stats/TopSeries "https://api.trackseries.tv/v1/Stats/TopSeries") The service will return you a JSON response with all the details about the top series:

<pre class="brush: js;">[
   {
      "id":121361,
      "name":"Game of Thrones",
      "followers":10230,
      "firstAired":"2011-04-17T21:00:00-04:00",
      "country":"us",
      "overview":"Seven noble families fight for control of the mythical land of Westeros. Friction between the houses leads to full-scale war. All while a very ancient evil awakens in the farthest north. Amidst the war, a neglected military order of misfits, the Night's Watch, is all that stands between the realms of men and the icy horrors beyond.",
      "runtime":55,
      "status":"Continuing",
      "network":"HBO",
      "airDay":"Sunday",
      "airTime":"9:00 PM",
      "contentRating":"TV-MA",
      "imdbId":"tt0944947",
      "tvdbId":121361,
      "tmdbId":1399,
      "language":"en",
      "images":{
         "poster":"http://static.trackseries.tv/banners/posters/121361-49.jpg",
         "fanart":"http://static.trackseries.tv/banners/fanart/original/121361-15.jpg",
         "banner":"http://static.trackseries.tv/banners/graphical/121361-g22.jpg"
      },
      "genres":[
         {
            "id":2,
            "name":"Adventure"
         },
         {
            "id":4,
            "name":"Drama"
         },
         {
            "id":5,
            "name":"Fantasy"
         }
      ],
      "added":"2014-08-08T13:30:46.227",
      "lastUpdated":"2016-08-18T03:03:50.05",
      "followedByUser":false,
      "slugName":"game-of-thrones"
   },
   {
      "id":257655,
      "name":"Arrow",
      "followers":7517,
      "firstAired":"2012-10-10T20:00:00-04:00",
      "country":"us",
      "overview":"Oliver Queen and his father are lost at sea when their luxury yacht sinks. His father doesn't survive. Oliver survives on an uncharted island for five years learning to fight, but also learning about his father's corruption and unscrupulous business dealings. He returns to civilization a changed man, determined to put things right. He disguises himself with the hood of one of his mysterious island mentors, arms himself with a bow and sets about hunting down the men and women who have corrupted his city.",
      "runtime":45,
      "status":"Continuing",
      "network":"The CW",
      "airDay":"Wednesday",
      "airTime":"8:00 PM",
      "contentRating":"TV-14",
      "imdbId":"tt2193021",
      "tvdbId":257655,
      "tmdbId":1412,
      "language":"en",
      "images":{
         "poster":"http://static.trackseries.tv/banners/posters/257655-8.jpg",
         "fanart":"http://static.trackseries.tv/banners/fanart/original/257655-47.jpg",
         "banner":"http://static.trackseries.tv/banners/graphical/257655-g9.jpg"
      },
      "genres":[
         {
            "id":1,
            "name":"Action"
         },
         {
            "id":2,
            "name":"Adventure"
         },
         {
            "id":4,
            "name":"Drama"
         }
      ],
      "added":"2014-08-08T13:37:00.133",
      "lastUpdated":"2016-08-15T03:11:32.013",
      "followedByUser":false,
      "slugName":"arrow"
   },
   {
      "id":153021,
      "name":"The Walking Dead",
      "followers":7185,
      "firstAired":"2010-10-31T21:00:00-04:00",
      "country":"us",
      "overview":"The world we knew is gone. An epidemic of apocalyptic proportions has swept the globe causing the dead to rise and feed on the living. In a matter of months society has crumbled. In a world ruled by the dead, we are forced to finally start living. Based on a comic book series of the same name by Robert Kirkman, this AMC project focuses on the world after a zombie apocalypse. The series follows a police officer, Rick Grimes, who wakes up from a coma to find the world ravaged with zombies. Looking for his family, he and a group of survivors attempt to battle against the zombies in order to stay alive.\n",
      "runtime":50,
      "status":"Continuing",
      "network":"AMC",
      "airDay":"Sunday",
      "airTime":"9:00 PM",
      "contentRating":"TV-MA",
      "imdbId":"tt1520211",
      "tvdbId":153021,
      "tmdbId":1402,
      "language":"en",
      "images":{
         "poster":"http://static.trackseries.tv/banners/posters/153021-38.jpg",
         "fanart":"http://static.trackseries.tv/banners/fanart/original/153021-77.jpg",
         "banner":"http://static.trackseries.tv/banners/graphical/153021-g44.jpg"
      },
      "genres":[
         {
            "id":1,
            "name":"Action"
         },
         {
            "id":4,
            "name":"Drama"
         },
         {
            "id":6,
            "name":"Horror"
         },
         {
            "id":20,
            "name":"Suspense"
         }
      ],
      "added":"2014-08-08T13:31:18.617",
      "lastUpdated":"2016-08-18T03:04:00.28",
      "followedByUser":false,
      "slugName":"the-walking-dead"
   },
   {
      "id":279121,
      "name":"The Flash (2014)",
      "followers":7069,
      "firstAired":"2014-10-07T20:00:00-04:00",
      "country":"us",
      "overview":"After a particle accelerator causes a freak storm, CSI Investigator Barry Allen is struck by lightning and falls into a coma. Months later he awakens with the power of super speed, granting him the ability to move through Central City like an unseen guardian angel. Though initially excited by his newfound powers, Barry is shocked to discover he is not the only \"meta-human\" who was created in the wake of the accelerator explosion – and not everyone is using their new powers for good. Barry partners with S.T.A.R. Labs and dedicates his life to protect the innocent. For now, only a few close friends and associates know that Barry is literally the fastest man alive, but it won't be long before the world learns what Barry Allen has become... The Flash.",
      "runtime":45,
      "status":"Continuing",
      "network":"The CW",
      "airDay":"Tuesday",
      "airTime":"8:00 PM",
      "contentRating":"TV-14",
      "imdbId":"tt3107288",
      "tvdbId":279121,
      "tmdbId":60735,
      "language":"en",
      "images":{
         "poster":"http://static.trackseries.tv/banners/posters/279121-37.jpg",
         "fanart":"http://static.trackseries.tv/banners/fanart/original/279121-23.jpg",
         "banner":"http://static.trackseries.tv/banners/graphical/279121-g7.jpg"
      },
      "genres":[
         {
            "id":1,
            "name":"Action"
         },
         {
            "id":2,
            "name":"Adventure"
         },
         {
            "id":4,
            "name":"Drama"
         },
         {
            "id":8,
            "name":"Science-Fiction"
         }
      ],
      "added":"2014-08-08T13:45:59.087",
      "lastUpdated":"2016-08-17T03:09:18.7",
      "followedByUser":false,
      "slugName":"the-flash-2014"
   },
   {
      "id":80379,
      "name":"The Big Bang Theory",
      "followers":6922,
      "firstAired":"2007-09-25T20:00:00-04:00",
      "country":"us",
      "overview":"What happens when hyperintelligent roommates Sheldon and Leonard meet Penny, a free-spirited beauty moving in next door, and realize they know next to nothing about life outside of the lab. Rounding out the crew are the smarmy Wolowitz, who thinks he's as sexy as he is brainy, and Koothrappali, who suffers from an inability to speak in the presence of a woman.",
      "runtime":25,
      "status":"Continuing",
      "network":"CBS",
      "airDay":"Monday",
      "airTime":"8:00 PM",
      "contentRating":"TV-PG",
      "imdbId":"tt0898266",
      "tvdbId":80379,
      "tmdbId":1418,
      "language":"en",
      "images":{
         "poster":"http://static.trackseries.tv/banners/posters/80379-43.jpg",
         "fanart":"http://static.trackseries.tv/banners/fanart/original/80379-38.jpg",
         "banner":"http://static.trackseries.tv/banners/graphical/80379-g28.jpg"
      },
      "genres":[
         {
            "id":3,
            "name":"Comedy"
         }
      ],
      "added":"2014-08-08T13:27:13.18",
      "lastUpdated":"2016-08-18T03:03:10.947",
      "followedByUser":false,
      "slugName":"the-big-bang-theory"
   },
   {
      "id":176941,
      "name":"Sherlock",
      "followers":6387,
      "firstAired":"2010-07-25T20:30:00+01:00",
      "country":"gb",
      "overview":"Sherlock is a British television crime drama that presents a contemporary adaptation of Sir Arthur Conan Doyle's Sherlock Holmes detective stories. Created by Steven Moffat and Mark Gatiss, it stars Benedict Cumberbatch as Sherlock Holmes and Martin Freeman as Doctor John Watson.",
      "runtime":90,
      "status":"Continuing",
      "network":"BBC One",
      "airDay":"Sunday",
      "airTime":"8:30 PM",
      "contentRating":"TV-14",
      "imdbId":"tt1475582",
      "tvdbId":176941,
      "tmdbId":19885,
      "language":"en",
      "images":{
         "poster":"http://static.trackseries.tv/banners/posters/176941-11.jpg",
         "fanart":"http://static.trackseries.tv/banners/fanart/original/176941-3.jpg",
         "banner":"http://static.trackseries.tv/banners/graphical/176941-g5.jpg"
      },
      "genres":[
         {
            "id":2,
            "name":"Adventure"
         },
         {
            "id":4,
            "name":"Drama"
         },
         {
            "id":14,
            "name":"Crime"
         },
         {
            "id":16,
            "name":"Mystery"
         },
         {
            "id":21,
            "name":"Thriller"
         }
      ],
      "added":"2014-08-08T13:32:27.247",
      "lastUpdated":"2016-08-17T03:07:09.747",
      "followedByUser":false,
      "slugName":"sherlock"
   },
   {
      "id":263365,
      "name":"Marvel's Agents of S.H.I.E.L.D.",
      "followers":5372,
      "firstAired":"2013-09-24T22:00:00-04:00",
      "country":"us",
      "overview":"Phil Coulson (Clark Gregg, reprising his role from \"The Avengers\" and \"Iron Man\" ) heads an elite team of fellow agents with the worldwide law-enforcement organization known as SHIELD (Strategic Homeland Intervention Enforcement and Logistics Division), as they investigate strange occurrences around the globe. Its members -- each of whom brings a specialty to the group -- work with Coulson to protect those who cannot protect themselves from extraordinary and inconceivable threats, including a formidable group known as Hydra.",
      "runtime":45,
      "status":"Continuing",
      "network":"ABC (US)",
      "airDay":"Tuesday",
      "airTime":"10:00 PM",
      "contentRating":"TV-PG",
      "imdbId":"tt2364582",
      "tvdbId":263365,
      "tmdbId":1403,
      "language":"en",
      "images":{
         "poster":"http://static.trackseries.tv/banners/posters/263365-16.jpg",
         "fanart":"http://static.trackseries.tv/banners/fanart/original/263365-26.jpg",
         "banner":"http://static.trackseries.tv/banners/graphical/263365-g7.jpg"
      },
      "genres":[
         {
            "id":1,
            "name":"Action"
         },
         {
            "id":2,
            "name":"Adventure"
         },
         {
            "id":4,
            "name":"Drama"
         },
         {
            "id":5,
            "name":"Fantasy"
         },
         {
            "id":8,
            "name":"Science-Fiction"
         }
      ],
      "added":"2014-08-08T13:39:45.967",
      "lastUpdated":"2016-08-18T03:05:30.987",
      "followedByUser":false,
      "slugName":"marvels-agents-of-shield"
   },
   {
      "id":81189,
      "name":"Breaking Bad",
      "followers":5227,
      "firstAired":"2008-01-20T21:00:00-04:00",
      "country":"us",
      "overview":"Walter White, a struggling high school chemistry teacher, is diagnosed with advanced lung cancer. He turns to a life of crime, producing and selling methamphetamine accompanied by a former student, Jesse Pinkman, with the aim of securing his family's financial future before he dies.",
      "runtime":45,
      "status":"Ended",
      "network":"AMC",
      "airDay":"Sunday",
      "airTime":"9:00 PM",
      "contentRating":"TV-MA",
      "imdbId":"tt0903747",
      "tvdbId":81189,
      "tmdbId":1396,
      "language":"en",
      "images":{
         "poster":"http://static.trackseries.tv/banners/posters/81189-10.jpg",
         "fanart":"http://static.trackseries.tv/banners/fanart/original/81189-21.jpg",
         "banner":"http://static.trackseries.tv/banners/graphical/81189-g21.jpg"
      },
      "genres":[
         {
            "id":4,
            "name":"Drama"
         },
         {
            "id":14,
            "name":"Crime"
         },
         {
            "id":20,
            "name":"Suspense"
         },
         {
            "id":21,
            "name":"Thriller"
         }
      ],
      "added":"2014-08-08T13:27:33.917",
      "lastUpdated":"2016-08-13T03:01:47.063",
      "followedByUser":false,
      "slugName":"breaking-bad"
   },
   {
      "id":247808,
      "name":"Suits",
      "followers":4835,
      "firstAired":"2011-06-24T21:00:00-04:00",
      "country":"us",
      "overview":"Suits follows college drop-out Mike Ross, who accidentally lands a job with one of New York's best legal closers, Harvey Specter. They soon become a winning team with Mike's raw talent and photographic memory, and Mike soon reminds Harvey of why he went into the field of law in the first place.",
      "runtime":45,
      "status":"Continuing",
      "network":"USA Network",
      "airDay":"Wednesday",
      "airTime":"9:00 PM",
      "contentRating":"TV-14",
      "imdbId":"tt1632701",
      "tvdbId":247808,
      "tmdbId":37680,
      "language":"en",
      "images":{
         "poster":"http://static.trackseries.tv/banners/posters/247808-27.jpg",
         "fanart":"http://static.trackseries.tv/banners/fanart/original/247808-43.jpg",
         "banner":"http://static.trackseries.tv/banners/graphical/247808-g17.jpg"
      },
      "genres":[
         {
            "id":4,
            "name":"Drama"
         }
      ],
      "added":"2014-08-08T13:33:45.423",
      "lastUpdated":"2016-08-18T03:04:21.37",
      "followedByUser":false,
      "slugName":"suits"
   },
   {
      "id":274431,
      "name":"Gotham",
      "followers":4718,
      "firstAired":"2014-09-23T20:00:00-04:00",
      "country":"us",
      "overview":"An action-drama series following rookie detective James Gordon as he battles villains and corruption in pre-Batman Gotham City.",
      "runtime":45,
      "status":"Continuing",
      "network":"FOX (US)",
      "airDay":"Monday",
      "airTime":"8:00 PM",
      "contentRating":"TV-14",
      "imdbId":"tt3749900",
      "tvdbId":274431,
      "tmdbId":60708,
      "language":"en",
      "images":{
         "poster":"http://static.trackseries.tv/banners/posters/274431-17.jpg",
         "fanart":"http://static.trackseries.tv/banners/fanart/original/274431-22.jpg",
         "banner":"http://static.trackseries.tv/banners/graphical/274431-g6.jpg"
      },
      "genres":[
         {
            "id":1,
            "name":"Action"
         },
         {
            "id":4,
            "name":"Drama"
         },
         {
            "id":8,
            "name":"Science-Fiction"
         },
         {
            "id":14,
            "name":"Crime"
         },
         {
            "id":21,
            "name":"Thriller"
         }
      ],
      "added":"2014-08-08T13:44:55.4",
      "lastUpdated":"2016-08-17T03:08:55.473",
      "followedByUser":false,
      "slugName":"gotham"
   }
]
</pre>

To use these APIs in the application, I’ve created a class called **TsApiService** with a set of methods that, by using the **HttpClient** class of the .NET Framework and the popular JSON.NET library, takes care of downloading the JSON, parsing it and returning a set of objects that can be easily manipulated using C#. To structure my solution in a better way, I’ve decided to place all the classes related to the communication with the REST APIs (like services and entities) in another Portable Class Library, called **InfoSeries.Core**, which is a different PCL than the one that hosts the real Xamarin Forms app.

Here is, for example, how the method that takes care of parsing the previous JSON and to return a list of C# objects looks like:

<pre class="brush: csharp;">public async Task&lt;List&lt;SerieFollowersVM&gt;&gt; GetStatsTopSeries()
{
    using (HttpClient client = new HttpClient())
    {
        try
        {
            var response = await client.GetAsync("https://api.trackseries.tv/v1/Stats/TopSeries");
            if (!response.IsSuccessStatusCode)
            {
                var error = await response.Content.ReadAsAsync&lt;TrackSeriesApiError&gt;();
                var message = error != null ? error.Message : "";
                throw new TrackSeriesApiException(message, response.StatusCode);
            }
            return await response.Content.ReadAsAsync&lt;List&lt;SerieFollowersVM&gt;&gt;();
        }
        catch (HttpRequestException ex)
        {
            throw new TrackSeriesApiException("", false, ex);
        }
        catch (UnsupportedMediaTypeException ex)
        {
            throw new TrackSeriesApiException("", false, ex);
        }
    }
}
</pre>

The **GetAsync()** method of the **HttpClient** class performs a GET request to the URL, returning as result the string containing the JSON response. This result is stored into the **Content** property ****of the response: in case the request is successful (we use the **IsSuccessStatusCode** property to check this condition), we use the **ReadAsAsync<T>** method exposed by the **Content** property to automatically convert the JSON result in a collection of **SerieFollowersVM** object. **SerieFollowersVM** is nothing else than a class that maps each property of the JSON response (like **name**, **country** or **runtime**) into a C# property:

<pre class="brush: csharp;">public class SerieFollowersVM
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Followers { get; set; }
    public DateTimeOffset FirstAired { get; set; }
    public string Country { get; set; }
    public string Overview { get; set; }
    public int Runtime { get; set; }
    public string Status { get; set; }
    public string Network { get; set; }
    public DayOfWeek? AirDay { get; set; }
    public string AirTime { get; set; }
    public string ContentRating { get; set; }
    public string ImdbId { get; set; }
    public int TvdbId { get; set; }
    public string Language { get; set; }
    public ImagesSerieVM Images { get; set; }
    public ICollection&lt;GenreVM&gt; Genres { get; set; }
    public DateTime Added { get; set; }
    public DateTime LastUpdated { get; set; }
    public string SlugName { get; set; }
}
</pre>

In the full sample on <a href="https://github.com/qmatteoq/XamarinForms-Prism" target="_blank">GitHub</a> you’ll find many classes like this (which maps the various JSON responses returned by the TrackSeries APIs). Additionally, the **TsApiService** will implement additional methods, one for each API we want to leverage in our application. I won’t explain in details each method, since it would be out of scope for the article: you can see all the details on GitHub. For the purpose of this post, you just need to know that the service simply exposes a set of methods that we can use in the various ViewModels to retrieve info about the available TV Shows.

**Note:** by default, the **HttpClient** class doesn’t offer a **ReadAsAsync<T>** method, which is able to automatically deserialize the JSON response into C# objects. To get access to this extension method, we need to add the **Microsoft.AspNet.WebApi.Client** NuGet package to our Portable Class Library. To get it properly working, you need to add this package to every project of the solution (the Xamarin Forms PCL, the Core PCL and all the platform specific projects).

To properly leverage <a href="http://blog.qmatteoq.com/the-mvvm-pattern-dependency-injection/" target="_blank">dependency injection</a>, however, we need an interface that describes the operations offered by the **TsApiService** class. Here is how our interface looks like:

<pre class="brush: csharp;">public interface ITsApiService
{
    Task&lt;List&lt;SerieFollowersVM&gt;&gt; GetStatsTopSeries();
    Task&lt;SerieVM&gt; GetSerieByIdAll(int id);
    Task&lt;SerieInfoVM&gt; GetSerieById(int id);
    Task&lt;List&lt;SerieSearch&gt;&gt; GetSeriesSearch(string name);
    Task&lt;SerieFollowersVM&gt; GetStatsSerieHighlighted();
}
</pre>

Now that we have a service, we can learn how, thanks to Prism, we can register it into its dependency container and have it automatically injected in our ViewModels. Actually, from this point of view, there’s nothing special to highlight: the approach is the same we would use with any other MVVM framework which leverages a dependency injection approach. First, we need to register the association between the interface and the implementation we want to use in the container. In case of Prism, we need to do it in the **RegisterTypes()** method of the **App** class, by using the **Container** object and the **RegisterType<T, Y**>**()** method (where **T** is the interface and **Y** is the concrete implementation):

<pre class="brush: csharp;">protected override void RegisterTypes()
{
    Container.RegisterTypeForNavigation&lt;MainPage&gt;();
    Container.RegisterType&lt;ITsApiService, TsApiService&gt;();
}
</pre>

Now, since both the **MainPage** and the **TsApiService** are registered in the container, we can get access to it in our ViewModel, by simply adding a parameter in the public constructor, like in the following sample:

<pre class="brush: csharp;">public class MainPageViewModel : BindableBase
{
    private readonly ITsApiService _apiService;

    public MainPageViewModel(ITsApiService apiService)
    {
        _apiService = apiService;
    }
}
</pre>

When the **MainPageViewModel** class will be loaded, the implementation of the **ITsApiService** we’ve registered in the container (in our case, the **TsApiService** class) will be automatically injected into the parameter in the constructor, allowing us to use it in all the other methods and properties we’re going to create in the ViewModel. With this approach, it will be easy for us to change the implementation of the service in case we need it: it will be enough to change the registered type in the **App** class and, automatically, every ViewModel will start to use the new version.

### Handle the navigation’s lifecycle

Now that we have a service that offers a method to retrieve the list of the top series, we need to call it when the ViewModel is loaded: our goal is to display, in the main page of the app, a list of the most trending TV shows. However, we are about to face a common problem when it comes to use the MVVM pattern: the method to retrieve the list of top series is asynchronous but, with the current implementation, the only place where we can perform the data loading is the ViewModel’s constructor, which can’t execute asynchronous calls (in C#, in fact, the constructor of a class can’t be marked with the **async** keyword and, consequently, you can’t use the await prefix with a method). In a non-MVVM application, this problem would be easy to solve, thanks to the navigation’s lifecycle methods offered basically by every platform. Xamarin Forms makes no exception and we could leverage, in the code behind class of a XAML page, the methods **OnAppearing()** and **OnDisappearing()**: since they are events, we can call asynchronous code without issues.

To solve this problem, Prism offers an interface that we can implement in our ViewModels called **INavigationAware**: when we implement it, we have access to the **OnNavigatedTo()** and **OnNavigatedFrom()** events, which we can use to perform data loading or cleanup operations. Here is our **MainPageViewModel** looks like after implementing this interface:

<pre class="brush: csharp;">public class MainPageViewModel : BindableBase, INavigationAware
{
    private readonly TsApiService _apiService;
    private ObservableCollection&lt;SerieFollowersVM&gt; _topSeries;

    public ObservableCollection&lt;SerieFollowersVM&gt; TopSeries
    {
        get { return _topSeries; }
        set { SetProperty(ref _topSeries, value); }
    }

    public MainPageViewModel(TsApiService apiService)
    {
        _apiService = apiService;
    }

    public void OnNavigatedFrom(NavigationParameters parameters)
    {

    }

    public async void OnNavigatedTo(NavigationParameters parameters)
    {
        var result = await _apiService.GetStatsTopSeries();
        TopSeries = new ObservableCollection&lt;SerieFollowersVM&gt;(result);
    }
}
</pre>

As you can see, now we have implemented a method called **OnNavigatedTo()**, where we can safely execute our asynchronous calls and load the data: we call the **GetStatsTopSeries()** method of the **TsApiService** class and we encapsulate the resulting collection into an **ObservableCollection** property. This is the property we’re going to connect, through binding, to a **ListView** control, in order to display the list of TV Shows in the main page.

For completeness, here is how the XAML of the **MainPage** looks like:

<pre class="brush: xml;">&lt;?xml version="1.0" encoding="utf-8" ?&gt;
&lt;ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:prism="clr-namespace:Prism.Mvvm;assembly=Prism.Forms"
             prism:ViewModelLocator.AutowireViewModel="True"
             x:Class="InfoSeries.Views.MainPage"
             Title="Info Series"&gt;
  
  &lt;ContentPage.Resources&gt;
    &lt;ResourceDictionary&gt;
      &lt;DataTemplate x:Key="TopSeriesTemplate"&gt;
        &lt;ViewCell&gt;
          &lt;ViewCell.View&gt;
            &lt;Grid&gt;
              &lt;Grid.ColumnDefinitions&gt;
                &lt;ColumnDefinition Width="1*" /&gt;
                &lt;ColumnDefinition Width="2*" /&gt;
              &lt;/Grid.ColumnDefinitions&gt;

              &lt;Image Source="{Binding Images.Poster}" Grid.Column="0" x:Name="TopImage" /&gt;
              &lt;StackLayout Grid.Column="1" Margin="12, 0, 0, 0" VerticalOptions="Start"&gt;
                &lt;Label Text="{Binding Name}" FontSize="18" TextColor="#58666e" FontAttributes="Bold" /&gt;
                &lt;StackLayout Orientation="Horizontal"&gt;
                  &lt;Label Text="Runtime: " FontSize="14" TextColor="#58666e" /&gt;
                  &lt;Label Text="{Binding Runtime}" FontSize="14" TextColor="#98a6ad" Margin="5, 0, 0, 0" /&gt;
                &lt;/StackLayout&gt;
                &lt;StackLayout Orientation="Horizontal"&gt;
                  &lt;Label Text="Air day: " FontSize="14" TextColor="#58666e" /&gt;
                  &lt;Label Text="{Binding AirDay}" FontSize="14" TextColor="#98a6ad" Margin="5, 0, 0, 0" /&gt;
                &lt;/StackLayout&gt;
                &lt;StackLayout Orientation="Horizontal"&gt;
                  &lt;Label Text="Country: " FontSize="14" TextColor="#58666e" /&gt;
                  &lt;Label Text="{Binding Country}" FontSize="14" TextColor="#98a6ad" Margin="5, 0, 0, 0" /&gt;
                &lt;/StackLayout&gt;
                &lt;StackLayout Orientation="Horizontal"&gt;
                  &lt;Label Text="Network: " FontSize="14" TextColor="#58666e" /&gt;
                  &lt;Label Text="{Binding Network}" FontSize="14" TextColor="#98a6ad" Margin="5, 0, 0, 0" /&gt;
                &lt;/StackLayout&gt;
              &lt;/StackLayout&gt;
            &lt;/Grid&gt;
          &lt;/ViewCell.View&gt;
        &lt;/ViewCell&gt;


      &lt;/DataTemplate&gt;
    &lt;/ResourceDictionary&gt;
  &lt;/ContentPage.Resources&gt;

  &lt;ListView ItemTemplate="{StaticResource TopSeriesTemplate}"
            ItemsSource="{Binding Path=TopSeries}" RowHeight="200"/&gt;
  
&lt;/ContentPage&gt;
</pre>

If you already know Xamarin Forms (or XAML in general), you should find this code easy to understand: the page contains a **ListView** control, with a template that describes how a single TV show looks like. We display the show’s poster, along with some other info like the title, the runtime, the production country, etc. Since, due to the naming convention, the **MainPageViewModel** class is already set as **BindingContext** of the page, we can simply connect them by binding the **ItemsSource** property of the **ListView** with the **TopSeries** collection we’ve previously populated in the ViewModel.

### Navigation with parameters

We’ve seen how to leverage the **OnNavigatedTo()** method to perform data loading, but often this method is useful also for another scenario: retrieving parameters passed by a previous page, which are usually needed to understand the current context (in our sample, in the detail page of our application we need to understand which TV Show the user has selected).

Prism support this feature thanks to a class called **NavigationParameters**, which can be passed as optional parameter of the **NavigationAsync()** method of the **NavigationService** and it’s automatically included as parameter of the **OnNavigatedTo()** and **OnNavigatedFrom()** events. Let’s see how to leverage this feature, by adding a detail page to our application, where to display some additional info about the selected show.

The first step is to add both a new page in the **Views** folder (called **DetailPage.xaml**) and a new class in the **ViewModels** folder (called **DetailPageViewModel.cs**). You need to remember also that every page needs to be registered in the container in the **App** class, inside the **OnRegisterTypes()** method:

<pre class="brush: csharp;">protected override void RegisterTypes()
{
    Container.RegisterTypeForNavigation&lt;MainPage&gt;();
    Container.RegisterTypeForNavigation&lt;DetailPage&gt;();
    Container.RegisterType&lt;ITsApiService, TsApiService&gt;();
}
</pre>

Due to the naming convention, we don’t have to do anything special: the new page and the new ViewModel are already connected. Now we need to pass the selected item in the **ListView** control to the new page. Let’s see, first, how to handle the selection in the **MainPage.** We’ll get some help by a library created by my dear friend <a href="http://codeworks.it/blog/" target="_blank">Corrado Cavalli</a>, which allows to implement behaviors in a Xamarin Forms app. Among the available behaviors, one of them is called **EventToCommand** and it allows us to connect any event exposed by a control to a command defined in the ViewModel. We’re going to use it to connect the **ItemTapped** event of the **ListView** control (which is triggered when the user taps on an item in the list) to a command we’re going to create in the **MainPageViewModel** to trigger the navigation to the detail page.

You can install the package created by Corrado from NuGet: its name is **Corcav.Behaviors**. To use it, you need to add an additional namespace to the root of the **MainPage**, like in the following sample:

<pre class="brush: xml;">&lt;ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:prism="clr-namespace:Prism.Mvvm;assembly=Prism.Forms"
             xmlns:behaviors="clr-namespace:Corcav.Behaviors;assembly=Corcav.Behaviors"
             prism:ViewModelLocator.AutowireViewModel="True"
             x:Class="InfoSeries.Views.MainPage"
             Title="Info Series"&gt;

    ...

&lt;/ContentPage&gt;
</pre>

Then you can apply the behavior to the **ListView** control like you would do in a regular Windows app:

<pre class="brush: xml;">&lt;ListView ItemTemplate="{StaticResource TopSeriesTemplate}"
          ItemsSource="{Binding Path=TopSeries}" RowHeight="200"&gt;
  &lt;behaviors:Interaction.Behaviors&gt;
    &lt;behaviors:BehaviorCollection&gt;
      &lt;behaviors:EventToCommand EventName="ItemTapped" Command="{Binding GoToDetailPage}" /&gt;
    &lt;/behaviors:BehaviorCollection&gt;
  &lt;/behaviors:Interaction.Behaviors&gt;
&lt;/ListView&gt;
</pre>

Thanks to this behavior, we have connected the **ItemTapped** event of the **ListView** control to a command called **GoToDetailPage**, that we’re going to define in the ViewModel. From a framework point of view, Prism doesn’t do anything out of the ordinary to help developers implementing commands: it simply offers a class called **DelegateCommand**, which allows to define the operation to execute when the command is invoked and, optionally, the condition to satisfy to enable the command. If you have some previous experience with MVVM Light, it works exactly in the same way as the **RelayCommand** class. Here is how our command in the **MainPageViewModel** class looks like:

<pre class="brush: csharp;">private DelegateCommand&lt;ItemTappedEventArgs&gt; _goToDetailPage;

public DelegateCommand&lt;ItemTappedEventArgs&gt; GoToDetailPage
{
    get
    {
        if (_goToDetailPage == null)
        {
            _goToDetailPage = new DelegateCommand&lt;ItemTappedEventArgs&gt;(async selected =&gt;
            {
                NavigationParameters param = new NavigationParameters();
                param.Add("show", selected.Item);
                await _navigationService.NavigateAsync("DetailPage", param);
            });
        }

        return _goToDetailPage;
    }
}
</pre>

The command we have created is a parametrized command; in fact, the property type is **DelegateCommand<ItemTappedEventArgs>:** this way, inside the method, we get access to the selected item, which is stored in the **Item** property. The method invoked when the command is triggered shows you how navigation with parameter works: first we create a new **NavigationParameters** object which, in the end, is nothing but a dictionary, where you can store key / value pairs. Consequently, we simply add a new item with, as key, the keyword **show** and, as value, the selected item, which type is **SerieFollowersVM.** This is the only difference compared to the navigation we’ve seen in the **App** class: the rest is the same, which means that we call the **NavigateAsync()** method of the **NavigationService**, passing as parameter the key that identifies the detail page (which is **DetailPage**) and the parameter.

**Important!** In the **App** class we were able to automatically use the **NavigationService** because it inherits from the **PrismApplication** class. If we want to use the **NavigationService** in a ViewModel (like in this case), we need to use the traditional approach based on dependency injection. The **NavigationService** instance is already registered in the Prism container, so we simply have to add an **INavigationService** parameter to the public constructor of the **MainPageViewModel**:

<pre class="brush: csharp;">public MainPageViewModel(TsApiService apiService, INavigationService navigationService)
{
    _apiService = apiService;
    _navigationService = navigationService;
}
</pre>

Now that we have performed the navigation to the detail page, we need to retrieve the parameter in the **DetailPageViewModel** class. The first step, like we did for the **MainPageViewModel**, is to let it inherit from the **INavigationAware** interface, other than the **BindableBase** class. This way, we have access to the **OnNavigatedTo()** event:

<pre class="brush: csharp;">public class DetailPageViewModel : BindableBase, INavigationAware
{
    private SerieFollowersVM _selectedShow;

    public SerieFollowersVM SelectedShow
    {
        get { return _selectedShow; }
        set { SetProperty(ref _selectedShow, value); }
    }

    public DetailPageViewModel()
    {

    }

    public void OnNavigatedFrom(NavigationParameters parameters)
    {
            
    }

    public void OnNavigatedTo(NavigationParameters parameters)
    {
        SelectedShow = parameters["show"] as SerieFollowersVM;
    }
}
</pre>

The previous code shows you how to handle the parameter we’ve received from the main page: the same **NavigationParamaters** object we’ve passed, in the **MainPageViewModel**, to the **NavigateAsync()** method is now passed as parameter of the **OnNavigatedTo()** method. As such, we can simply retrieve the item we’ve previously stored with the **show** key. In this case, since we are expecting an object which type is **SerieFollowersVM**, we can perform a cast and store it into a property of the ViewModel called **SelectedShow.** Thanks to this property, we can leverage binding to connect the various information of the selected show to the controls in the XAML page. Here is how the **DetailPage.xaml** looks like:

<pre class="brush: xml;">&lt;?xml version="1.0" encoding="utf-8" ?&gt;
&lt;ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:prism="clr-namespace:Prism.Mvvm;assembly=Prism.Forms"
             prism:ViewModelLocator.AutowireViewModel="True"
             Title="{Binding Path=SelectedShow.Name}"
             x:Class="InfoSeries.Views.DetailPage"&gt;

  &lt;StackLayout&gt;
    &lt;Image x:Name="InfoPoster"
           Source="{Binding Path=SelectedShow.Images.Fanart}" Aspect="AspectFill" /&gt;
    &lt;Label Text="{Binding Path=SelectedShow.Overview}" LineBreakMode="WordWrap" FontSize="13" TextColor="#98a6ad" Margin="15" /&gt;
  &lt;/StackLayout&gt;

&lt;/ContentPage&gt;
</pre>

The content is very simple: we display an image of the show (stored in the **SelectedShow.Images.Fanart** property) and a brief description (stored in the **SelectedShow.Overview** property).

### Wrapping up

In this post we’ve seen some basic concepts to handle navigation and dependency injection in a Xamarin Forms app created with Prism as MVVM framework. In the next post we’re going to see a couple of advanced scenarios, related to navigation and handling of platform specific code. You can find the sample app used for this post on my GitHub repository: [https://github.com/qmatteoq/XamarinForms-Prism](https://github.com/qmatteoq/XamarinForms-Prism "https://github.com/qmatteoq/XamarinForms-Prism")