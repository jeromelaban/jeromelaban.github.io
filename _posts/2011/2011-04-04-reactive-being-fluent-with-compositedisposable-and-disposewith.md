---
layout: post
title: "[Reactive] Being fluent with CompositeDisposable and DisposeWith"
date: 2011-04-04 13:48:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2011/04/04/Reactive-Being-fluent-with-CompositeDisposable-and-DisposeWith", "/post/2011/04/04/reactive-being-fluent-with-compositedisposable-and-disposewith"]
author: jay
---
<!-- more -->
<p>When you're writing a few queries with the <a href="http://msdn.microsoft.com/en-us/devlabs/ee794896">reactive extensions</a>, you'll probably end up doing a lot of this kind of code :</p>
<p>[code:c#]</p>
<p>moveSubscription = Observable.FromEvent&lt;MouseEventArgs&gt;(this, "MouseMove")<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Subscribe(_ =&gt; { });</p>
<p>clickSubscription = Observable.FromEvent&lt;MouseEventArgs&gt;(this, "MouseClick")<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Subscribe(_ =&gt; { });&nbsp;</p>
<p>[/code]</p>
<p>You'll probably call the dispose method on both subscriptions in some dispose method in the class that creates the subscriptions. But this is a bit too manual for my taste.</p>
<p>Rx provides the <a href="http://msdn.microsoft.com/en-us/library/microsoft.phone.reactive.compositedisposable_methods(v=vs.92).aspx">CompositeDisposable</a> class, which is basically a list of <a href="http://msdn.microsoft.com/en-us/library/aax125c9(vs.95)">IDisposable</a> instances that all get disposed when CompositeDisposable.Dispose() is called. So we can write it like this :</p>
<p>[code:c#]</p>
<p>var cd = new CompositeDisposable();</p>
<p>var moveSubscription = Observable.FromEvent&lt;MouseEventArgs&gt;(this, "MouseMove")<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; .Subscribe(_ =&gt; { });<br />cd.Add(moveSubscription);</p>
<p>var clickSubscription = Observable.FromEvent&lt;MouseEventArgs&gt;(this, "MouseClick")<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; &nbsp; &nbsp;.Subscribe(_ =&gt; { });<br />cd.Add(clickSubscription);</p>
<p>[/code]</p>
<p>That's better in a sense that&nbsp;it is not needed to get my subscriprions out, but this is still too "manual". There are two lines for each subscriptions, and the need for&nbsp;temporary variables.</p>
<p>That's where a simple&nbsp;DisposeWith extension&nbsp;comes in handy :</p>
<p>[code:c#]</p>
<p>public static class DisposableExtensions<br />{<br />&nbsp;public static void DisposeWith(this IDisposable observable, CompositeDisposable disposables)<br />&nbsp;{<br />&nbsp;&nbsp;&nbsp;&nbsp; disposables.Add(observable);<br />&nbsp;}<br />}<br />[/code]</p>
<p>Very simple extension,&nbsp;a bit more fluent,&nbsp;and that allows to avoid creating a temporary variable just to unsubscribe on a subscription:</p>
<p>[code:c#]</p>
<p>var cd = new CompositeDisposable();</p>
<p>Observable.FromEvent&lt;MouseEventArgs&gt;(this, "MouseMove")<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .Subscribe(_ =&gt; { })<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .DisposeWith(cd);</p>
<p>Observable.FromEvent&lt;MouseEventArgs&gt;(this, "MouseClick")<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .Subscribe(_ =&gt; { })<br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .DisposeWith(cd);</p>
<p>[/code]</p>
<p>This is&nbsp;a method with a side effect, but that's acceptable in this case, and it always ends the declaration of a observable query.</p>
{% include imported_disclaimer.html %}
