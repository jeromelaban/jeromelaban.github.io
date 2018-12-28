---
layout: post
title: "Writing a Xaml attached property in C++/CX to resize Images, with a Performance twist"
date: 2013-02-04 21:41:00 -0500
comments: true
category: Archive
tags: [".NET", "Windows 8"]
redirect_from: ["/post/2013/02/04/Writing-an-Xaml-attached-property-in-CppCX-to-resize-Images-with-a-Perf-twist.aspx", "/post/2013/02/04/writing-an-xaml-attached-property-in-cppcx-to-resize-images-with-a-perf-twist.aspx"]
author: jay
---
<!-- more -->
<p><em>TL;DR: Writing Xaml/C++ attached properties sometimes gives a 30% improvement over the C# version, which can be caused by the use of events. This article shows code sample for both versions.</em></p>
<p><em></em>&nbsp;</p>
<p>&nbsp;</p>
<p>Since it is possible to write a XAML application entirely in C++/CX, I decided to give a try to the performance of some simple code.</p>
<p>There is, after all, some marshaling involved when communicating from C# to native code, particularly with events.</p>
<p>&nbsp;</p>
<h2>The ImageDecodeSizeBehavior</h2>
<p>WinRT&rsquo;s <a href="http://msdn.microsoft.com/en-us/library/ms619224.aspx">BitmapImage</a> class supports, as does WPF and Silverlight, the <a href="http://msdn.microsoft.com/en-us/library/system.windows.media.imaging.bitmapimage.decodepixelwidth.aspx">DecodePixelWidth</a>&nbsp;and DecodePixelHeight properties.</p>
<p>These&nbsp;are very useful properties that forces the memory surface to store the image to fit a certain size, and avoid the waste of memory induced by large downscaled images. This is a very common performance issue for applications that display variable sized images, where the memory can grow <strong>very</strong> quickly.</p>
<p>[more]</p>
<p>So in C#, the following attached property can be written pretty easily :</p>
<pre class="brush: c-sharp">public static class ImageDecodeSizeBehavior
{
    public static void SetSource(Image image, string value)
    {
        image.SetValue(SourceProperty, value);
    }

    public static string GetSource(Image image)
    {
        return (string)image.GetValue(SourceProperty);
    }

    public static readonly DependencyProperty SourceProperty =
        DependencyProperty.Register(
            "Source",
            typeof(string), 
            typeof(ImageDecodeSizeBehavior), 
            new PropertyMetadata(null, (s, e) =&gt; OnSourceChanged(s, e))
        );

    private static void OnSourceChanged(DependencyObject s, DependencyPropertyChangedEventArgs e)
    {
        var image = (Image)s;

	    image.SetValue(SourceProperty, e.NewValue);

	    image.SizeChanged += (_, __) =&gt; RefreshBitmap(image);

	    RefreshBitmap(image);
    }

    private static void RefreshBitmap(Image image)
    {
	    var source = (String)image.GetValue(SourceProperty);

	    if(source != null)
	    {
		    var uri = new Uri(source);
		    var bitmap = new BitmapImage(uri);

		    bitmap.DecodePixelWidth = (int)image.Width;
		    bitmap.DecodePixelHeight = (int)image.Height;

		    image.Source = bitmap;
	    }
    }
}
</pre>
<p><span>The idea here is to use the current image control size as the target surface size, do be gentle with the memory. The downside is that if images need to be zoomed in, then they're going to be pixelated, unless the image is re-created.</span></p>
<p>In this sample, the hook on SizeChanged is more of a speed test, because one would not want to re-create the image every time the size changes, particularly if it changes too much (though the Rx Throttle operator may help).</p>
<p>It&rsquo;s used that way :</p>
<pre class="brush: xml">&lt;Image local:ImageDecodeSizeBehavior.Source="{Binding}" Width="50" Height="50" /&gt;</pre>
<pre class="brush: xml">&nbsp;</pre>
<h2>The same, in C++/CX</h2>
<p>With C++/CX, the code is looking very much like its C# counter part :</p>
<pre class="brush: cpp">using namespace Platform;
using namespace Windows::UI::Xaml;
using namespace Windows::UI::Xaml::Controls;
using namespace Windows::UI::Xaml::Interop;
using namespace Windows::UI::Xaml::Media::Imaging;
using namespace Windows::Foundation;
using namespace Windows::Foundation::Metadata;

public ref class ImageDecodeSizeBehavior sealed : DependencyObject
{
public:
	ImageDecodeSizeBehavior();

	static property DependencyProperty^ SourceProperty { 
		DependencyProperty^ get() { return _sourceProperty; }
	}

	static String^ ImageDecodeSizeBehavior::GetSource(Image^ image) {
		return (String^)image-&gt;GetValue(SourceProperty);
	}

	static void ImageDecodeSizeBehavior::SetSource(Image^ image, String^ source) {
		image-&gt;SetValue(SourceProperty, source);
	}

private:
	~ImageDecodeSizeBehavior();

	static DependencyProperty^ _sourceProperty;

	static void RefreshBitmap(Image^ image, String^ value);
	static void OnSourceChanged(DependencyObject^ d, DependencyPropertyChangedEventArgs^ e);
};
</pre>
<p>And the cpp file:</p>
<pre class="brush: cpp">using namespace Platform;
using namespace Windows::Foundation;
using namespace Windows::Foundation::Collections;
using namespace Windows::UI::Xaml;
using namespace Windows::UI::Xaml::Controls;
using namespace Windows::UI::Xaml::Data;
using namespace Windows::UI::Xaml::Documents;
using namespace Windows::UI::Xaml::Input;
using namespace Windows::UI::Xaml::Interop;
using namespace Windows::UI::Xaml::Media;
using namespace Windows::UI::Xaml::Media::Imaging;

