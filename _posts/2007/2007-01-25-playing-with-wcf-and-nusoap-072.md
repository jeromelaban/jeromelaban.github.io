---
layout: post
title: "Playing with WCF and NuSOAP 0.7.2"
date: 2007-01-25 10:24:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2007/01/25/Playing-with-WCF-and-NuSOAP-072", "/post/2007/01/25/playing-with-wcf-and-nusoap-072"]
author: jerome
---
<!-- more -->
<p align="justify">
<font face="Verdana" size="2">.NET 3.0&nbsp;has&nbsp;now&nbsp;been released, along with Windows Communication Foundation (WCF). I thought I could give it a shot for one of my projects at work, where I have to create an internal web service. The problem is that the client at the other end&nbsp;will be&nbsp;using <a href="http://sourceforge.net/projects/nusoap/">NuSOAP</a> 0.7.2&nbsp;to contact this WebService, so I had to make sure it would work fine.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">First observation, compared to ASP.NET generated WebService, the WCF&nbsp;wsdl is much more complex. Actually, it has a little bit more information such as the validation rules for a guid, but it also has its schema split up into multiple XSD files. I was a bit worried that NuSOAP wouldn&#39;t handle that well, but it does fine... I also wanted to be able to expose a signature like this :</font>
</p>
<!-- code formatted by http://manoli.net/csharpformat/ -->
<div class="csharpcode">
<pre>
<font face="courier new,courier"><span class="lnum">1: </span>    [<span style="color: lightseagreen">DataContract</span>]
<span class="lnum">2: </span>    <span class="kwrd">public</span> <span class="kwrd">class</span> Parameter
<span class="lnum">3: </span>    {
<span class="lnum">4: </span>        <span class="kwrd">private</span> <span class="kwrd">string</span> _key;
<span class="lnum">5: </span>        <span class="kwrd">private</span> <span class="kwrd">object</span> _value;
<span class="lnum">6: </span>&nbsp;
<span class="lnum">7: </span>        [<span style="color: lightseagreen">DataMember</span>]
<span class="lnum">8: </span>        <span class="kwrd">public</span> <span class="kwrd">string</span> Key
<span class="lnum">9: </span>        {
<span class="lnum">10: </span>            get { <span class="kwrd">return</span> _key; }
<span class="lnum">11: </span>            set { _key = <span class="kwrd">value</span>; }
<span class="lnum">12: </span>       }
<span class="lnum">13: </span>&nbsp;
<span class="lnum">14: </span>       [<span style="color: lightseagreen">DataMember</span>]
<span class="lnum">15: </span>       <span class="kwrd">public</span> <span class="kwrd">object</span> Value
<span class="lnum">16: </span>       {
<span class="lnum">17: </span>            get { <span class="kwrd">return</span> _value; }
<span class="lnum">18: </span>            set { _value = <span class="kwrd">value</span>; }
<span class="lnum">19: </span>       }
<span class="lnum">20: </span>   }</font>
</pre>
</div>
<p align="justify">
<font face="Verdana" size="2">You&#39;ll notice that the second property is an object, and that implies that the serializer/deserializer handles properly types defined at runtime, by using <strong>xsd:anyType </strong>in the schema.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">So, after a few attempts to get working linux LiveCD distro, for which none of them had PHP compiled the correct --enable-soap flag, I decided to fall back from the native PHP SOAP extensions to the OpenSource SOAP library NuSOAP and use EasyPHP on Windows.</font>
</p>
<p align="justify">
<font face="Verdana" size="2">First, I had to change NuSOAP&#39;s response encoding to UTF-8 using <strong>soap_defencoding</strong>, which seems to be the default for WCF, then I had to figure out how to pass an arry of structures to call my method.</font>
</p>
<p>
<font face="Verdana" size="2">So, for the following signature :</font>
</p>
<div class="csharpcode">
<pre>
<font face="courier new,courier">    <span class="kwrd">void</span> MyMethod(<span class="kwrd">string</span> a, <span class="kwrd">string</span> b, <span style="color: lightseagreen">Parameter</span>[] parameters)</font>
</pre>
</div>
<p>
<font face="Verdana" size="2">The caller&#39;s php parameter structure should be :</font>
</p>
<!-- code formatted by http://manoli.net/csharpformat/ -->
<pre class="csharpcode">
$<span class="kwrd">params</span> = array(
<span class="str">&#39;a&#39;</span> =&gt; $a,
<span class="str">&#39;b&#39;</span> =&gt; $b,
<span class="str">&#39;parameters&#39;</span> =&gt; array(
<span class="str">&#39;Parameter&#39;</span> =&gt; array(
array(<span class="str">&#39;Key&#39;</span> =&gt; <span class="str">&#39;abc&#39;</span>, <span class="str">&#39;Value&#39;</span> =&gt; 10),
array(<span class="str">&#39;Key&#39;</span> =&gt; <span class="str">&#39;def&#39;</span>, <span class="str">&#39;Value&#39;</span> =&gt; 42)
)
),
); 
</pre>
<p align="justify">
<font face="Verdana" size="2">Notice that you have to place a&nbsp;&quot;Parameter&quot; element inside the &quot;parameters&quot; parameter.</font>
</p>
<p>
<font face="Verdana" size="2">Then, to call the method, use the following line :</font>
</p>
<div class="csharpcode">
<pre>
<font face="courier new,courier">    $client-&gt;call(&quot;MyMethod&quot;, array($params));</font>
</pre>
</div>
<p align="justify">
<font face="Verdana" size="2">by encapsulating the parameter array once again. This makes a lot of levels to call one little function... I prefer the .NET way of doing thing :)</font>
</p>
<p align="justify">
<font face="Verdana" size="2">An other problem came up right at this point : The array, although the information being in the soap body, was not filled on the .NET side. After comparing with a .NET to WCF call, I figured that there was a&nbsp;missing namespace. This is what is generated by default with the code I&#39;ve presented above using NuSOAP :</font>
</p>
<div class="csharpcode">
<pre>
<font face="courier new,courier">&lt;parameters&gt;
&lt;Parameter&gt;
&lt;Key&gt;abc&lt;/Key&gt;
&lt;Value&gt;10&lt;/Value&gt;
&lt;/Parameter&gt;
&lt;Parameter&gt;
&lt;Key&gt;def&lt;/Key&gt;
&lt;Value&gt;42&lt;/Value&gt;
&lt;/Parameter&gt;
&lt;/parameters&gt;</font>
</pre>
</div>
<p align="justify">
<font face="Verdana" size="2">And this is what .NET is generating : </font>
</p>
<div class="csharpcode">
<pre>
<font face="courier new,courier">&lt;parameters&gt;
&lt;Parameter xmlns=&quot;http://schemas.datacontract.org/2004/07/MyService&quot;&gt;
&lt;Key&gt;abc&lt;/Key&gt;
&lt;Value&gt;10&lt;/Value&gt;
&lt;/Parameter&gt;
&lt;Parameter xmlns=&quot;http://schemas.datacontract.org/2004/07/MyService&quot;&gt;
&lt;Key&gt;def&lt;/Key&gt;
&lt;Value&gt;42&lt;/Value&gt;
&lt;/Parameter&gt;
&lt;/parameters&gt;</font>
</pre>
</div>
<p align="justify">
<font face="Verdana" size="2">For some reason, the WCF basicHttp binding point is generating two different namespaces for the service contract and for the data contract. To fix this, you just have to specify explicitly&nbsp;a namespace for each ServiceContract and DataContract attribute of your service :</font>
</p>
<div class="csharpcode">
<pre>
   <font face="courier new,courier"> [<span style="color: lightseagreen">DataContract</span>(&quot;http://mywebservice.example.com&quot;)]</font>
</pre>
</div>
<p align="justify">
<font face="Verdana" size="2">The other problem was about using an unspecified data type as a member of a structure, a <strong>System.Object</strong> in my case. Well, it turns out that NuSOAP does not support it, as it does not include the data type of the serialized element, so the WCF deserializer cannot interpret it. I changed the data type back to string, unfortunately losing the type information. I can get it from somewhere else but still, this can lead to serialization problems related to culture (comma being a dot and the opposite depending on systems, for instance).</font>
</p>
<p align="justify">
<font face="Verdana" size="2">Anyway, there are a few things to remember to get things to work fine with NuSOAP :</font>
</p>
<ul>
	<li>
	<div align="justify">
	<font face="Verdana" size="2">Change the encoding to UTF-8 or whatever encoding you choose to use, </font>
	</div>
	</li>
	<li>
	<div align="justify">
	<font face="Verdana" size="2">Don&#39;t forget to specify the name of the type of an element in an array in PHP,&nbsp;</font>
	</div>
	</li>
	<li>
	<div align="justify">
	<font face="Verdana" size="2">Do not expose unspecified parameters or attributes,</font>
	</div>
	</li>
	<li>
	<div align="justify">
	<font face="Verdana" size="2">Explicitly specify the namespace of each <strong>DataContract</strong> and <strong>ServiceContact</strong> attribute of your service. </font>
	</div>
	</li>
</ul>
<p align="justify">
<font face="Verdana" size="2">It&#39;s been a while since I&#39;ve written a line of PHP code, and I didn&#39;t miss it at all. I&#39;m going back to WCF now :)</font>
</p>

{% include imported_disclaimer.html %}
