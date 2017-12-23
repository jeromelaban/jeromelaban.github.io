---
layout: post
title: "[WP7Dev] Using the WebClient with Reactive Extensions for Effective Asynchronous Downloads"
date: 2010-06-22 21:07:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2010/06/22/WP7Dev-Using-the-WebClient-with-Reactive-Extensions-for-Effective-Asynchronous-Downloads", "/post/2010/06/22/wp7dev-using-the-webclient-with-reactive-extensions-for-effective-asynchronous-downloads"]
author: jay
---
<!-- more -->
<p>There&rsquo;s a very cool framework that has slipped into the Windows Phone SDK : The <a href="http://msdn.microsoft.com/en-us/devlabs/ee794896.aspx" target="_blank"> Reactive Extensions</a>.</p>
<p>It's actually a quite misunderstood framework, mainly because it is a bit hard to harness, but when you get a handle on it, it is very handy ! I particularly like the <a href="http://jaylee.org/post/2010/02/04/Rx-MemoizeAll.aspx">MemoizeAll</a> extension, a tricky one, but very powerfull.</p>
<p>But I digress.</p>
<p>&nbsp;</p>
<h2>A Non-Reactive String Download Sample</h2>
<p>On Windows Phone 7, the <a href="http://msdn.microsoft.com/en-us/library/system.net.webclient%28VS.95%29.aspx" target="_blank">WebClient</a> class only has a <a href="http://msdn.microsoft.com/en-us/library/system.net.webclient.downloadstringasync%28VS.95%29.aspx" target="_blank">DownloadStringAsync</a> method and a corresponding <a href="http://msdn.microsoft.com/en-us/library/system.net.webclient.downloadstringcompleted%28VS.95%29.aspx" target="_blank">DownloadStringCompleted</a> event. That means that you're forced to be asynchronous, to be nice to the UI and not make the application freeze on the user, because of the bad coding habit of being synchronous on remote calls.</p>
<p>In a world without the reactive extensions, you would use it like this :</p>
<pre class="brush: c-sharp">public void StartDownload()
{
    var wc = new WebClient();
    wc.DownloadStringCompleted += 
      (e, args) =&gt; DownloadCompleted(args.Result);
                  
    // Start the download
    wc.DownloadStringAsync(new Uri("http://www.data.com/service"));
}

public void DownloadCompleted(string value)
{
    myLabel.Text = value;
}
</pre>
<p>Pretty easy. But you soon find out that the execution of the DownloadStringCompleted event is performed on the UI Thread. That means that if, for some reason you need to perform some expensive calculation after you&rsquo;ve received the string, you&rsquo;ll freeze the UI for the duration of your calculation, and since the Windows Phone 7 is all about fluidity and you don't want to be the bad guy, you then have to queue it in the ThreadPool.</p>
<p>But you also have to update the UI in the dispatcher, so you have to come back from the thread pool.</p>
<p>You then have :</p>
<pre class="brush: c-sharp"> public void StartDownload()
 {
&nbsp;&nbsp;&nbsp;&nbsp; WebClient wc = new WebClient();
&nbsp;&nbsp;&nbsp;&nbsp; wc.DownloadStringCompleted += 
        (e, args) =&gt; ThreadPool.QueueUserWorkItem(d =&gt; DownloadCompleted(args.Result));

&nbsp;&nbsp;&nbsp;&nbsp; // Start the download
&nbsp;&nbsp;&nbsp;&nbsp; wc.DownloadStringAsync(new Uri("http://www.data.com/service"));
  }

 public void DownloadCompleted(string value)
 {
&nbsp;&nbsp;&nbsp;&nbsp; // Some expensive calculation
&nbsp;&nbsp;&nbsp;&nbsp; Thread.Sleep(1000);

&nbsp;&nbsp;&nbsp;&nbsp; Dispatcher.BeginInvoke(() =&gt; myLabel.Text = value);
 }</pre>
<p>That&rsquo;s a bit more complex. And then you notice that you also have to handle exceptions because, well, it&rsquo;s the Web. It&rsquo;s unreliable.</p>
<p>So, let&rsquo;s add the exception handling :</p>
<pre class="brush: c-sharp">public void StartDownload()
{
    WebClient wc = new WebClient();

    wc.DownloadStringCompleted += (e, args) =&gt; {
        try {
            ThreadPool.QueueUserWorkItem(d =&gt; DownloadCompleted(args.Result));
        }
        catch (WebException e) {
            myLabel.Text = "Error !";
        }
    };
   
    // Start the download
    wc.DownloadStringAsync(new Uri("http://www.data.com/service"));
}

