---
layout  : post
title   : Eclipse Integration with Hybris Groovy Console
date    : 2014-02-18
tags    : hybris groovy eclipse java
comments: true
imgdir  : /images/2014-02-18/
links   :
  plugin: http://groovy.codehaus.org/Eclipse+Plugin
  part1 : http://flexblog.faratasystems.com/2013/04/28/ecommerce-with-hybris-debugging-hybris-with-beanshell-from-eclipse-ide
---

This post explains how to set up Eclipse IDE in order to remotely execute Groovy scripts on the running Hybris instance. The post is based on and develops further the idea of the article [eCommerce with Hybris: Debugging Hybris with Beanshell from Eclipse IDE]({{ page.links.part1 }}). Make sure you are familiar with the topics explained there before continuing reading this post.

## Motivation

If you follow instructions from the [previous post]({{ page.links.part1 }}), you already know how to get the job done. But what are the areas we can improve there? Here is what I think can be done better:

1. BeanShell. Although it's fairly simple language, a lot of people are already familiar with Groovy and often it's their language of choice when it comes to the scripting on the JVM. It is also my personal preference.

2. Eclipse integration. If you draw attention to the existing approach, we actually write _Java_ code in the source file within restricted area bounded with specially-treated comments. When you run the program your script is fetched from this area and converted to the BeanShell on the fly, then it is sent to the Hybris BeanShell endpoint. While it's definitely valid and working approach, it seems a bit complex and fragile to me.

Let's work towards improving the points we highlighted.

:warning: _The approach discussed below, right now works only on unix systems, because of the unix-spceific bash script used to post Groovy scripts to Hybris._

## Adding Groovy Support

We don't want to pollute our project with debugging code, so we will create a separate project for our Groovy scripts. And since we are going to write our scripts in Groovy we need to add Groovy support to Eclipse. To teach Eclipse understand Groovy install [this]({{ page.links.plugin }}) plugin. This post doesn't cover installation instructions, the process is well explained on the plugin's [web page]({{ page.links.plugin }}), you need to install it on your own.

After successfully installing Groovy-Eclipse plugin, create new Groovy project
and add it to your workspace. The workspace should look similar to this:

<img alt="Workspace after adding Groovy project"
     title="Workspace after adding Groovy project"
     src="{{ page.imgdir }}fig-01.png"
     width="318" height="327">

Since your scripts will be executed within Hybris process that knows nothing about your Groovy project, all the scripts should have default package, i.e. source files should be located right in the `src` directory:

<img alt="Groovy scripts in the `src` directory"
     title="Groovy scripts in the `src` directory"
     src="{{ page.imgdir }}fig-02.png"
     width="269" height="118">

Despite the fact Groovy is dynamic language you'd rather use typed variables, to leverage auto-completion support. To make Hybris platform's classes visible in our scripts, add appropriate project to the Groovy project dependencies:

<img alt="Adding Hybris dependencies to Groovy project"
     title="Adding Hybris dependencies to Groovy project"
     src="{{ page.imgdir }}fig-03.png"
     width="717" height="537">

## Posting Groovy Script To Hybris

Here is the interesting part. In order to post your scripts to Hybris Groovy
endpoint we will use this bash script:

<a id="bash-script" href></a>
{% highlight bash linenos %}
#!/bin/bash

# This script posts BeanShell/Groovy script to the
# remote Hybris endpoint and parses the response.

_usage_ ()
{
  echo "Usage: ./hybris_script <path_to_script> [<hybris_url>]"
}

case $# in
  0) _usage_; exit 1 ;;
  1) SCRIPT=$1; HYBRIS_URL="http://localhost:9001" ;;
  2) SCRIPT=$1; HYBRIS_URL=$2 ;;
  *) echo "Too many arguments."; _usage_; exit 1 ;;
esac

FILE_NAME=$(basename "$SCRIPT")
SCRIPT_EXTENSION="${FILE_NAME##*.}"

# Extract extension and obtain hybris url for script processing.

echo "Script extension: $SCRIPT_EXTENSION"
if [ "$SCRIPT_EXTENSION" == "bsh" ]
then
  echo "BeanShell script obtained."
  HYBRIS_SCRIPT_URL="/console/beanshell/execute"
