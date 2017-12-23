---
layout: post
title: "Implementing an asynchronous settings service, Part 1 : Going Async"
date: 2012-10-07 21:32:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows 8", "Windows Phone Dev"]
redirect_from: ["/post/2012/10/07/Implementing-an-asynchronous-settings-service-part-1-going-async", "/post/2012/10/07/implementing-an-asynchronous-settings-service-part-1-going-async"]
author: jay
---
<!-- more -->
<p><em>TL;DR: This article is part of a series about implementing asynchronous services contract, and starts by&nbsp;an the creation of basic functionality&nbsp;for a User Settings storage service using C# 5.0 async features. In the next episode, we'll talk about writing a user setting and consuming settings change notifications.</em></p>
<p>&nbsp;</p>
<p>Most applications require the storage of user settings, such as the login, authentication token, preferences, you name it. All these settings have been stored in many locations over the years, such <a href="http://msdn.microsoft.com/en-us/library/windows/desktop/ms724353%28v=vs.85%29.aspx">ini files</a>, <a href="http://msdn.microsoft.com/en-us/library/system.configuration.configurationmanager.appsettings.aspx">AppSettings</a>, <a href="http://msdn.microsoft.com/en-us/library/system.io.isolatedstorage.isolatedstoragesettings(v=vs.95).aspx">IsolatedStorageSettings </a>in Silverlight and more recently in&nbsp;<a href="http://msdn.microsoft.com/en-US/library/windows/apps/windows.storage.applicationdata.roamingsettings">RoamingStorage</a> or LocalStorage&nbsp;in WinRT, etc&hellip;</p>
<p>Most of the time, this is pretty much CRUD-like contracts that do not offer much to perform asynchronous reading/writing, easy two-way data-binding and notifications. Some offer change notifications, but most of the time, this is related to roaming settings that have been updated.</p>
<p>Reading settings in a synchronous way is a common performance issue, and most of the time accessing the data can be expensive, which is not good for the user experience. For instance, the Silverlight and Windows Phone&nbsp;IsolatedStorageSettings implementation has a pretty big performance hit during the first access, due to the internal use of Xml Serialization (and its heavy use of reflection). It also requires to be synchronized during persistence, and its persistence takes a lot of time too, suspending simultaneous read or write operations.</p>
<p>In this article, I&rsquo;m going to discuss an implementation of a service that abstracts the use of application or users settings using the C# async/await keywords.</p>
<p>[more]</p>
<h2>A basic settings service</h2>
<p>Considering that reading settings needs to be asynchronous, we can implement an interface that looks like this:</p>
<pre class="brush: c-sharp"> public interface ISettingsService
 {
     Task&lt;T&gt; Read&lt;T&gt;(string key, Func&lt;T&gt; defaultValue);
     Task Write&lt;T&gt;(string key, T value);
 }