DependencyProperty^ ImageDecodeSizeBehavior::_sourceProperty = 
	DependencyProperty::RegisterAttached(
		"Source", 
		TypeName(String::typeid), 
		TypeName(ImageDecodeSizeBehavior::typeid), 
		ref new PropertyMetadata(
			nullptr, 
			ref new PropertyChangedCallback(OnSourceChanged)
		)
	);

void ImageDecodeSizeBehavior::OnSourceChanged(DependencyObject^ d, DependencyPropertyChangedEventArgs^ e)
{
	auto image = (Image^)d;
	auto value = (String^)e-&gt;NewValue;

	image-&gt;SetValue(SourceProperty, value);

	auto sizeChanged = [=](Object^ sender, SizeChangedEventArgs^){
			RefreshBitmap(image, value);
	};

	image-&gt;SizeChanged += ref new SizeChangedEventHandler(sizeChanged);

	RefreshBitmap(image, value);
}


void ImageDecodeSizeBehavior::RefreshBitmap(Image^ image, String^ source)
{
	if(source == nullptr)
	{
		source = (String^)image-&gt;GetValue(SourceProperty);
	}

	if(source != nullptr)
	{
		auto uri = ref new Uri(source);
		auto bitmap = ref new BitmapImage(uri);

		bitmap-&gt;DecodePixelWidth = (int)image-&gt;Width;
		bitmap-&gt;DecodePixelHeight = (int)image-&gt;Height;

		image-&gt;Source = bitmap;
	}
}</pre>
<p>Except that it&rsquo;s a lot more verbose. Particularly the lambda syntax that uses all the available braces on the keyboard.</p>
<p>&nbsp;</p>
<h2>Performance</h2>
<p>How do the two compare, in terms of performance ? Testing this in a real control is a bit complex, because of the performance issue <a href="http://jaylee.org/post/2013/02/02/On-the-Performance-of-WinRTXaml-Template-Expansion.aspx">I outlined in this post</a>, but also because the measured time varies a lot, of about +/- 2%. This margin of error is not small enough to show the actual improvements.</p>
<p>Still, it is possible to measure in a more isolated code :</p>
<pre class="brush: c-sharp">var w = Stopwatch.StartNew();
for (int i = 0; i &lt; 1000; i++)
{
   var b = new Image();
   ImageDecodeSizeBehavior.SetSource(b, "http://google.com/images/srpr/logo3w.png");
}
var elapsed = w.Elapsed;
</pre>
<p>Which shows that, on a Surface RT, attaching to 1000 images takes 722ms in C#, and 505ms in C++/CX, which is about a 30% faster.</p>
<p>Now, the reason for this difference is the event registration.</p>
<p>Removing both event registrations sets the bar to 474ms for C++/CX and 493ms for C#, which <em>almost</em> accounts for all the complexity added by the hidden <a href="http://msdn.microsoft.com/en-us/library/system.runtime.interopservices.windowsruntime.eventregistrationtoken.aspx">EventRegistrationToken </a>and <a href="http://msdn.microsoft.com/en-us/library/hh138484.aspx">WindowsRuntimeMarshal.AddEventHandler </a>magic added by the C# compiler.</p>
<p>The simple&nbsp;+= operator added in C# is expanded to something like this, in IL :</p>
<pre class="brush: c-sharp">WindowsRuntimeMarshal.AddEventHandler(
   new Func(image.add_SizeChanged), 
   new Action(image.remove_SizeChanged), 
   delegate(object _, SizeChangedEventArgs __) {
      ImageDecodeSizeBehavior.RefreshBitmap(image);
   }
);</pre>
<p>Which is a bit of a change from a simple delegate&nbsp;registration. I'm guessing that there is a lot of work related to reference tracking and marshalling, under the hood, which may account for the loss of performance.</p>
<p>&nbsp;</p>
<h2>It is worth the investment ?</h2>
<p>We&rsquo;re talking about quite small amounts of time, yet we&rsquo;ve got 16.6ms to perform operations on the UI thread, and keep the 60fps rate to preserve smooth animations.&nbsp;So, 0.7ms may count in the balance.</p>
<p>C++/CX requires a bit more maintenance than its C# counterpart, which may add to the development cost. It also forces the creation of packages for all the available platforms, as it requires the creation of a native WinMD component.</p>
<p>I, for one, tend to favor maintainability over performance, unless it is in a very critical path.</p>
{% include imported_disclaimer.html %}
