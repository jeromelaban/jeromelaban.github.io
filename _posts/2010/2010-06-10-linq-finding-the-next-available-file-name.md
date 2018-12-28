---
layout: post
title: "[LINQ] Finding the next available file name"
date: 2010-06-10 20:16:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2010/06/10/LINQ-Finding-the-next-available-file-name.aspx", "/post/2010/06/10/linq-finding-the-next-available-file-name.aspx"]
author: jay
---
<!-- more -->
<p><a href="http://blogs.codes-sources.com/jay/archive/2010/06/10/linq-trouver-le-nom-de-fichier-suivant-disponible.aspx"><em>Cet article est disponible en francais.</em></a></p>
<p><em><br /></em></p>
<p>Sometimes, the most simple examples are the best.</p>
<p>&nbsp;</p>
<p>Let&rsquo;s say you have a configuration file, but you want to make a copy of it         before you modify it. Easy, you copy that file to &ldquo;filename.bak&rdquo;. But         what happens there&rsquo;s already that file ? Well, either you replace it, or you         create an autoincremented file.</p>
<p>&nbsp;</p>
<p>If you want to do the latter, you could do it using a for loop. But since you&rsquo;re         a happy functional programming guy, you want to make it using LINQ.</p>
<p>&nbsp;</p>
<p>You then can do it like this :</p>
<pre class="brush: c-sharp">&nbsp;&nbsp;&nbsp; public static string CreateNewFileName(string filePath)
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if (!File.Exists(filePath))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return filePath;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // Don't do that for each file.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var name = Path.GetFileNameWithoutExtension(filePath);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var extension = Path.GetExtension(filePath);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // Now find the next available file
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var fileQuery = from index in Enumerable.Range(2, 10000)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // Build the file name
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; let fileName = string.Format("{0} ({1}){2}", name, index, extension)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // Does it exist ?
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; where !File.Exists(fileName)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // No ? Select it.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; select fileName;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; // Return the first one.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return fileQuery.First();
&nbsp;&nbsp;&nbsp; }</pre>
<p>Note the use of the <span style="font-family: courier new,courier;">let </span>operator, which allows the reuse of what is called a &ldquo;range         variable&rdquo;. In this case, it avoids using <span style="font-family: courier new,courier;">string.Format</span> multiple times.</p>
<p>&nbsp;</p>
<h2>The case of Infinity<br /></h2>
<p>There&rsquo;s actually one problem with this implementation, which is the arbitrary         &ldquo;10000&rdquo;. This might be fine if you don&rsquo;t intend to make more than         10000 backups of your configuration file. But if you do, to lift that limit, we         could write this iterator method :</p>
<pre class="brush: c-sharp">&nbsp;&nbsp;&nbsp; public static IEnumerable&lt;int&gt; InfiniteRange(int start)
&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; while(true)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; yield return start++;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }</pre>
<p>Which basically will return an new value each time you ask for one. To use that         method you have to make sure that you have an exit condition (the file does not exist, in the previous example), or you may well be         enumerating until the end of times... Actually up to <span style="font-family: Courier New;">int.MaxValue</span>, for the <em>nit-pickers</em>, but .NET 4.0 adds <a href="http://msdn.microsoft.com/en-us/library/system.numerics.biginteger%28v=VS.100%29.aspx" target="_blank">System.Numerics.BigInteger</a> to be sure to get to the end of times. You never know.</p>
<p>&nbsp;</p>
<p>To use this iterator, just replace :</p>
<pre class="brush: c-sharp">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var fileQuery = from index in Enumerable.Range(2, 10000)</pre>
<p>by</p>
<pre class="brush: c-sharp">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var fileQuery = from index in InfiniteRange()</pre>
<p>And you&rsquo;re done.</p>
{% include imported_disclaimer.html %}