</pre>
<p>This is very simple service that can read and write content to a Settings storage.</p>
<pre class="brush: c-sharp">public class SettingsService : ISettingsService
{
&nbsp;&nbsp;&nbsp; private ApplicationDataContainer _data;
&nbsp;&nbsp;&nbsp; private object _gate = new object();

&nbsp;&nbsp;&nbsp; public SettingsService(ApplicationDataContainer data)
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _data = data;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; public async Task&lt;T&gt; Read&lt;T&gt;(string key, Func&lt;T&gt; defaultValue)
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return await Task.Run(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; () =&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lock(_gate)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   object value;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   if (!_data.Values.TryGetValue(key, out value))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; value = defaultValue();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;return (T)value;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; );
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; public async Task Write&lt;T&gt;(string key, T value)
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; await Task.Run(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; () =&gt; { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    lock(_gate) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;       _data.Values[key] = value;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;    }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; );
&nbsp;&nbsp;&nbsp; }
}
</pre>
<p>This basic implementation turns the synchronous API of ApplicationDataContainer into an asynchronous API. Concurrent access is protected via a simple <a href="http://msdn.microsoft.com/en-us/library/System.Threading.Monitor.aspx">monitor (lock)</a>, and this is needed because of the use of <a href="http://msdn.microsoft.com/en-us/library/hh195051.aspx">Task.Run</a> which runs in the background, using the Task Pool.</p>
<h2>Consuming the Settings Service</h2>
<p>Now, because of the async nature of this service, this means that the code that consumes it needs&nbsp;to be adapted.</p>
<p>Here is an example :</p>
<pre class="brush: c-sharp">public class MainViewModel : INotifyPropertyChanged
{
&nbsp;&nbsp;&nbsp; private readonly ISettingsService _settings;

&nbsp;&nbsp;&nbsp; public MainViewModel(ISettingsService settings)
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _settings = settings;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FillSettings();
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; private async void FillSettings()
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MySetting = await _settings.Read&lt;string&gt;("MySetting", () =&gt; "42");
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; private string _mySetting = null;
&nbsp;&nbsp;&nbsp; public string MySetting
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; get { return _mySetting; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; set
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _mySetting = value;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; OnPropertyChanged();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; private void OnPropertyChanged([CallerMemberName] string propertyName = null)
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if (PropertyChanged != null)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PropertyChanged(this, new PropertyChangedEventArgs(propertyName));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; public event PropertyChangedEventHandler PropertyChanged;
}
</pre>
<p>Note that the FillSettings async void method is called from the MainViewModel to initiate the get the value of a setting. Constructors cannot be marked async.</p>
<h3>Property State Management</h3>
<p>There a first problem at this point.&nbsp;While the async call is in progress, the value of the MySetting property will be null. This means that code that may depend on the availability of this value will most probably break and needs to be refactored to wait for that value to be available.</p>
<p>In this example, there is actually only one asynchronous assigned property, but if the view model code depends on multiple properties such as code in a UI command, state management code can grow pretty fast to handle all the cases. Going a bit further, the code that waits for the initialization of all properties will not necessarily be the same that handles the change (and error conditions)&nbsp;of these properties after they've been assigned their first value, adding another layer of complexity.</p>
<h3>Task Cancellation</h3>
<p>There is a second problem with the FillSettings method. It&nbsp;does not provide any cancellation mechanism, meaning that the async processing will continue even though the view model has been destroyed. The method can be changed to an async task, and could be passed a Cancellation token, but this increases the plumbing of the method. In this particular case though, since the underlying Settings is synchronous, cancelling it is not possible. However, in the case of an actual async operation, such as using a WebRequest, cancelling is possible.</p>
<p>The cancellation concept in the TPL is something that permeates at all levels of an API and underlying code. It is required for every method used in the underlying code to expose a <a href="http://msdn.microsoft.com/en-us/library/system.threading.cancellationtoken.aspx">CancellationToken</a>, making cancellation a very apparent and viral consideration. It pretty much needs to be passed as a parameter or&nbsp;as part of a closure,&nbsp;everywhere in the inner call stack&nbsp;of an async method.</p>
<h3>UI Thread Affinity</h3>
<p>The third problem with this implementation is that the FillSettings method is implicitly bound to the UI Thread, because of the fact that the PropertyChanged event must be raised on the UI Thread. Moving the creation of this view model to a background thread would require a change in the way the property changed is raised, such as introducing explicitly the notion of UI Thread.</p>
<p>In all fairness, many MVVM frameworks take care of all this UI thread plumbing back and forth plumbing, but some don&rsquo;t.</p>
<p>A solution for this would be to do the following :</p>
<pre class="brush: c-sharp">public class MainViewModel : IDisposable, INotifyPropertyChanged
{
    private SynchronizationContext _dispatcher;

    public MainViewModel(ISettingsService settings, SynchronizationContext dispatcher)
    {
        // ...
        _dispatcher = dispatcher;
    }

    // ...

    private void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        if (PropertyChanged != null)
        {
            _dispatcher.Post(_ =&gt; 
                PropertyChanged(this, new PropertyChangedEventArgs(propertyName)),
                null
            );
        }
    }
}
</pre>
<p>Injecting a synchronization context allows for an easy dispatch to the UI thread, but also allows for an easier unit testing and avoid a dependency to the actual dispatcher.</p>
<h2>In the next episode...</h2>
<p>We'll talk about writing a new setting value3&nbsp;to the settings service, and how to get notified that a setting has changed.</p>
{% include imported_disclaimer.html %}
