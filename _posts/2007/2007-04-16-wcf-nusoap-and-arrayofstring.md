---
layout: post
title: "WCF, NuSOAP and ArrayOfString"
date: 2007-04-16 15:21:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2007/04/16/WCF-NuSOAP-and-ArrayOfString", "/post/2007/04/16/wcf-nusoap-and-arrayofstring"]
author: jerome
---
<!-- more -->
<p align="justify">
<span style="font-size: 10pt; font-family: Verdana"><font face="trebuchet ms,geneva">When exposing a WebService via WCF, you might want to expose something like this :</font></span> 
</p>
<font face="courier new,courier">[code:c#]<br />
[DataContract]<br />
public class SomeContract<br />
{<br />
&nbsp; [DataMember]&nbsp;<br />
&nbsp;&nbsp;public<strong> <em>string[]</em></strong> Values { get; set; } <br />
}<br />
[/code]</font> 
<p align="justify">
<font face="trebuchet ms,geneva"><span style="font-size: 10pt; font-family: Verdana">For that particular data contract, WCF will be generating a WSDL with something like this :</span><font size="2"> <br />
</font></font><br />
[code:xml]<br />
&lt;xs:complexType name=&quot;SomeContract&quot;&gt;<br />
&nbsp; &lt;xs:sequence&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;xs:element minOccurs=&quot;0&quot; name=&quot;Values&quot;&nbsp;nillable=&quot;true&quot; <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; type=&quot;q1:ArrayOfstring&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <strong><em><u>xmlns:q1=&quot;http://schemas.microsoft.com/2003/10/Serialization/Arrays&quot;</u></em></strong> /&gt;<br />
&nbsp; &lt;/xs:sequence&gt;<br />
&lt;/xs:complexType&gt;<br />
[/code] 
</p>
<p>
<font face="trebuchet ms,geneva"><span style="font-size: 10pt; font-family: Verdana">With ArrayOfString being defined like this :</span> </font>
</p>
<p>
[code:xml]<br />
&nbsp; &lt;xs:schema elementFormDefault=&quot;qualified&quot; <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; targetNamespace=&quot;http://schemas.microsoft.com/2003/10/Serialization/Arrays&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xmlns:xs=&quot;http://www.w3.org/2001/XMLSchema&quot;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;xmlns:tns=&quot;http://schemas.microsoft.com/2003/10/Serialization/Arrays&quot;&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;xs:complexType name=&quot;ArrayOfstring&quot;&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;xs:sequence&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;xs:element minOccurs=&quot;0&quot; maxOccurs=&quot;unbounded&quot; name=&quot;string&quot; nillable=&quot;true&quot; type=&quot;xs:string&quot;/&gt;<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;/xs:sequence&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;/xs:complexType&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;xs:element name=&quot;ArrayOfstring&quot; nillable=&quot;true&quot; <strong><em>type=&quot;tns:ArrayOfstring&quot;</em></strong>/&gt;<br />
&nbsp; &lt;/xs:schema&gt;<br />
[/code] 
</p>
<p align="justify">
<font face="trebuchet ms,geneva"><span style="font-size: 10pt; font-family: Verdana">In general, that would be fine. The type &quot;ArrayOfString&quot; is defined in a different namespace, but this should not be a problem.</span><font size="2"> </font></font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva"><span style="font-size: 10pt; font-family: Verdana">So, to use that particular type in a method call, you should have a document like this one :</span><font size="2"> </font></font>
</p>
[code:xml]<br />
&lt;SomeContract&gt;<br />
&nbsp; &lt;Values <strong><em>xmlns:a=&quot;http://schemas.microsoft.com/2003/10/Serialization/Arrays&quot;</em></strong>&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;<strong><em>a:</em></strong>string&gt;My Value&lt;/<strong><em>a:</em></strong>string&gt;<br />
&nbsp; &lt;/Values&gt;<br />
&lt;/SomeContract&gt;<br />
[/code] 
<p align="justify">
<font face="trebuchet ms,geneva"><span style="font-size: 10pt; font-family: Verdana">&quot;string&quot; elements are contained in a different namespace from the SomeContract element. However, the NuSOAP stock version 0.7.2 has a problem with that kind of schema, and generates instead something like this :</span> </font>
</p>
[code:xml]<br />
&lt;SomeContract&gt;<br />
&nbsp; &lt;Values&gt;<br />
&nbsp;&nbsp;&nbsp; &lt;string&gt;My Value&lt;/string&gt;<br />
&nbsp; &lt;/Values&gt;<br />
&lt;/SomeContract&gt; <br />
[/code]<br />
<br />
<font face="trebuchet ms,geneva"><span style="font-size: 10pt; font-family: Verdana">When the WCF deserializer receives a document like this one, it does not find the &quot;Values&quot; member in the namespace he&#39;s looking and ends up creating a SomeContract instance with a null array of strings.</span><font size="2"> </font></font>
<p align="justify">
<font face="trebuchet ms,geneva"><span style="font-size: 10pt; font-family: Verdana">Since there&#39;s no way of fixing NuSOAP, you may need to tweak your contract to help NuSOAP serializing your data without a namespace.</span><font size="2"> </font></font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva"><span style="font-size: 10pt; font-family: Verdana">The CollectionDataContract attribute seems to be the way to go, since there is a way to specify the namespace to use when generating the metadata. The service contract then looks like this :</span><font size="2"> </font></font>
</p>
<p>
[code:c#]<br />
&nbsp; [DataContract(Namespace = &quot;http://my.name.space&quot;)]&nbsp;<br />
&nbsp; public class SomeContract&nbsp;<br />
&nbsp;&nbsp;{&nbsp;<br />
&nbsp;&nbsp;&nbsp; [DataMember]&nbsp;<br />
&nbsp;&nbsp;&nbsp;&nbsp;public ArrayOfString InvalidIdentifiers { get; set; }&nbsp;<br />
&nbsp;&nbsp;}&nbsp;<br />
<br />
&nbsp;&nbsp;[CollectionDataContract(ItemName=&quot;string&quot;, Namespace=&quot;http://my.name.space&quot;)] <br />
&nbsp; public class ArrayOfString : List&lt;string&gt; { }&nbsp;<br />
[/code] 
</p>
<p>
<font face="trebuchet ms,geneva"><span style="font-size: 10pt; font-family: Verdana">Thereby placing everything in the &quot;http://my.name.space&quot; namespace.</span><font size="2"> </font></font>
</p>
<p align="justify">
<font face="trebuchet ms,geneva"><span style="font-size: 10pt; font-family: Verdana">You might need to tweak a bit the ArrayOfString class, especially if you need to assign it from an actual string[] instance, but you get the idea.</span><font size="2"> </font></font>
</p>

{% include imported_disclaimer.html %}
