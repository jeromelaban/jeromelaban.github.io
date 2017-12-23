---
layout: post
title: "Implementing an asynchronous settings service, Part 2 : Writing a setting"
date: 2012-10-08 20:12:00 -0400
comments: true
category: Archive
tags: [".NET", "Windows 8", "Windows Phone Dev"]
redirect_from: ["/post/2012/10/08/Implementing-an-asynchronous-settings-service-Part-2-Writing-a-setting", "/post/2012/10/08/implementing-an-asynchronous-settings-service-part-2-writing-a-setting"]
author: jay
---
<!-- more -->
<p><em>tl;dr: This article is the second of a series talking about the implementation of an async user Settings Service, using the C# async keyword. This part talks about writing new settings values, asynchronously and the challenges associated with it.</em></p>
<p>&nbsp;</p>
<p>In this continuing implementation of&nbsp;a Settings Service&nbsp;series, I'll continue with the addition new features to the service.</p>
<p><a href="http://jaylee.org/post/2012/10/07/Implementing-an-asynchronous-settings-service-part-1-going-async.aspx">Part one talked about reading the settings asynchronously</a>, and this time, we'll talk about writing user settings new values.</p>
<p>&nbsp;</p>
<h2>Writing to the Settings Service</h2>
<p>Now let's say that the user has selected a new temperature unit in a weather app,&nbsp;writing a new value to the settings service takes a few more lines, [more] by adding this:</p>
<pre class="brush: c-sharp">public string TemperatureUnit
{
&nbsp;&nbsp;&nbsp; // ...
&nbsp;&nbsp;&nbsp; set
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _temperatureUnit = value;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; StoreValue("TemperatureUnit", _temperatureUnit);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; OnPropertyChanged();
&nbsp;&nbsp;&nbsp; }
}

private async void StoreValue(string key, string value)
{
&nbsp;&nbsp;&nbsp; await _settings.Write(key, value);
}
</pre>
<p>This is a pretty simple code, and you'll notice that it uses async void, which should be avoided most of the time. But in this case, since the property is synchronous, using an async&nbsp;Task is not very useful, and produces a compiler warning.</p>
<p>The warning removal&nbsp;is actually not an excuse, but the environment leads us to write that kind of code, because of the mix of synchronous and asynchronous code.</p>
<p>&nbsp;</p>
<h2>Cancellation Management</h2>
<p>Again here, cancellation needs to be added using an explicit cancellation token, passed-in as a parameter which is not represented in the previous example.</p>
<p>But for the sake of clarity, let's look at this :</p>
<pre class="brush: c-sharp">public class MainViewModel : IDisposable, INotifyPropertyChanged
{
    // ...
    private CancellationTokenSource _source = new CancellationTokenSource();

    public void Dispose()
    {
        // ...
        _source.Dispose();
    }

    private async Task StoreValue(string key, string value)
    {
        await _settings.Write(key, value, _source.Token);
    }
}
</pre>
<p>This implementation is using the cancellation token provided by the <a href="http://msdn.microsoft.com/en-us/library/system.threading.cancellationtokensource.aspx">CancellationTokenSource</a> that can be disposed at any time. This will work properly if the view model is disposed before the end of an async operation.</p>
<p>But what happens if a write&nbsp;operation takes way too much time to complete&nbsp;? Do we keep stacking up these async calls ?</p>
<p>This could happen if the settings service is actually sending data off to a remote server, and the server is not responding properly. Another solution would be to create a cancellation token for each call make to Write, and cancel it if there's a new one made before the other ends.</p>
<p>Let's see that :</p>
<pre class="brush: c-sharp">private IDisposable _outstandingWrite;

private async void StoreValue(string key, string value)
{
    if (_outstandingWrite != null)
    {
        _outstandingWrite.Dispose();
    }

    var source = new CancellationTokenSource();
    _outstandingWrite = source;

    await _settings.Write(key, value, source.Token);

    _outstandingWrite = null;
}
</pre>
<p>By creating a disposable field that gets disposed when entering the StoreValue, it's now possible to cancel an outstanding Write operation.</p>
<p>Unfortunately, this code only guaranteed to work if it is running on the UI Thread, because there's only one UI Thread and that processing is serialized. By default, if nothing in particular&nbsp;is done since the property will mostly be assigned by the data-binding engine, everything will run on the Dispatcher.</p>
<p>But if for some reason, the code ends up&nbsp;running on a different thread, the problem here is that the _outstandingWrite private&nbsp;field is now a parallelism liability that could be modified by multiple threads at the same time. We can't put a lock here to protect it though, <a href="http://jaylee.org/post/2012/06/18/CSharp-5-0-Async-Tips-and-Tricks-Part-1.aspx">because it not permitted in an async method</a>.</p>
<p>This leaves us using the <a href="http://msdn.microsoft.com/en-us/library/system.threading.interlocked.aspx">Interlocked </a>class :</p>
<pre class="brush: c-sharp">private IDisposable _outstandingWrite;

private async Task StoreValue(string key, string value)
{
    var source = new CancellationTokenSource();

    // Get the currently running operation, if any
    var existingWrite = Interlocked.Exchange(ref _outstandingWrite, source);

    if (existingWrite != null)
    {
        existingWrite.Dispose();
    }

    await _settings.Write(key, value, source.Token);

    // The operation is done, reset the oustanding write
    Interlocked.Exchange(ref _outstandingWrite, null);
}</pre>
<p>That way, we've synchronized the access to the _outstandingWrite, without the use of a lock statement.</p>
<p>You'll probably agree with me though,&nbsp;that this is getting a bit complex.</p>
<p>&nbsp;</p>
<h2>Write Throttling</h2>
<p>Some more complexity comes when, let&rsquo;s say, you want to throttle the updates coming in from the settings property. This can happen when it's been data-bound using a <a href="http://msdn.microsoft.com/en-us/library/system.windows.data.bindingmode.aspx">Two-Way mode&nbsp;binding</a> on a TextBox.</p>
<p>The code will actually send the new values to the settings service when a certain amount of time has passed, to avoid overloading&nbsp;the service underneath. Let&rsquo;s take a look:</p>
<pre class="brush: c-sharp">int storeCounter = 0;

private async void StoreValue(string key, string value)
{
&nbsp;&nbsp;&nbsp; var storeCounterCapture = ++storeCounter;

&nbsp;&nbsp;&nbsp; await Task.Delay(TimeSpan.FromMilliseconds(300));

&nbsp;&nbsp;&nbsp; if (storeCounterCapture == storeCounter)
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; await _settings.Write(key, value);
&nbsp;&nbsp;&nbsp; }
}
</pre>
<p>The throttle logic is integrated directly into the ViewModel, which is a lot of plumbing to repeat if there are multiple settings. Of course, this can be refactored and placed into a utility class, but there&rsquo;s still going to be a need for state to be explicitly maintained to perform the throttle.</p>
<p>I've excluded the cancellation logic from this sample, but it'd eventually need to be added here.</p>
<p>This can also be used to persist the data to the storage, using the <a href="http://msdn.microsoft.com/en-us/library/system.io.isolatedstorage.isolatedstoragesettings.save(v=vs.95).aspx">IsolatedStorageSettings.Save()</a> method, but only write when enough time has passed by.</p>
<p>&nbsp;</p>
<h2>Next...</h2>
<p>Writing to the settings service is actually not as easy as it seems, as there can be multiple concurrency scenarios to deal with.</p>
<p>In the next article, we'll talk about notifying code that the settings have changed which could be used to alter the behavior of the application when, let's say,&nbsp;the temperature unit has changed.</p>
{% include imported_disclaimer.html %}
