---
layout: post
title: "WinForms, DataBinding and Updates from multiple Threads"
date: 2010-01-02 23:08:00 -0500
comments: true
category: Archive
tags: [".NET"]
redirect_from: ["/post/2010/01/02/WinForms-DataBinding-and-Updates-from-multiple-Threads.aspx", "/post/2010/01/02/winforms-databinding-and-updates-from-multiple-threads.aspx"]
author: jay
---
<!-- more -->
<p><em>Cet article est <a href="http://blogs.codes-sources.com/jay/archive/2010/01/02/WinForms-DataBinding-et-Mises-a-Jour-depuis-plusieurs-Threads-.aspx" target="_blank">disponible en francais</a>.</em></p>
<p>When one is trying to use the MVC model on the WinForms, it is possible to use the <a href="http://msdn.microsoft.com/en-us/library/system.componentmodel.inotifypropertychanged.aspx" target="_blank">INotifyPropertyChanged</a> interface to allow DataBinding between the controler and form.</p>
<p>It is then possible to write a controller like this :</p>
<pre class="csharp">    [code:c#] <br />    public class MyController : INotifyPropertyChanged<br />    {<br />        // Register a default handler to avoid having to test for null<br />        public event PropertyChangedEventHandler PropertyChanged = delegate { };<br /><br />        public void ChangeStatus()<br />        {<br />            Status = DateTime.Now.ToString();<br />        }<br /><br />        private string _status;<br /><br />        public string Status <br />        {<br />            get { return _status; }<br />            set <br />            { <br />                _status = value;<br /><br />                // Notify that the property has changed<br />                PropertyChanged(this, new PropertyChangedEventArgs("Status"));<br />            }<br />        }<br />    }<br /><br />[/code]</pre>
<p>The form is defined like this :</p>
<pre class="csharp">[code:c#]<br />    public partial class MyForm : Form<br />    {<br />        private MyController _controller = new MyController();<br /><br />        public MyForm()<br />        {<br />            InitializeComponent();<br /><br />            // Make a link between labelStatus.Text and _controller.Status<br />            labelStatus.DataBindings.Add("Text", _controller, "Status");<br />        }<br /><br />        private void buttonChangeStatus_Click(object sender, EventArgs e)<br />        {<br />            _controller.ChangeStatus();<br />        }<br /><br />    }<br /><br />[/code]<br /></pre>
<p>The form will update the &ldquo;labelStatus&rdquo; when the &ldquo;Status&rdquo; property of controller changes.</p>
<p>All of this code is executed in the main thread, where the <a href="http://msdn.microsoft.com/en-us/library/system.windows.forms.application.run.aspx" target="_blank">message pump</a> of the main form is located.</p>
<p>&nbsp;</p>
<h3>A touch of asynchronism</h3>
<p>Let&rsquo;s imagine now that the controller is going to perform some operations asynchronously, using a timer for instance.</p>
<p>We update the controller by adding this :</p>
<pre class="csharp">[code:c#]<br />        private System.Threading.Timer _timer;<br /><br />        public MyController()<br />        {<br />            _timer = new Timer(<br />                d =&gt; ChangeStatus(), <br />                null,<br />                TimeSpan.FromSeconds(1),   // Start in one second<br />                TimeSpan.FromSeconds(1)    // Every second<br />            );<br />        }<br /><br />[/code]<br /></pre>
<p>By altering the controller this way, the &ldquo;Status&rdquo; property is going to be updated regularly</p>
<p>The operation model of the <a href="http://msdn.microsoft.com/en-us/library/system.threading.timer.aspx" target="_blank">System.Threading.Timer</a> implies that the ChangeStatus method is called from a different thread than the thread that created the main form. Thus, when the code is executed, the update of the label is halted by the following exception :</p>
<p><span style="font-family: Courier New;">&nbsp;&nbsp; Cross-thread operation not valid: Control 'labelStatus' accessed from a thread other than the thread it was created on.</span></p>
<p>The solution is quite simple, the update of the UI must be performed on the main thread using <a href="http://msdn.microsoft.com/en-us/library/zyzhdc6b.aspx" target="_blank">Control.Invoke()</a>.</p>
<p>That said, in our example, it&rsquo;s the DataBinding engine that hooks on the <a href="http://msdn.microsoft.com/en-us/library/system.componentmodel.inotifypropertychanged.propertychanged.aspx" target="_blank">PropertyChanged</a> event. We must make sure that the <a href="http://msdn.microsoft.com/en-us/library/system.componentmodel.inotifypropertychanged.propertychanged.aspx" target="_blank">PropertyChanged</a> event is called &ldquo;decorated&rdquo; by a call to <a href="http://msdn.microsoft.com/en-us/library/zyzhdc6b.aspx" target="_blank">Control.Invoke()</a>.</p>
<p>We could update the controller to invoke the event on the main Thread:</p>
<pre class="csharp">[code:c#]<br />    set <br />    { <br />        _status = value;<br /><br />        // Notify that the property has changed<br />	Action action = () =&gt; PropertyChanged(this, new PropertyChangedEventArgs("Status"));<br />	_form.Invoke(action);<br />    }<br /><br />[/code]<br /></pre>
<p>But that would require the addition of WinForms depend code in the controller, which is not acceptable. Since we want to put the controller in a Unit Test, calling the <a href="http://msdn.microsoft.com/en-us/library/zyzhdc6b.aspx" target="_blank">Control.Invoke()</a> method would be problematic, as we would need a Form instance that we would not have in this context.</p>
<p>&nbsp;</p>
<h3>Delegation by Interface</h3>
<p>The idea is to delegate to the view (here the form) the responsibility of placing the call to the event on the main thread. We can do so by using an interface passed as a parameter of the controller&rsquo;s constructor. It could be an interface like this one :</p>
<pre class="csharp">[code:c#]<br />    public interface ISynchronousCall<br />    {<br />        void Invoke(Action a);<br />    }<br /><br />[/code]<br /></pre>
<p>The form would implement it:</p>
<pre class="csharp">[code:c#]<br />    void ISynchronousCall.Invoke(Action action)<br />    {<br />        // Call the provided action on the UI Thread using Control.Invoke()<br />        Invoke(action);<br />    }<br /><br />[/code]<br /></pre>
<p>We would then raise the event like this :</p>
<pre class="csharp">[code:c#]<br />    _synchronousInvoker.Invoke(<br />        () =&gt; PropertyChanged(this, new PropertyChangedEventArgs("Status"))<br />    );<br /><br />[/code]</pre>
<p>But like every efficient programmer (read lazy), we want to avoid writing an interface.</p>
<p>&nbsp;</p>
<h3>Delegation by Lambda</h3>
<p>We will try to use lambda functions to call the method Control.Invoke() method. For this, we will update the constructor of the controller, and instead of taking an interface as a parameter, we will use :</p>
<pre class="csharp">[code:c#]<br />    public MyController(Action&lt;Action&gt; synchronousInvoker)<br />    {<br />        _synchronousInvoker = synchronousInvoker;<br />        ...<br />    }<br /><br />[/code]</pre>
<p>To clarify, we give to the constructor an action that has the responsibility to call an action that is passed to it by parameter.</p>
<p>It allows to build the controller like this :</p>
<pre>[code:c#]<br />    _controller = new MyController(a =&gt; Invoke(a));<br /><br />[/code]</pre>
<p>Here, no need to implement an interface, just pass a small lambda that invokes an actions on the main thread. And it is used like this :</p>
<pre class="csharp">[code:c#]<br />    _synchronousInvoker(<br />        () =&gt; PropertyChanged(this, new PropertyChangedEventArgs("Status"))<br />    );<br /><br />[/code]</pre>
<p>This means that the lambda specified as a parameter will be called on the UI Thread, in the proper context to update the associated label.</p>
<p>The controller is still isolated from the view, but adopts anyway the behavior of the view when updating &ldquo;databound&rdquo; properties.</p>
<p>If we would have wanted to use the controller in a unit test, it would have been constructed this way :</p>
<pre class="csharp">[code:c#]<br />    _controller = new MyController(a =&gt; a());<br /><br />[/code]</pre>
<p>The passed lambda would only need to call the action directly.</p>
<p>&nbsp;</p>
<h3>Bonus: Easier writing of the notification code</h3>
<p>A drawback of using <a href="http://msdn.microsoft.com/en-us/library/system.componentmodel.inotifypropertychanged.aspx" target="_blank">INotifyPropertyChanged</a> is that it is required to write the name of the property as string. This is a problem for many reasons, mainly when using refactoring or obfuscation tools.</p>
<p>C# 3.0 brings <a href="http://msdn.microsoft.com/en-us/library/bb397951.aspx" target="_blank">expression trees</a>, a pretty interesting feature that can be used in this context. The idea is to use the expression trees to make an hypothetical &ldquo;memberof&rdquo; that would get the MemberInfo of a property, much like <a href="http://msdn.microsoft.com/en-us/library/58918ffs.aspx" target="_blank">typeof</a> gets the System.Type of a type.</p>
<p>Here is a small helper method that raises events :</p>
<pre class="csharp">[code:c#]<br />    private void InvokePropertyChanged&lt;T&gt;(Expression&lt;Func&lt;T&gt;&gt; expr)<br />    {<br />        var body = expr.Body as MemberExpression;<br /><br />        if (body != null)<br />        {<br />                PropertyChanged(this, new PropertyChangedEventArgs(body.Member.Name));<br />        }<br />    }<br /><br />[/code]</pre>
<p>A method that can be used like this :</p>
<pre class="csharp">[code:c#]<br />    _synchronousInvoker(<br />        () =&gt; InvokePropertyChanged(() =&gt; Status)<br />    );<br /><br />[/code]</pre>
<p>The &ldquo;Status&rdquo; property is used as a property in the code, not as a string. It is then easier to rename it with a refactoring tool without breaking the code logic.</p>
<p>Note that the lambda <span style="font-family: Courier New;">() =&gt; Status</span> is never called. It is only analyzed by the <span style="font-family: Courier New;">InvokePropertyChanged</span> method as being able to provide the name of a property.</p>
<p>&nbsp;</p>
<h3>The Whole Controller</h3>
<pre class="csharp">[code:c#]<br />    public class MyController : INotifyPropertyChanged<br />    {<br />        // Register a default handler to avoid having to test for null<br />        public event PropertyChangedEventHandler PropertyChanged = delegate { };<br /><br />        private System.Threading.Timer _timer;<br />        private readonly Action&lt;Action&gt; _synchronousInvoker;<br /><br />        public MyController(Action&lt;Action&gt; synchronousInvoker)<br />        {<br />            _synchronousInvoker = synchronousInvoker<br /><br />            _timer = new Timer(<br />                d =&gt; Status = DateTime.Now.ToString(),<br />                null,<br />                1000,   // Start in one second<br />                1000    // Every second<br />            );<br />        }<br /><br />        public void ChangeStatus()<br />        {<br />            Status = DateTime.Now.ToString();<br />        }<br /><br />        private string _status;<br /><br />        public string Status<br />        {<br />            get { return _status; }<br />            set<br />            {<br />                _status = value;<br /><br />                // Notify that the property has changed<br />                _synchronousInvoker(<br />                    () =&gt; InvokePropertyChanged(() =&gt; Status)<br />                );<br />            }<br />        }<br /><br />        /// &lt;summary&gt;<br />        /// Raise the PropertyChanged event for the property &ldquo;get&rdquo; specified in the expression<br />        /// &lt;/summary&gt;<br />        /// &lt;typeparam name="T"&gt;The type of the property&lt;/typeparam&gt;<br />        /// &lt;param name="expr"&gt;The expression to get the property from&lt;/param&gt;<br />        private void InvokePropertyChanged&lt;T&gt;(Expression&lt;Func&lt;T&gt;&gt; expr)<br />        {<br />            var body = expr.Body as MemberExpression;<br /><br />            if (body != null)<br />            {<br />                PropertyChanged(this, new PropertyChangedEventArgs(body.Member.Name));<br />            }<br />        }<br />    }<br /><br />[/code]</pre>
{% include imported_disclaimer.html %}
