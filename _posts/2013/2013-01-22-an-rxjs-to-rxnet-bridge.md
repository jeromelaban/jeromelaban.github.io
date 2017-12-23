---
layout: post
title: "An RxJS to Rx.NET bridge"
date: 2013-01-22 21:19:00 -0500
comments: true
category: Archive
tags: [".NET", "Windows 8"]
redirect_from: ["/post/2013/01/22/An-RxJS-to-RxNET-bridge", "/post/2013/01/22/an-rxjs-to-rxnet-bridge"]
author: jay
---
<!-- more -->
<p><em>tl;dr: JavaScript can call C# code in Metro apps, and allows for C# code to observe notifications coming from JavaScript, using the Reactive Extensions grammar.</em></p>
<p>Javascript is (almost) everywhere, literally.</p>
<p>Not in everyone's minds though, which is only around <a href="https://sites.google.com/site/pydatalog/pypl/PyPL-PopularitY-of-Programming-Language">7% according to the PYPL index</a>.</p>
<p>Yet, Microsoft is pushing a lot on that front, particularly in Metro apps where the documentation shows JavaScript examples first. Subjectively, I&rsquo;m not a big fan of the decision, because it pushes us back a few years and not simply because of the language, but because state&nbsp;of the full JavaScript ecosystem, compared to the .NET counterpart.</p>
<p>[more]</p>
<h2>JavaScript ?</h2>
<p><a href="http://www.youtube.com/watch?v=hQVTIJBZook">Quirkiness of the language aside</a>, things are going in a good direction -- particularly for large applications -- <a href="http://www.typescriptlang.org/">with TypeScript</a>, where type safety is introduced, among other things.</p>
<p>Arguably, JavaScript is not very good a parallelism (which is <strong>very</strong> different from asynchrony), mainly because it is single-threaded. There are ways around it, like using Web Workers, but this is bare-metal isolated parallelism, where everything needs to be done by hand. Maybe soon enough, we&rsquo;ll end up with someone reinventing a .NET Remoting (or DCOM) look alike to address these issues.</p>
<p>We&rsquo;re going in a multi-core CPU direction, because processors scaling up seems to have slowed down quite a bit, and that scaling out is now the trend. The immutability trend seems to rise here and there to embrace this change, C# hints at taking that route with <a href="http://blogs.msdn.com/b/bclteam/archive/2012/12/18/preview-of-immutable-collections-released-on-nuget.aspx">BCL&rsquo;s immutable collections</a>, and F# has it at its core.</p>
<p>I foresee JavaScript lagging behind in that regard, but maybe that&rsquo;s not a problem for now because it is used as an HTML back-end (and used it for this only&nbsp;for the time being). We still need to address a lot of concerns with&nbsp;raw performance, particularly when applications are not just displaying data, but are actually performing a lot of business computation on the client side.</p>
<p>&nbsp;</p>
<h2>Mixing C# and JavaScript</h2>
<p>Fortunately, there is a very good way to do actual parallelism in Metro apps because of two things :</p>
<ul>
<li>JavaScript can call WinMD files code</li>
<li>C# can expose code as WinMD files</li>
</ul>
<p>and in C#, there is Rx.NET that excels at mixing both asynchrony and parallelism. It is very easy to share and use immutable data using Rx and its Query based programming which makes it a good fit.</p>
<p>It&rsquo;s actually pretty easy to create a mixed environments application :</p>
<ul>
<li>Create a WinJS app</li>
<li>Create a C# Windows Runtime Component, which creates a WinMD file</li>
<li>Add a reference to the WinMD project in the WinJS project</li>
<li>Call some code, and voila !</li>
</ul>
<p>Sounds too good to be that easy ?&nbsp;Yeah, you&rsquo;re actually right&hellip;</p>
<p>&nbsp;</p>
<h2>Bridging RxJS and Rx.NET</h2>
<p>Microsoft&rsquo;s preferred way for exchanging messaging is through events. Even in WinMD type definitions, and pretty much all APIs in WinRT expose some sort of .NET like events.</p>
<p>I&rsquo;m not a big fan of events, for subscription lifetime, memory management, composition, delegation, &hellip; I&rsquo;m transforming them into Rx observable&nbsp;where ever I can, and being able to have a common way&nbsp;for exchanging asynchronous&nbsp;notifications, using&nbsp;multi-valued returning functions,&nbsp;between both worlds seems a pretty good way to go.</p>
<p>Both RxJS and Rx.NET seem to share the same notification&nbsp;grammar, meaning that it is should be very easy to map between the two.</p>
<p>But there is a problem with all this, being that JavaScript objects, as far as I know,&nbsp;cannot be marshaled through to .NET. Only WinMD exposed types, primitives and delegates can. If you try to pass a Javascript object, you&rsquo;ll be greeted with a Type Mismatch exception&hellip;</p>
<p>&nbsp;</p>
<h2>Exposing an RxJS observable to Rx.NET</h2>
<p>The idea is to be able to expose an RxJS observable to an Rx.NET C# observer, and subscribe to it.</p>
<p><em>Note: Excuse my JavaScript, I&rsquo;m probably using it somehow the wrong way&hellip; Feel free to correct me !</em></p>
<p>First, here is the JavaScript&nbsp;observable we want to subscribe to :</p>
<pre>var observable = Rx.Observable.return(42);</pre>
<p>To be able to pass this around we need to expose a way to subscribe to that observable inside JavaScript, but allow for C# to provide onNext, onError and onCompleted handlers, but also a way to dispose the created subscription :</p>
<pre class="brush: javascript">Rx.Observable.prototype.toWrapper = function () {
        var parent = this;
        return new Umbrella.ObservableWrapper(
            function (onNext, onError, onCompleted) {
                var subscription = parent.subscribe(
                   onNext, 
                   function (e) { return onError(e.toString()); },
                   onCompleted
                );

                return function () { subscription.dispose(); }
            }
        );
    }; 
    </pre>
