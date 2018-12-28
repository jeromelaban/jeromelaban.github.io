---
layout: post
title: "An update to @matthieumezil, Rx and the FileSystemWatcher"
date: 2012-08-26 20:52:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2012/08/26/An-update-to-matthieumezil-Rx-and-the-FileSystemWatcher.aspx", "/post/2012/08/26/an-update-to-matthieumezil-rx-and-the-filesystemwatcher.aspx"]
author: jay
---
<!-- more -->
<p>@MatthieuMEZIL is a fellow MVP and friend of mine, and back in february at the MVP summit we shared our mutual interests with Rx on my side, and Roslyn on his side.</p>
<p>I&rsquo;ve yet to blog about Roslyn, (even though a blog post is in preparation) but he started using Rx and is a bit blogging about it.</p>
<p>Now, Rx is a tricky beast and there are many ways to do the same thing. <a href="https://twitter.com/jlaban/status/231358203248668672">I&rsquo;ve promised</a> Matthieu I&rsquo;d give him a few tricks on <a href="http://msmvps.com/blogs/matthieu/archive/2012/08/02/filesystemwatcher-rx-and-throttle.aspx">how to improve his code</a>. Now that I do have a bit of time, here it is.</p>
<p>[more]</p>
<p>Here is the original code :</p>
<pre class="brush: c-sharp">    public class FileWatcher
    {
        public FileWatcher(string path, string filter, TimeSpan throttle)
        {
            Path = path;
            Filter = filter;
            Throttle = throttle;
        }

        public string Path { get; private set; }
        public string Filter { get; private set; }
        public TimeSpan Throttle { get; private set; }

        public IObservable&lt;FileChangedEvent&gt; GetObservable()
        {
            return Observable.Create&lt;FileChangedEvent&gt;(observer =&gt;
            {
                FileSystemWatcher fileSystemWatcher = new FileSystemWatcher(Path, Filter) { EnableRaisingEvents = true };
                FileSystemEventHandler created = (_, __) =&gt; observer.OnNext(new FileChangedEvent());
                FileSystemEventHandler changed = (_, __) =&gt; observer.OnNext(new FileChangedEvent());
                RenamedEventHandler renamed = (_, __) =&gt; observer.OnNext(new FileChangedEvent());
                FileSystemEventHandler deleted = (_, __) =&gt; observer.OnNext(new FileChangedEvent());
                ErrorEventHandler error = (_, errorArg) =&gt; observer.OnError(errorArg.GetException());

                fileSystemWatcher.Created += created;
                fileSystemWatcher.Changed += changed;
                fileSystemWatcher.Renamed += renamed;
                fileSystemWatcher.Deleted += deleted;
                fileSystemWatcher.Error += error;

                return () =&gt;
                {
                    fileSystemWatcher.Created -= created;
                    fileSystemWatcher.Changed -= changed;
                    fileSystemWatcher.Renamed -= renamed;
                    fileSystemWatcher.Deleted -= deleted;
                    fileSystemWatcher.Error -= error;
                    fileSystemWatcher.Dispose();
                };

            }).Throttle(Throttle);
        }
    }

    static void Main(string[] args)
    {
        var fileWatcher = new FileWatcher(".", "*.*", TimeSpan.FromSeconds(5)); 
        var fileObserver = fileWatcher.GetObservable().Subscribe(fce =&gt; { /*TODO*/ });
    }
