---
layout   : post
title    : Introducing Polymer TypeScript Starter
date     : 2016-04-11
comments : true
---

# Introducing Polymer TypeScript Starter

Polymer is a library created by Google for ... In this post I'll introduce a Polymer TypeScript starter project, which can help you to build the Polymer-based UI for Web applications.

In [our company][farata] we use [Polymer][polymer] for a couple of customer-facing applications for the insurance industry. The Polymer library really shines when you need to create beautiful UI that works great on both desktop and mobile devices. However Polymer is not an application framework as it lacks the features that every application needs, e.d. ... Polymer makes it quite challenging to build typical business applications that usually involve complex logic, frequent refactoring and high scalability requirements due to the large code bases.

[farata]: http://faratasystems.com
[polymer]: https://www.polymer-project.org

> The Polymer team is actively working on a set of Polymer elements that aim to isseriously improve the situation. One of the newly introduced features is routing with the `carbon-route` elemet that we already use in this starter project.

To address our concerns (which concerns???) we've created this starter project. It served as a good starting point for our Polymer applications, and may help with yours as well.

## Features

If I had to name only two the most important Polymer features, I would name two:

* An ability to write code in TypeScript
* Using the code that implements `web-component-shards`.

Let's briefly highlight the key ideas behind this starter project:

### 1. TypeScript

We use TypeScript throughout the project for the application code as well as for the configuration and build scripts. I won't enumerate all the TypeScript advantages over JavaScript, but it's a huge leap for us in terms of code maintainability and developer's productivity. I won't think twice the next time I have to choose between JavaScript and TypeScript a language for a Web project.

Another nice thing about using TypeScript is its incremental compilation, so whenever you make a change in the TypeScript code it usually takes milliseconds to recompile the source code regardless of the project size.

### 2. Automatic Bundling

In JavaScript there is a rich ecosystem of tools that can help building and bundling your applications for deployment. But Polymer is quite unique: it's based on the emerging Web standards such as HTML Imports that make it impossible to use most of the bundlers available. Actually I'm aware only of one tool called `vulcanize` that was developed by the Polymer team. Vulcanize can transform your application from multiple HTML, CSS and JavaScript files into a single HTML bundle. It works fine for most of the projects, but when your codebase starts grow, you need a way to split this single bundle into multiple ones and lazy load them on demand.

In the past we were trying to manually do it (do what ???) by introducing conventions and complicating the build process. It wasn't easy for the developers, hard to maintain, and error prone. Thankfully the Polymer team developed another tool called `web-component-shards` that does exactly what we were looking for - it takes a set of "entry points", turns them into bundles by recursively in-lining all the dependent HTML, JS and CSS files, and what most important, it creates _one more_ bundle that contains all the dependencies common for the given entry points.

As the result we write that code in the way that's natural for Polymer and don't worry about bundling during the development. During the build process `web-component-shards` automatically figures out how to split our code into bundles, which is great!

### 3. Simplicity

There are never-ending debates what's better - conventions or the explicit configuration. I prefer to avoid both. In fact one of the main goals of this project was to simplify the build configuration. We have experienced developers working on our projects, but since all of them come from different backgrounds, we wanted to minimize the amount of tools they need to learn to help them focus on application-specific tasks.

And while it's impossible to completely get rid of the build configuration, we are content with the result (which result???). We split the build configuration into three files, one per environment - development, testing, and production. There is a trade-off - we introduced some code duplication across these files, but on the other hand there is a low risk of you breaking the production configuration while tuning the development one. In fact the half of our project's configuration was taken from the Polymer Starter Kit (the link ???) (although the projects cannot be directly compared since they provide different set of features). Keep in mind that all  configuration files in our project are written in TypeScript, so we leverage the TypeScript's excellent refactoring features here as well.

### 4. Other

Besides major features discussed above, we also support various smaller features:

* Highly efficient minification. After `web-component-shards` generates the bundles, wepreprocess them with `html-minifier`, which crashes (???) all types of resources inside the bundles - JavaScript, HTML and CSS (works fine with CSS variables and Polymer mixins).
* Polylint that helps statically catch Polymer-specific errors in the code.
* Test configuration with `web-component-tester`.
* No third-party dependencies for the application (except Polymer and Polymer elements). Comparing to the Polymer Starter Kit we switched to `carbon-route` from `page.js`.
* HTML5 History API fallback middleware for the development server.
* Shrinkwrap npm dependencies to make sure all the developers in the team have the same set of packages and deal with the same problems.

We are currently considering the following features for the next releases of our starter project:

1. The `tslint` support should be added.
2. Conditionally load Web Components polyfills.
3. Optional Service Workers configuration.

Service Workers are not well supported by browsers yet and are rather complex to configure and work with at the moment, but they open lots of very compelling possibilities. So we are planning to the configuration a bit more advanced in order to be able optionally enable Service Workers.

## Getting Started With The Project

The source code of our starter can be found here - [Polymer TypeScript Starter][repo]. To launch the application you need to install the dependencies first with `npm install`.

All the commands that developers will manually run are exposed as npm scripts. There are five of them:

1. `npm start` - starts the development Web server that watches for the changes in the project. As soon as a change is made, it performs the required build steps (e.g. the TypeScript compilation) and then refreshes all currently opened pages in the browsers with the application.

2. `npm run lint` - runs `polylint` one the project's code. In the future versions it'll also run `tslint`.

3. `npm run test` - runs `web-component-tester` for the project.

4. `npm run build` - prepares the production version of the application. It creates a new directory `dist` and puts there all files that need to be deployed. Usually you want to run it on your continuous integration server.

5. `npm run serve:dist` - this command internally executes `npm run build` and then additionally launches a static Web server (it supports sending gzipped files as well) in the `dist` directory. Use this command when you are done with implementing a feature but before pushing the changes to the central repository. It gives you a chance to take a look at your application in the environment close to production.

[repo]: https://github.com/Farata/polymer-typescript-starter

If you have any thoughts on this starter project, please share them with us. We'd love to hear how it can be improved. If there is an interest in the project we could supplement it with the short "how to" recipes, so it'll be easier for you to get started and make your project-specific changes in the configuration.