<p>For the same reasons we can&rsquo;t pass the observable directly to C#, we can&rsquo;t pass the subscription returned by to subscribe back to C#. We also can&rsquo;t pass the exception as-is.</p>
<p>The trick here is to provide a first function that create the subscription, and return another that disposes it.</p>
<p>The ObservableWrapper class is a WinMD exposed C# class that takes this form :</p>
<pre class="brush: c-sharp">public sealed class ObservableWrapper
    {
        private SubscribeHandler _subscribe;
</pre>
<pre class="brush: c-sharp"> 
        public ObservableWrapper(SubscribeHandler subscribe)
        {
            _subscribe = subscribe;
        }
</pre>
<pre class="brush: c-sharp"> 
        internal IObservable&lt;object&gt; AsObservable()
        {
            return Observable.Create&lt;object&gt;(o =&gt; {
                var disposeAction = _subscribe(
                    v =&gt; o.OnNext(v), 
                    e =&gt; o.OnError(new Exception(e.ToString())),
                    o.OnCompleted
                );
 
                return Disposable.Create(() =&gt; disposeAction());
            });
        }
    }
    </pre>
<p>The wrapper is created with a delegate that calls subscribe, and the observer is decomposed as three lambdas for the notifications. The subscribe delegate create a delegate that will dispose the subscription on the JavaScript side, when the C# observable will be disposed.</p>
<p>Also note that the class is sealed, and that the AsObservable method is internal, because it exposes .NET only types and will only be used from .NET code.</p>
<p>Next, C# methods can then take ObservableWrapper instances as parameters, which can be subscribed to :</p>
<pre class="brush: c-sharp">public void DoStuff(ObservableWrapper wrapper)
{
   var s = 
      wrapper
       .AsObservable()
       .Subscribe(
          v =&gt; Debug.WriteLine(v), 
          e =&gt; Debug.WriteLine(e),
          () =&gt; Debug.WriteLine("Completed")
       );
}
</pre>
<p>It can then used like this, on the JavaScript side :</p>
<blockquote>
<pre class="brush: javascript">var foo = new MyApp.Foo(); 
foo.doStuff(observable.toWrapper());</pre>
</blockquote>
<p>All this works pretty well !</p>
<p>I&rsquo;ll probably talk on how to do it the other way around in another blog post, as there are some other tricks to use to make this work.</p>
{% include imported_disclaimer.html %}
