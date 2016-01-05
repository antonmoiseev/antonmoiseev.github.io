---
layout  : post
title   : Demystifying Ext JS Managed Event Listeners
date    : 2012-06-30
tags    : extjs javascript
comments: true
---

In Ext JS there exists the concept of managed event listeners. This topic often confuses Ext JS developers. Official documentation states that managed event listeners are those that _"are automatically removed when the Component is destroyed"_. But what does that mean? Which component is destroyed? How does this differ from unmanaged (ordinary) event listeners? Does it mean one must manually unsubscribe unmanaged event listeners when a component is destroyed? Let’s sort it out.

Consider following example. We have a text field, a label, and a button. The label listens to `change` event of the text field. When the event is fired the label’s value is set to match the value of the text field. The click on the button destroys the label. Destruction of the label as well as every change of its value are tracked to console.

We use two approaches to subscribe to the event. The first is unmanaged listener, i.e. listener attached with `on` method:

{% highlight js %}
// In the first case we use ‘on’
txt.on('change', lbl.changeHandler, lbl);
{% endhighlight %}

The second is managed listener. It is attached with `mon` method:

{% highlight js %}
// In the second case we use ‘mon’
lbl.mon(txt, 'change', lbl.changeHandler, lbl);
{% endhighlight %}

Below is the whole example (you can play with the code online [here](http://jsfiddle.net/amoiseev/VekQY/light/)):

<iframe width="100%" height="830" src="http://jsfiddle.net/amoiseev/VekQY/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Comment out one or the other while playing with the code. Type the value in the text field, watch the console, then click on the _Destroy Label_ button and type again. Watch the console again. When using `mon` there will be nothing new in the console. However, when using `on` the console will keep showing the text coming from the event handler that was defined inside the component which is already destroyed.

In other words, with `on`, despite the `destroy()` the object (function that handles the event) has not been released; it is actually alive. However, it can cause two problems: unpredictable side effects and memory leak.

Look at the `mon` again. Note that `mon` is called on the object that owns the event handler. Thereby the event subscription is stored inside the very label. So, when the label is destroyed, the subscription is as well. That is why event handler function gets released along with the component.

An object carrying `Observable` mixin automatically gains two properties: `events` and `managedListeners`. Property `events` is an object that maps events of this object to listeners registered via `on` method. Property `managedListeners` is an array of listeners registered via `mon`. All listeners - managed and unmanaged ones - are released when the object is destroyed. However if a method is registered as `on` listener of the different object, it is not released until that other object is destroyed (or the listener is explicitly removed).

Essentially, the trick is that `mon` explicitly changes the place that stores the event subscription. There is no definite rule when to use or not to use managed event listeners. If the label is never destroyed it can be safely used. But every time the owner of the event handler - individually or along with its container - may get destroyed earlier than the event source, the use of `mon` should be considered.
