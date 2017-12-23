---
layout: post
title: "[VS2010] On the Impacts of Debugging with “Just My Code”"
date: 2010-07-05 19:58:00 -0400
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2010/07/05/VS2010-On-the-Impacts-of-Debugging-with-Just-My-Code", "/post/2010/07/05/vs2010-on-the-impacts-of-debugging-with-just-my-code"]
author: jay
---
<!-- more -->
<p><em><a href="http://blogs.developpeur.org/jay/archive/2010/07/05/vs2010-a-propos-de-just-my-code-et-de-son-influence-sur-le-debugger.aspx">Cet article est disponible en francais.</a></em></p>
<p>The &ldquo;<a href="http://msdn.microsoft.com/en-us/library/h5e30exc.aspx" target="_blank">Just My Code</a>&rdquo; feature has been there for a while in Visual Studio. Since Visual Studio 2005 actually. And it's fairly easy to miss its details...</p>
<p>High level, this feature only shows you the stack that contains your code, mostly those assemblies that are in debug mode and have debugging symbols (pdb files). Most of the time, this is interesting, particularly if you&rsquo;re debugging fairly simple code.</p>
<p>But if you&rsquo;re debugging somehow complex issues, where you want to intercept exceptions that may be rethrown in some parts of the code that are not &ldquo;Just Your Code&rdquo;, then you have to disable it.</p>
<p>If you&rsquo;re an experienced .NET developer, chances are you disabled it because it annoyed you at some point. I did, until a while back.</p>
<p>&nbsp;</p>
<h2>Debugger Exception Handling</h2>
<p>The &ldquo;Just my Code&rdquo; (I&rsquo;ll call it JMC for the rest of the article) feature changes a few things in the way the debugger handles exceptions.</p>
<p>If it is enabled, you&rsquo;ll notice two columns in the &ldquo;Debug / Exceptions&rdquo; menu :</p>
<ul>
<li>Thrown, which means that if you check that box, the debugger will break on the least deep rethrow in the stack of the exception </li>
<li>User-unhandled, which means that if you check that box the debugger will break if the exception has not been handled by any user code exception handler in the current stack. </li>
</ul>
<p>&nbsp;</p>
<p>If it is not enabled, then the same dialog box will display one column :</p>
<ul>
<li>Thrown, which means that the debugger will break as soon as the exception is thrown</li>
</ul>
<p>&nbsp;</p>
<p>You&rsquo;ll probably notice a big difference in the way the debugger handles the &ldquo;Thrown&rdquo; option. To be a bit more clear about that difference, let&rsquo;s consider this code sample :</p>
<pre class="brush: c-sharp">    static void Main(string[] args) 
    { 
        try 
        { 
            var t = new Class1(); 
            t.Throw(); 
        } 
        catch (Exception e) 
        { 
            Console.WriteLine(e); 
        } 
      }
    </pre>
<p><em>Main executable, in debug configuration with </em><a href="http://msdn.microsoft.com/en-us/library/s4wcexbc(VS.90).aspx" target="_blank"><em>debugging symbols enabled</em></a></p>
<pre class="brush: c-sharp">    public class Class1 
    { 
        public void Throw() 
        { 
            try 
            { 
                Throw2(); 
            } 
            catch (Exception e) 
            { 
                throw; 
            } 
        }
        private void Throw2() 
        { 
            throw new InvalidOperationException("Test"); 
        } 
      }
</pre>
<p><em>Different assembly, in debug configuration without debugging symbols.</em></p>
<p>If we execute this code with the debugger with JMC enabled and with the &ldquo;Thrown&rdquo; column check for &ldquo;InvalidOperationException&rdquo;, here is the stack trace :</p>
<pre>     NotMyCode.dll!NotMyCode.Class1.Throw() + 0x51 bytes
  &gt; MyCode.exe!MyCode.Program.Main(string[] args = {string[0]}) Line 15 + 0xb bytes