</pre>
<p>&nbsp;</p>
<p>There are a few things that can be improved, or be more readable such as :</p>
<ul>
<li><span style="color: #2a2a2a;">The use of Observable.FromEventPattern, to avoid the use of the += and &ndash;= C# event syntax, make the use of a C#&nbsp;event located at a single place and avoid the use of observer.OnNext,</span></li>
<li><span style="color: #2a2a2a;">The use of Merge, to subscribe to all events at once, and propagate all notifications down the stream, without the use of multiple calls to OnNext on the observer. You&rsquo;ll notice that Merge accepts IEnumerable&lt;IObservable&lt;T&gt;&gt; which makes for a very readable merge source.</span></li>
<li><span style="color: #2a2a2a;">The use of Finally, to dispose the only visible state of the expression, the file system watcher itself,</span></li>
<li><span style="color: #2a2a2a;">The use of Subscribe(IObserver), to effectively pass all the notifications from the merged events to the created observable,</span></li>
<li><span style="color: #2a2a2a;">The removal of the instance of &ldquo;FileWatcher&rdquo;, for which the configuration stored can be in the closure used to create the observable. This allows for the creation of a single self-contained method &ldquo;ObserveFolderChanges&rdquo;, without an additional instance whose variables are used only at the creation of the observable.</span></li>
<li><span style="color: #2a2a2a;">The use of Observable.Throw to generate an error notification using the exception provided by the Error event from FileSystemWatcher.</span></li>
<li><span style="color: #2a2a2a;">The use of the OnError delegate in Subscribe, to handle errors coming from the ObserveFolderChanges observable.</span></li>
</ul>
<p>&nbsp;</p>
<p>Here is another version, among others :</p>
<pre class="brush: c-sharp">    public class FileWatcher
    {
        public static IObservable&lt;FileChangedEvent&gt; ObserveFolderChanges(string path, string filter, TimeSpan throttle)
        {
            return Observable.Create&lt;FileChangedEvent&gt;(
                observer =&gt;
                {
                    var fileSystemWatcher = new FileSystemWatcher(path, filter) { EnableRaisingEvents = true };

                    var sources = new[] 
                    { 
                        Observable.FromEventPattern&lt;FileSystemEventArgs&gt;(fileSystemWatcher, "Created")
                                  .Select(ev =&gt; new FileChangedEvent()),

                        Observable.FromEventPattern&lt;FileSystemEventArgs&gt;(fileSystemWatcher, "Changed")
                                  .Select(ev =&gt; new FileChangedEvent()),

                        Observable.FromEventPattern&lt;RenamedEventArgs&gt;(fileSystemWatcher, "Renamed")
                                  .Select(ev =&gt; new FileChangedEvent()),

                        Observable.FromEventPattern&lt;FileSystemEventArgs&gt;(fileSystemWatcher, "Deleted")
                                  .Select(ev =&gt; new FileChangedEvent()),

                        Observable.FromEventPattern&lt;ErrorEventArgs&gt;(fileSystemWatcher, "Error")
                                  .SelectMany(ev =&gt; Observable.Throw&lt;FileChangedEvent&gt;(ev.EventArgs.GetException()))
                    };

                    return sources.Merge()
                                  .Throttle(throttle)
                                  .Finally(() =&gt; fileSystemWatcher.Dispose())
                                  .Subscribe(observer);
                }
            );
        }
    }

    static void Main(string[] args)
    {
        var fileWatcher =
            FileWatcher.ObserveFolderChanges(".", "*.*", TimeSpan.FromSeconds(5))
                        .Subscribe(fce =&gt; { Console.WriteLine("Changed"); }, e =&gt; Debug.WriteLine(e));

        Console.ReadLine();
    }</pre>
<p>&nbsp;</p>
<p>As a side note, Throttle has the effect of discarding events that have been generated in excess, which can be acceptable if the observer only needs to be notified that something changed. But if the observer is interested in <em>what</em> has changed, then Observable.Buffer() would be more appropriate because it would give all events in an List&lt;T&gt; every five seconds.</p>
<p>Finally, the outer Subscribe is probably not going to be the last one that goes back to synchronous code. That subscribe is probably going to be replaced by another observable that reacts to the change of the folder, the propagates that to other observables...</p>
<p>All this to have (eventually)&nbsp;a single subscribe that runs the whole&nbsp;app :)</p>
{% include imported_disclaimer.html %}
