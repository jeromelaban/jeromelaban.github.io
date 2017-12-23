---
layout: post
title: "C# 3.0, a sneak peek"
date: 2006-07-22 10:22:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2006/07/22/C-302c-a-sneak-peek", "/post/2006/07/22/c-302c-a-sneak-peek"]
author: jerome
---
<!-- more -->
<p align="justify">
<font size="2"><font face="Tahoma">If you&#39;ve used both DataSet and DataTable, you must have seen the DataTable.Select method. This is an interesting method that allows to select rows using a set of criterias, like IS NULL and comparison operators referencing columns of the current table, as well as columns from other tables using relations. </font><font face="Tahoma">The problem with method is that is returns a <font face="Courier New">DataRow[]</font>, on which you cannot perform an other select.</font></font>
</p>
<font face="Tahoma" size="2">The solution is actually quite simple : Just copy the rows you&#39;ll answer me. Yes, but you can&#39;t just reference rows in two DataTable instances, so you also have to perform a deep copy of the rows. So, with a little digging in the DataTable methods, here is what you get :</font><font face="Tahoma" size="2"><font size="2"></font></font><font face="Tahoma" size="2"><font size="2"> 
<p>
<font face="Courier New">public static DataTable Select(DataTable table, string filter, string sort)<br />
{<br />
&nbsp;&nbsp;&nbsp;DataRow[] rows = table.Select(filter, sort);<br />
&nbsp;&nbsp;&nbsp;DataTable outputTable = table.Clone();<br />
<br />
&nbsp;&nbsp;&nbsp;outputTable.BeginLoadData();<br />
<br />
&nbsp;&nbsp;&nbsp;foreach(DataRow row in rows)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;outputTable.LoadDataRow(row.ItemArray, true);<br />
<br />
&nbsp;&nbsp;&nbsp;outputTable.EndLoadData();<br />
&nbsp;&nbsp;&nbsp;return outputTable;<br />
}</font>
</p>
<p style="margin: 0cm 0cm 0pt" class="MsoNormal" align="justify">
Clone is used to copy the table schema only, BeginLoadData/EndLoadData to disable any event processing during the load operation, and LoadDataRow to effectively load each row. This seems to be a fairly fast way to copy a table&#39;s data.
</p>
</font></font>
<p align="justify">
<font face="Tahoma"><font size="2">Now, I wondered how they would do this in C# 3.0, since there is a lot of data manipulation with the new LINQ syntax. This version is quite interesting because instead of evolving the runtime, they chose to upgrade only the language by adding features that&nbsp;generate a lot of code under the hood. That was the case in C# 2.0 with iterators and&nbsp;anonymous methods. C# 1.0 also had this with</font> <font face="Courier New" size="2">foreach</font>, <font face="Courier New" size="2">using</font> <font size="2">or</font> <font face="Courier New" size="2">lock</font> <font size="2">for instance.</font></font>
</p>
<p align="justify">
<font face="Tahoma" size="2">In the particular case of Linq, C# 3.0 generates a method invocation list&nbsp; of a LINQ query,&nbsp;producing standard C# 3.0 code with the help of lambda expressions. For example, these two lines are equivalent :</font>
</p>
<font size="2">
<p>
<font face="Courier New">&nbsp;&nbsp;&nbsp;<font color="#0000ff">var</font> query = <font color="#0000ff">from</font> a <font color="#0000ff">in</font> test <font color="#0000ff">where</font> a &gt; 2 <font color="#0000ff">select</font> a;<br />
&nbsp;&nbsp;&nbsp;<font color="#0000ff">var</font> query2 = Sequence.Where(test, a =&gt; a &gt; 2);</font>
</p>
<p align="justify">
<font face="Tahoma" size="2">This ties a little more the compiler to the system asssemblies, but this does not matter anymore.</font>
</p>
</font>
<p align="justify">
<font face="Tahoma" size="2">By the way, you can apply queries to standard arrays and join them :</font>
</p>
<span style="font-size: 10pt; font-family: 'Courier New'"><font face="Tahoma"><font size="2">
<p>
<font face="Courier New">static void Main(string[] args)<br />
{<br />
&nbsp;&nbsp;&nbsp;var names = new[] {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new { Id=0, name=&quot;test&quot; },<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new { Id=1, name=&quot;test1&quot; },<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new { Id=2, name=&quot;test2&quot; },<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new { Id=4, name=&quot;test2&quot; },<br />
&nbsp;&nbsp;&nbsp;};</font>
</p>
<p>
<font face="Courier New">&nbsp;&nbsp;&nbsp;var addresses = new[] {<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new { Id=0, address=&quot;address&quot; },<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new { Id=1, address=&quot;address1&quot; },<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new { Id=2, address=&quot;address2&quot; },<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new { Id=3, address=&quot;address2&quot; },<br />
&nbsp;&nbsp;&nbsp;};<br />
<br />
&nbsp;&nbsp;&nbsp;var query = from name in names<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;join address in addresses on name.Id equals address.Id<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;orderby name.name<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;select new {name = name.name, address = address.address};<br />
<br />
&nbsp;&nbsp;&nbsp;foreach(var value in query)<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Console.WriteLine(value);<br />
}</font>
</p>
<font size="2">
<p align="justify">
I&#39;ve joined the two arrays using the Id field, and creating a new type that extracts both name and address. I really like inline querying because you can query anything that implements IEnumerable. 
</p>
<p align="justify">
I&#39;m also wondering how it&#39;ll fit into eSQL (Entity SQL)...
</p>
<p align="justify">
But back to the original subject of this post. They had to do some kind of a DataTable copy in the C# 3.0 helper library, which uses extension methods :<br />
<br />
&nbsp;&nbsp;&nbsp;<font face="Courier New">DataTableExtensions.ToDataTable&lt;T&gt;(IEnumerable&lt;T&gt;)</font><br />
<br />
And with some further digging, I found that the LoadDataRow method for copying data is the fastest way to go.
</p>
<p align="justify">
I also found out using the great reflector that there is an Expression compiler in <font face="Courier New">System.Expressions.Expression&lt;T&gt;.</font> Maybe they finally did expose an expression parser that we can use... I&#39;ll try this one too !
</p>
</font></font></font></span>

{% include imported_disclaimer.html %}