</pre>
<p>&nbsp;</p>
<p>And here is the stack trace without the JMC feature :</p>
<pre>     NotMyCode.dll!NotMyCode.Class1.Throw2() + 0x46 bytes<br />    NotMyCode.dll!NotMyCode.Class1.Throw() + 0x3d bytes<br />  &gt; MyCode.exe!MyCode.Program.Main(string[] args = {string[0]}) Line 15 + 0xb bytes<br />  </pre>
<p>&nbsp;</p>
<p>You&rsquo;ll notice the impact of the &ldquo;least deep in the stack rethrow&rdquo;, which means that if you enable JMC, you will not have the original location of the exception.</p>
<p>Then you may wonder why it may be interesting to have the original location of the exception in the debugger. It is a debugging technique that is commonly used to find tricky issues that throw exceptions deep in code you do not own, and one of these exceptions is often <a href="http://msdn.microsoft.com/en-us/library/system.typeinitializationexception.aspx" target="_blank">TypeInitializerException</a>. It can be useful to break at the original location to have the proper context, or stack that lead to the exception.</p>
<p>Lately, I&rsquo;ve been using this technique of &ldquo;Break on all exceptions&rdquo; without JMC to troubleshoot loading of 32 bits assemblies in a 64 Bits CLR. You don&rsquo;t exactly know which exception you&rsquo;re looking for in the first place, and having JMC &ldquo;hiding&rdquo; some exceptions is not of a great help.</p>
<p>Also, to be fair, a more deep and intense debugging often leads to the use of <a href="http://www.microsoft.com/whdc/devtools/debugging/default.mspx" target="_blank">WinDBG</a> and the <a href="http://msdn.microsoft.com/en-us/library/bb190764.aspx" target="_blank">SOS extension</a> (and here is a <a href="http://geekswithblogs.net/.netonmymind/archive/2006/03/14/72262.aspx" target="_blank">good SOS cheat sheet</a>). But that&rsquo;s another topic.</p>
<p>&nbsp;</p>
<h2>Step Into &ldquo;Debugging Experience&rdquo; with JMC</h2>
<p>If you&rsquo;ve read this far, you may now ask yourself why you would ever want to enable JMC. After all, you can handle your code yourself and with enough experience, you can easily mentally ignore pieces of the stack that are not yours. Actually, the gray font used for code that does not have debugging symbols helps <strong>a lot </strong>for that.</p>
<p>Well, there&rsquo;s one example of good use of JMC : The debugger &ldquo;Step into&rdquo; feature. A very simple feature that allows step by step debugging of the software.</p>
<p>If you&rsquo;re in debugging mode, you&rsquo;ll step into the code that is called on the next line, if that&rsquo;s possible, and see what&rsquo;s in there.</p>
<p>So demonstrate this, let&rsquo;s consider this example :</p>
<pre class="brush: c-sharp">    static void Main(string[] args) 
    { 
        var myObject = new MyObject();

        Console.WriteLine(myObject); 
    }
    
    class MyObject 
    { 
        public override string ToString() 
        { 
            return "My object";
        } 
    }
      </pre>
<p>This is a very simple program that will use the fact that Console.WriteLine will call the ToString method on the object that is passed as a parameter.</p>
<p>The point of this sample is to make &ldquo;My Code&rdquo; (Main) call some of &ldquo;No My Code&rdquo; (Console.WriteLine) that will call &ldquo;My Code&rdquo; (MyObject.ToString). Easy.</p>
<p>Now if you run this sample with the debugger with JMC disabled, if you try to &ldquo;Step Into&rdquo; <a href="http://msdn.microsoft.com/en-us/library/swx4tc5e.aspx" target="_blank">Console.WriteLine</a>, you&rsquo;ll actually step over. This is not very helpful from the point of view of debugging you own code.</p>
<p>A very concrete example of that lack of &ldquo;Step Into&rdquo; can be found when you have proxies like the ones found in <a href="http://www.springframework.net/">Spring.NET</a> or <a href="http://www.castleproject.org/dynamicproxy/index.html">Castle's DynamicProxy</a>, they get in the way of simple debugging. You can&rsquo;t step into objects that have been proxied to perform some <a href="http://www.springframework.net/doc-latest/reference/html/aop.html">AOP</a>, for instance.</p>
<p>But if you enable JMC, well, you can actually &ldquo;Step Into&rdquo; your own code, even if the next actual method when you step into was not one of yours.</p>
<p>&nbsp;</p>
<h2>Final Words<br /></h2>
<p>Using JMC in this context is very useful and natural I would say. And the feature has been there for so long I missed its original goals. It originally got into my way for deep debugging purposes, and I dismissed as a &ldquo;junior&rdquo; feature, even cosmetic. Well, I was wrong&hellip;</p>
<p>Anyway, in Visual Studion 2010, the JMC has been improved a bit, as the way to enable and disable it is now far more easier to reach because it is now in the IntelliTrace &ldquo;Show Calls View&rdquo;.</p>
<p>Time to switch to Visual Studio 2010, people ! :)</p>
{% include imported_disclaimer.html %}