public void DownloadCompleted(string  value)
{
    // Some expensive calculation
    Thread.Sleep(1000);
    Dispatcher.BeginInvoke(() =&gt; myLabel.Text = value);
}
</pre>
<p>That&rsquo;s starting to be a bit complex. <strong>But then you have to wait for an other call from an other WebClient to end its call and show both results.</strong></p>
<p>Oh well. Fine, I'll spare you that one.</p>
<p>&nbsp;</p>
<h2>The Same Using the Reactive Extensions</h2>
<p>The reactive extensions treats asynchronous events like a stream of events. You subscribe to the stream of events and leave, and you let the reactive framework do the heavy lifting for you.</p>
<p>I&rsquo;ll spare you the explanation of the duality between IObservable and IEnumerable, because <a href="http://research.microsoft.com/en-us/um/people/emeijer/" target="_blank"> Erik Meijer</a> explains it <a href="http://channel9.msdn.com/shows/Going+Deep/Expert-to-Expert-Brian-Beckman-and-Erik-Meijer-Inside-the-NET-Reactive-Framework-Rx/" target="_blank">very well</a>.</p>
<p>So, I&rsquo;ll start again with the simple example, and after adding the System.Observable and System.Reactive references, I can downloading a string :</p>
<pre class="brush: c-sharp">public void StartDownload()
{
    WebClient wc = new WebClient();

    var o = Observable.FromEvent&lt;DownloadStringCompletedEventArgs&gt;(wc, "DownloadStringCompleted")

                      // When the event fires, just select the string and make
                      // an IObservable&lt;string&gt; instead
                      .Select(newString =&gt; newString.EventArgs.Result);

    // Subscribe to the observable, and set the label text
    o.Subscribe(s =&gt; myLabel.Text = s);


    // Start the download
    wc.DownloadStringAsync(new Uri("http://www.data.com/service"));
}
</pre>
<p>This does the same thing the very first example did. You&rsquo;ll notice the use of Observable.FromEvent to transform the event into a string from the DownloadStringCompleted event args. For this exemple, the event stream will only contain one event, since the download only occurs once. Each of these ocurrence of the event is then &ldquo;projected&rdquo;, using the Select statement, to a string that represents the result of the web request.</p>
<p>It&rsquo;s a bit more complex for the simple case, because of the additional plumbing.</p>
<p>But now we want to handle the threads context changes. The Reactive Extensions has the concept of scheduler, to observe an IObservable in a specific context.</p>
<p>So, we use the scheduler like this :</p>
<pre class="brush: c-sharp">public void StartDownload()
{
    WebClient wc = new WebClient();

    var o = Observable.FromEvent&lt;DownloadStringCompletedEventArgs&gt;(wc, "DownloadStringCompleted")

                      // Let's make sure that we&rsquo;re on the thread pool
                      .ObserveOn(Scheduler.ThreadPool)

                      // When the event fires, just select the string and make
                      // an IObservable&lt;string&gt; instead
                      .Select(newString =&gt; ProcessString(newString.EventArgs.Result))

                      // Now go back to the UI Thread
                      .ObserveOn(Scheduler.Dispatcher)

                      // Subscribe to the observable, and set the label text
                      .Subscribe(s =&gt; myLabel.Text = s);

    wc.DownloadStringAsync(new Uri("http://www.data.com/service"));
}

public string ProcessString(string s)
{
    // A very very very long computation
    return s + "1";
}
</pre>
<pre>&nbsp;</pre>
<p>In this example, we&rsquo;ve changed contexts twice to suit our needs, and now, it&rsquo;s getting a bit less complex than the original sample.</p>
<p>And if we want to handle exceptions, well, easy :</p>
<pre class="brush: c-sharp">    .Subscribe(s =&gt; myLabel.Text = s, e =&gt; myLabel.Text = "Error ! " + e.Message);
</pre>
<p>And you have it !</p>
<p>&nbsp;</p>
<h2>Combining the Results of Two Downloads</h2>
<p>Combining two or more asynchronous operations can be very tricky, and you have to handle exceptions, rendez-vous and complex states. That make a very complex piece of code that I won&rsquo;t write here, I promised, but instead I&rsquo;ll give you a sample using Reactive Extensions :</p>
<pre class="brush: c-sharp">public IObservable&lt;string&gt; StartDownload(string uri)
{
    WebClient wc = new WebClient();

    var o = Observable.FromEvent&lt;DownloadStringCompletedEventArgs&gt;(wc, "DownloadStringCompleted")

                      // Let's make sure that we're not on the UI Thread
                      .ObserveOn(Scheduler.ThreadPool)

                      // When the event fires, just select the string and make
                      // an IObservable&lt;string&gt; instead
                      .Select(newString =&gt; ProcessString(newString.EventArgs.Result));

    wc.DownloadStringAsync(new Uri(uri));

    return o;
}

public string ProcessString(string s)
{
    // A very very very long computation
    return s + "&lt;!-- Processing End --&gt;";
}

public void DisplayMyString()
{
    var asyncDownload = StartDownload("http://bing.com");
    var asyncDownload2 = StartDownload("http://google.com");

    // Take both results and combine them when they'll be available
    var zipped = asyncDownload.Zip(asyncDownload2, (left, right) =&gt; left + " - " + right);

    // Now go back to the UI Thread
    zipped.ObserveOn(Scheduler.Dispatcher)

          // Subscribe to the observable, and set the label text
          .Subscribe(s =&gt; myLabel.Text = s);
}
</pre>
<p>You&rsquo;ll get a very interesting combination of google and bing :)</p>
{% include imported_disclaimer.html %}
