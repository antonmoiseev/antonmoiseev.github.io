---
layout       : post
title        : "Comparing the Upgrade Process of Dart and TypeScript Apps to Angular 2 beta.0"
date         : 2015-12-22
categories   : angular dart javascript typescript
redirect_from:
  - /2015/12/22/compare-ts-and-dart-angular2-beta0-upgrades/
---

During the last 6 months I had a chance to work on JavaScript and TypeScript projects. Before this I was working on a Dart project for about a year. Since the April of 2015 my co-author and I have been writing [a book on Angular 2][book], which officially supports APIs for three different languages: JavaScript, TypeScript and Dart. I primarily use TypeScript but occasionally checkout the state of the Angular 2 for Dart.

A week ago Angular 2 hit its first beta version. Early adopters and Angular 2 enthusiasts know that the preceding week introduced 7:exclamation: alpha versions. There were a few breaking changes, the upgrading experience was quite painful (no complaints though, after all that's what alpha versions exist for). So after we finished upgrading all our [book examples][tweet] to `beta.0` I decided to take another look at Angular 2 for Dart.

I took a small application that hasn't been changed since `alpha.35` and decided to upgrade it to `beta.0`. My Angular for Dart upgrade experience drastically differed from the TypeScript one, so I want to share it with you.

## Upgrading the Dart application

I can tell you upfront that the upgrade process is trivial and includes the standard steps for upgrading any other Dart package. However I still want to show it for the readers who may not have a preliminary experience with Dart.

The sample application we use in this post is pretty minimal, you can find the source code here - [ng2-dart-beta-upgrade][repo] consists of one Dart file:

{% highlight dart %}
import 'package:angular2/angular2.dart';
import 'package:angular2/bootstrap.dart';

@Component(selector: 'my-app')
@View(template: '<h1>Hello from Angular 2!</h1>')
class AppComponent {}

main() => bootstrap(AppComponent);
{% endhighlight %}

And the _pubspec.yaml_ (a Dart equivalent for _package.json_):

{% highlight yaml %}
name: ng2_dart_upgrade

dependencies:
  angular2: 2.0.0-alpha.35
  browser: any

transformers:
- angular2:
    entry_points: web/main.dart
{% endhighlight %}

To upgrade the application from version `alpha.35` to `beta.0` you need to do two things:

1. In _pubspec.yaml_ change the version of `angular2` package: `angular2: 2.0.0-beta.0`
2. Run `pub upgrade` to obtain new version of the library (and its dependencies).

That's it! :tada: You can start the development Web server `pub serve` and explore the results of your hard work :wink:. If you have the experience upgrading JavaScript or TypeScript Angular 2 applications to `beta.0` you must be surprised how it's ridiculously straightforward for the Dart version. For those who don't have such experience let me breifly discuss the major changes and the pain points connected with them.

## Why it's so different for Dart apps?

I won’t recall all the changes that were introduced between `alpha.35` and `beta.0`, but the major ones are:

1. JavaScript and TypeScript Angular 2 applications depend on several 3-rd party packages (_zone.js_, _rxjs_, etc.) which are in the alpha state as well. These packages have been updated on the path to `beta.0`. This caused several issues, that Dart developers didn't even have to deal with since applications written in Dart do not have any 3-rd party dependencies other than `angular2` itself.

2. Some of the most frequently used Angular classes, annotations and directives were moved to different modules in order to better organize the public API. This happened partially due to the immature ES6 modules ecosystem and the lack of best practices. In contrast Dart libraries (ES6 modules analogue) exist from the very beginning of the language. There are certain [conventions][public_libs] how the public API should be exposed by the packages. Angular follows these conventions and the recent changes didn't affect much the Dart applications. Actually our minimal app didn't notice any changes at all.

3. _zone.js_ that helps Angular 2 in implementing its data binding magic is now part of the _angular2-polyfills.js_ bundle. It _must_ be the very first code executed on the page and added with the script tag. This change caused confusion and a number of issues in JavaScript applications. [Zones][zones] originated from Dart and are part of the platform so for the Dart apps it just worked out of the box.

4. _reflect-metadata.js_ is the library that implements the metadata reflection API proposed as [the ES2016 feature][reflection]. It's now part of the _angular-polyfills.js_ as well, so the code had to be updated accordingly. However in Dart [the pub transformer][pub_transformer] replaces reflective code with its static version during the build rather than using reflection at the runtime, so _reflect-metadata.js_ is not needed.

5. Angular 2 heavily relies on [Observable][rxjs_observable] objects in its API. The Observable type is also [planned for ES2016][observable]. But at the moment RxJS fills in this gap. Angular uses RxJS 5 which is a complete re-write of the previous version. It's under active development and recently hit the beta version as well. The RxJS beta introduced the breaking changes so _every_ Angular 2 application written with JavaScript or TypeScript had to be updated accordingly. However [Streams][dart_streams] (which is a special case of Observables) are backed into the Dart SDK, so you don't need RxJS. If you are in doubt whether Dart streams are sufficient enough to replace RxJS observables here it is:
    <blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">Ever wonder what native Rx would look like in a language? <a href="https://twitter.com/dart_lang">@dart_lang</a> does it well with async* <a href="https://t.co/N1quqTyRpe">https://t.co/N1quqTyRpe</a></p>&mdash; ReactiveExtensions* (@ReactiveX) <a href="https://twitter.com/ReactiveX/status/581479004319838208">March 27, 2015</a></blockquote><script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

6. Angular 2 is written in TypeScript and relies on ES2015 features, hence it requires the _es6-promise.js_ and _es6-shim.js_ polyfills at the runtime. The version constrains of these libraries have also caused some minor issues during the upgrade. But guess what? Dart apps don't need these polyfills as well. `dart2js` compiler takes care of the cross-browser compatibility issues.

7. With new Angular bundles structure there is a certain order in which scripts should be loaded. For example _zone.js_ must be loaded first, then _system.js_ module loader, then _Rx.js_ (since it uses `System.register` module format), _es6-shim.js_ and _es6-promise.js_ must be loaded before any Angular code, etc. With UMD bundles the order may vary (UMD is another JavaScript module format used by Angular as a compilation target). This is something you don't even need to think about in Dart applications, since the only script you add to the page is your entrypoint Dart file that contains the `main()` function.

## Summary

You might be annoyed with the ubiquitous Dart's mantra "batteries included", but hey, it's not [the commonplace slide]({{ page.imgdir }}theslide.jpg) from a presentation, or an abstract "Hello World" example. Angular is one of the most popular Web application frameworks in the World. Millions of lines of code are written in AngularJS, and I anticipate even more yet to be written with Angular 2. So don't underestimate this mantra.

I don't urge you to give up developing Angular applications with JavaScript or TypeScript and immediately move to the Dart Side :wink:. Dart isn't perfect and has its own issues. However the development ergonomics and overall experience is something I miss a lot in the JavaScript world. Hopefully the JavaScript ecosystem will be closing the gap overtime and the development will become more productive.

[book]:            https://bit.ly/ng2book
[dart_streams]:    https://www.dartlang.org/articles/creating-streams/
[new_router]:      https://docs.google.com/document/d/1IKZLXU9Y3zdnedj5M7LfW5HQEDf9zyVjNpk_79Rf3SQ/edit?ts=56611fd1&pref=2&pli=1
[observable]:      https://github.com/zenparsing/es-observable
[pub_transformer]: https://docs.google.com/document/d/1Oe7m96QnOrilxpH1B5o9G_PnfBGovhH-n_o7RU6LYII/edit#
[public_libs]:     https://www.dartlang.org/tools/pub/package-layout.html#public-libraries
[reflection]:      https://github.com/jonathandturner/decorators/blob/master/specs/metadata.md
[repo]:            https://github.com/antonmoiseev/ng2-dart-beta-upgrade
[rxjs_observable]: http://reactivex.io/documentation/observable.html
[tweet]:           https://twitter.com/yfain/status/679261812290887680
[zones]:           https://www.dartlang.org/articles/zones/