elif [ "$SCRIPT_EXTENSION" == "groovy" ]
then
  echo "Groovy script obtained."
  HYBRIS_SCRIPT_URL="/console/groovy/execute"
else
  echo "Not supported script type passed."
  echo "Only .bsh and .groovy allowed"
  exit 1
fi

# Post script to the remote endpoint and parse response.

FULL_URL="$HYBRIS_URL$HYBRIS_SCRIPT_URL"
echo "Posting $SCRIPT to the $FULL_URL..."

OUTPUT_TEXT=$(curl "$FULL_URL"                           \
  -k -iL -b cookies.txt -c cookies.txt -s -u admin:nimda \
  -H "Origin: $HYBRIS_URL"                               \
  -H "Content-Type: application/x-www-form-urlencoded"   \
  -H "Accept: application/json"                          \
  -H "Referer: $FULL_URL"                                \
  -H "X-Requested-With: XMLHttpRequest"                  \
  -H "Connection: keep-alive"                            \
  --data-urlencode "script@$SCRIPT"                      \
  --data "commit=false")

echo "Obtained response:"
echo -e $OUTPUT_TEXT
{% endhighlight %}

1. **Line 13:** Update URL to your project-specific value.
2. **Line 44:** `-u admin:nimda` - change admin user's credentials if your settings differ from default ones.

:warning: _Because we explicitly provide admin user's credentials along with the request, we don't need to modify Spring Security configuration to allow access to the Hybris Groovy console endpoint._

In order to integrate this script with Eclipse IDE, we need to configure it as "External Tool". Here is how to do this:

1. Open Eclipse IDE
2. In the main menu find `Run -> External Tools -> External Tools Configuration...`
command and click on it:

    <img alt="External Tools in the main menu"
         title="External Tools in the main menu"
         src="{{ page.imgdir }}fig-04.png"
         width="565" height="682">

3. `External Tools Configuration` dialog window should appear. Inside the window
do following steps:

    1. Click `Program` list item on the left panel.
    2. Click `New launch configuration` toolbar button. In the appeared panel on the right, choose `Main` tab.
    3. In the `Name` text field type descriptive name for your custom configuration, e.g. _Hybris Groovy Console_.
    4. In the `Location` field specify full path to the <<bash-script,bash script>> shown above.
    5. <a id="step-5"></a>In the `Arguments` text area specify single argument `${resource_loc}` which always refers to full path of the file currently active in the Eclipse.
    6. Click `Apply` button to save your settings. Here is how the `External Tools Configuration` window should look like after completing these steps:

        <img alt="External tools dialog window, Main tab"
             title="External tools dialog window, Main tab"
             src="{{ page.imgdir }}fig-05.png"
             width="652" height="599">

    7. Optionally you may want to disable building the entire workspace every time you launch a Groovy script. Navigate to `Build` tab and switch radio button to the `The project containing the selected resource` option:

        <img alt="External tools dialog window, Build tab"
             title="External tools dialog window, Build tab"
             src="{{ page.imgdir }}fig-06.png"
             width="652" height="599">

:information_source: _You can create several `External Tools Configurations` for different environments, e.g. test, staging, etc. Just provide URL for a specific environment as the second argument on the [step 5](#step-5)._

Now we are all set and ready to execute Groovy scripts on a live Hybris instance. Let's try a simple example. Type following code in your `GroovyMain.groovy` file:

{% highlight groovy %}
import de.hybris.platform.catalog.CatalogService
import de.hybris.platform.core.Registry

CatalogService catalogService = Registry
    .applicationContext
    .getBean("catalogService") as CatalogService

println "\nHybris catalogs:"
catalogService.allCatalogs.each {
    println "$it.id - $it.version"
}
{% endhighlight %}

As you can see full auto-completion support is available:

<img alt="Auto-completion support"
     title="Auto-completion support"
     src="{{ page.imgdir }}fig-07.png"
     width="514" height="440">

Execute `Run -> External Tools -> Hybris Groovy Console` command:

<img alt="Hybris Groovy Console command"
     title="Hybris Groovy Console command"
     src="{{ page.imgdir }}fig-08.png"
     width="567" height="104">

After running this command you should be able to see results of the script execution in the Eclipse console:

<img alt="Result of Groovy script remote execution"
     title="Result of Groovy script remote execution"
     src="{{ page.imgdir }}fig-09.png"
     width="792" height="486">
