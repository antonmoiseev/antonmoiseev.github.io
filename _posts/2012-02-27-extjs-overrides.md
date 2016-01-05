---
layout  : post
title   : How and Why use Overrides in Ext JS Framework
tags    : extjs javascript
comments: true
---

Ext JS is a powerful HTML5 framework that allows you to get things done in a robust and maintainable way. It does a great job fulfilling your needs for the client-side development on the web. However, you may sometimes come across non-trivial use-cases, for example, fixing issues within the framework itself. There is no such thing as a perfect framework. Bug happens, but what I like about Ext JS is that if you'll run into a bug, you don't need to wait for the framework creators to schedule, create, test, and release a fix. You can do it yourself, and in this blog I'll talk about an important overriding mechanism that exists in Ext JS.

To override a class method in object-oriented languages like Java, you start with declaring a subclass of another class. The flexibility of the prototype-based languages like JavaScript allows you to override the method of the existing classes. Ext JS goes one step further by adding a convenient mechanism of overrides known as `Ext.override` and starting from Ext JS 4 with the configuration property `overrides`.

## A Short Recap of Ext JS Overrides

Let's consider the following real-life example. While debugging I find it handy to write the log records to know when my controllers are initialized. Instead of adding `console.log()` into the `constructor` or `init` of _each_ controller I use overrides. Here is how I prescribe Ext JS to override the behavior of the standard _Controller_ class using new Ext JS 4 overriding mechanism - [Ext.Class.override](http://docs.sencha.com/extjs/4.0.7/#!/api/Ext.Class-cfg-override) configuration:

{% highlight javascript %}
Ext.override(Ext.app.Controller, {
   init: function() {
        this.callOverridden();
        console.log(this.self.getName() + ' created');
    }
});
{% endhighlight %}

Both versions have the same effect - they override the `init` method. (As any prototype-based overriding they affect even _existing_ instances of the classes, although in case of `init()` method this is not relevant). I recommend you use the first technique as it's more compact, readable, and most importantly - safe, since `override: 'Ext.app.Controller'` ensures that `Ext.app.Controller` class is loaded by the time the override is executed.

In the classical object model you must inherit a class to override its methods, so these overrides might look weird at first glance. The internal implementation is dead simple - the function declaration gets replaced. The existing declaration is preserved in an internal variable, and you get access to it via `this.callOverriden()`.

## Managing Overrides In Your Project

As your project grows there are more and more overrides to manage and deploy in production. Prior to the introduction of `Ext.Loader` it was popular to combine all overrides into a single file, say `overrides.js` that you would reference in the `index.html` right after `ext.js`. However, this approach had several drawbacks:

1. It's hard to navigate between the overrides located in the single file.
2. The `<script>` tag doesn't guarantee the order of scripts loading. So, you can face the problem when your application is loaded and started, and only after that your overrides will come into play.

A better approach is to explicitly load your overrides by `Ext.Loader`. This allows to keep them in the nice folder structure and to control the order of execution. I like my overrides to mirror the folder structure of the patched Ext JS sources. For instance, an override of `Ext.app.Controller` will be in the `App.patch.app.Controller`, while an override of `Ext.data.Model` will be in the `App.patch.data.Model` etc.

Let's continue with overriding the Controller. Here is the modified version:

{% highlight javascript %}
Ext.define('App.patch.app.Controller', {
    override: 'Ext.app.Controller',

    init: function() {
        this.callOverridden();
        console.log(this.self.getName() + ' created');
    }
});
{% endhighlight %}

Assuming that `App.controller.Main` is my application specific controller, below is the application that pre-loads the patch. All I need to force the patch pre-load is `requires: ['App.patch.app.Controller']`. The patch will be loaded before application is instantiated, therefore before any controller, view, or store is created:

{% highlight javascript %}
Ext.Loader.setConfig({ enabled: true });

Ext.application({
    requires: ['App.patch.app.Controller'],
    name: 'App',
    controllers: ['Main']
});
{% endhighlight %}

## Combining and Versioning The Overrides

What do you do with multiple patches? Simply make the "root" patch that requires all others. Here is an example: `App.path.ExtJSPatch`:

{% highlight javascript %}
Ext.define('App.patch.ExtJSPatch', {
    requires: [
        'App.patch.app.Controller'
        // ...Other overrides
    ]
});
{% endhighlight %}

Whether you are extending Ext JS or fixing the bugs, you should revise your overrides with each new Ext JS release. To reflect the version of Ext JS that your overrides are relevant for, I suggest embedding the name of the version into the name of the "root" patch and maintaining different ones, per Ext JS version:

{% highlight javascript %}
Ext.application({
    requires: ['App.patch.ExtJS407Patch'],
...
{% endhighlight %}

Hope you'll find this helpful!
