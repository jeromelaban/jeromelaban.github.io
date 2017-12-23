---
layout: post
title: "Implementing an asynchronous settings service, Part 3 : Getting notified"
date: 2012-10-10 21:25:00 -0400
comments: true
category: Archive
tags: []
redirect_from: ["/post/2012/10/10/Implementing-an-asynchronous-settings-service-Part-3-Getting-notified", "/post/2012/10/10/implementing-an-asynchronous-settings-service-part-3-getting-notified"]
author: jay
---
<!-- more -->
<h2>&nbsp;</h2>
<p>and getting notified about changes in these settings.It can be particularly interesting, for a ViewModel in a weather app to be notified that a new temperature unit has been selected, or that it needs to be notified that the user is now signed in and the UI should change accordingly.Surely, interacting with the Settings Service directly may not be always the best idea, and that there should be some abstraction between a view model and the settings service&nbsp;(particularly for the user credentials), but you get the idea.</p>
<h2>&nbsp;</h2>
<h2>Notifying of Settings Changes</h2>
<p>Now going a bit farther, code that depends on these settings may need to be notified that a setting has changed. Other than the bad solution of polling for values of a specific setting,&nbsp;a better one is&nbsp;to use C# events:</p>
<pre class="brush: c-sharp">public class SettingsService : ISettingsService
{
&nbsp;&nbsp;&nbsp; // ...

&nbsp;&nbsp;&nbsp; public event EventHandler&lt;SettingChangedEventArgs&gt; SettingChanged;
&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp; // ...

&nbsp;&nbsp;&nbsp; public async Task Write&lt;T&gt;(string key, T value)
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; await Task.Run(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; () =&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;_data.Values[key] = value;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if (SettingChanged != null)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SettingChanged(this, new SettingChangedEventArgs(key, value));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; );
&nbsp;&nbsp;&nbsp; }
}
</pre>
<p>This is the commonly used way to notify listeners in C#. It requires the creation of a SettingChangedEventArgs to notify the listeners of the event.</p>
<p>You'll note, if you've read the <a href="http://jaylee.org/post/2012/07/08/c-sharp-async-tips-and-tricks-part-2-async-void.aspx">Part 2 of my Async Tips and Tricks</a>, that there's a call to Task.Run, which forces the inner code to run in the background. It is the inner code which raises the event, which means the subscribers will be notified in the background thread's context. This may require the listener to post back to the UI thread to perform work.</p>
<p>Using this event is fairly easy WPF-like frameworks, let&rsquo;s look at a ViewModel :</p>
<pre class="brush: c-sharp">public class MainViewModel : IDisposable, INotifyPropertyChanged
{
&nbsp;&nbsp; &nbsp;// ...

&nbsp;&nbsp;&nbsp; public MainViewModel(ISettingsService settings)
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // ...
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _settings.SettingChanged += OnSettingChanged;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; public void Disposable()
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; _settings.SettingChanged -= OnSettingChanged;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; private void OnSettingChanged(object sender, Settings.SettingChangedEventArgs e)
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; switch (e.Key)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; case "MySetting":
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MySetting = (string)e.Value;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; break;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; // ...
}
</pre>
<p>You&rsquo;ll note that the property&nbsp;filtering logic must be integrated directly into the OnSettingsChanged method, but also that the notion of setting type has been lost because the event cannot be typed. It is&nbsp;also&nbsp;required for the class to now implement deterministic cleanup with the IDisposable interface to avoid memory leaks that may be introduced by the use of the C# event.</p>
<p>Finally, the same logical operation needs to be split at least in three places, which is not very convenient for maintenance and code readability.</p>
<h2>Next&hellip;</h2>
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;">This simple example is taking a synchronous approach, and using the standard C# techniques, is make use of&nbsp;the three different elements:&nbsp;asynchrony, parallelism and the time that passes by in the form of notifications.&nbsp;C# and the .NET BCL&nbsp;do not provide out of the&nbsp;box&nbsp;a way to blend these three concepts, which&nbsp;requires the introduction of plumbing code make them work properly together.</p>
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;">In the next article, I&rsquo;ll talk about how to implement this logic using the <a href="http://msdn.microsoft.com/en-us/data/gg577609.aspx">Reactive Extensions</a> and a bit of the <a href="http://rxx.codeplex.com/">Rxx library</a>, and reduce (a lot) the code needed to perform the same operations or at the very least, group them in the same logical place to make them more readable.</p>
<p class="MsoNormal" style="margin: 0cm 0cm 10pt;">&nbsp;</p>
{% include imported_disclaimer.html %}
