---
layout: default
title: Scala Environment Setup
root: "../"
---

## Overview

This page explains how to set up your environment for using Scala.

* . <span style="color: red">NEW</span>&nbsp; I made a [video](https://vimeo.com/73566617) showing how to set things up.


## Download and Install Scala

Scala has both a compiler (`scalac`) and an interactive environment called the REPL ("read-evaluate-print loop") (`scala`).

1. Go here: [scala-lang.org/downloads](http://www.scala-lang.org/downloads)
2. Download the current stable release (2.10.2 as of this writing).  
3. Install it according to the instructions.

Now you should be able to run `scala` from the command line to get into the REPL.
{% highlight text %}
$ scala
Welcome to Scala version 2.10.2 (Java HotSpot(TM) 64-Bit Server VM, Java 1.6.0_43).
Type in expressions to have them evaluated.
Type :help for more information.

scala> 
{% endhighlight %}

This provides you an interactive environment for running Scala code:
{% highlight text %}
scala> 3 + 4
res0: Int = 7
{% endhighlight %}

You will probably never call the compiler (`scalac`) directly.  See below.


## Download and Install SBT

SBT is a build tool, dependency manager, and more that makes a lot of tasks very easy.  You should use it.

Download and install most current package from [here](http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html#installing-sbt) (the TGZ package probably).  


## Creating a Scala project

### Create the project structure

1. Create a directory for your project.  For example: 
    {% highlight text %} mkdir project-name{% endhighlight %}

2. Create a file `project-name/build.sbt` that contains these exact contents:
    {% highlight text %}
import com.typesafe.sbt.SbtStartScript
    
name := "<project-name>"
    
version := "0.0.1"
    
organization := "<something>"
    
scalaVersion := "2.10.2"
    
libraryDependencies ++= Seq(
   "com.typesafe" % "scalalogging-log4j_2.10" % "1.0.1",
   "org.apache.logging.log4j" % "log4j-core" % "2.0-beta3",
   "junit" % "junit" % "4.10" % "test",
   "com.novocode" % "junit-interface" % "0.8" % "test->default"
  )
    
seq(SbtStartScript.startScriptForClassesSettings: _*)
    
SbtStartScript.stage in Compile := Unit
    
scalacOptions ++= Seq("-deprecation"){% endhighlight %}

3. Create a file `project-name/project/build.properties` that contains these exact contents:
    {% highlight text %}sbt.version=0.12.+{% endhighlight %}

4. Create a file `project-name/project/build.sbt` that contains these exact contents:
    {% highlight text %}addSbtPlugin("com.typesafe.sbt" % "sbt-start-script" % "0.9.0"){% endhighlight %}


### Add some code

Create a file `project-name/src/main/scala/mypkg/First.scala`.

{% highlight scala %}
package mypkg

object First {
  val something = "this is a string"

  def main(args: Array[String]) {
    println("Running the First application.")
  }
}
{% endhighlight %}

This defines a very simple application.

In your own code, you will create your own package structure and use it in place of `mypkg`.


### Interacting with your code

There are several ways interact with your code using SBT.  All of the examples below assume that you are in your project root directory (the directory that we called `project-name` above).

You can compile it (which will only re-compile modified portions of code):
{% highlight text %}
$ sbt compile
[...]
[success] Total time: 0 s, completed [...]
{% endhighlight %}

You can run the application (which will automatically compile portions of code that have been modified, so you don't have to separately call `compile`):
{% highlight text %}
$ sbt "run-main mypkg.First"
[...]
Running the First application.
[success] Total time: 0 s, completed [...]
{% endhighlight %}

You can interact with your code from the REPL.  Instead of just running `scala`, you'll need to start the REPL via SBT so that your code is (re-compiled and) loaded:
{% highlight text %}
$ sbt console
[...]
Welcome to Scala version 2.10.2 (Java HotSpot(TM) 64-Bit Server VM, Java 1.6.0_43).
Type in expressions to have them evaluated.
Type :help for more information.

scala> import mypkg.First
import mypkg.First

scala> First.something
res0: String = this is a string
{% endhighlight %}

You can generate a script for running you code.  This will re-compile any modified files, and create (or re-create) a script (`target/start`) that allows you to run your code without going through SBT.  This is nice since running through SBT is slow because it has to load the SBT environment and check whether recompilation is necessary.  Further, you do not need to rerun `start-script` after each re-compilation; you only need to regenerate the script if your dependencies change.
{% highlight text %}
$ sbt start-script
[...]
Running the First application.
[success] Total time: 0 s, completed [...]
$ target/start mypkg.First
Running the First application.
{% endhighlight %}


### Testing your code

Create a test file `project-name/src/test/scala/mypkg/FirstTests.scala`:

{% highlight scala %}
package mypkg

import org.junit.Assert._
import org.junit.Test

class FirstTests {

  @Test
  def testSomething {
    assertEquals("this is a string", First.something)
  }

}
{% endhighlight %}

Run the test class:
{% highlight text %}
$ sbt "test-only mypkg.FirstTests"
[...]
[info] Passed: : Total 1, Failed 0, Errors 0, Passed 1, Skipped 0
[success] Total time: 0 s, completed [...]
{% endhighlight %}

Or run all the tests in the project (assuming you had additional test classes):
{% highlight text %}
$ sbt test
[...]
[info] Passed: : Total 1, Failed 0, Errors 0, Passed 1, Skipped 0
[success] Total time: 0 s, completed [...]
{% endhighlight %}


## Pre-Configured Download

As an example, I have built a tar file of the above files as an example.  You can download it here: [newproject.tgz](newproject.tgz)

If you unpack this file, you should see this:

    $ tar zxvf newproject.tgz
    x newproject/
    x newproject/build.sbt
    x newproject/project/
    x newproject/src/
    x newproject/src/main/
    x newproject/src/main/scala/
    x newproject/src/main/scala/mypkg/
    x newproject/src/main/scala/mypkg/First.scala
    x newproject/project/build.properties
    x newproject/project/build.sbt

The directory structure is as follows:

    newproject/
        build.sbt
        project/
            build.sbt
            build.properties
        src/
            main/
                scala/
                    mypkg/
                        First.scala

From the root of the project (the `newproject` directory), you should be able to run all the sbt commands:

    $ cd newproject
    $ sbt "run-main mypkg.First"
    [info] Loading global plugins from /Users/dhg/.sbt/plugins
    [info] Loading project definition from /Users/dhg/newproject/project
    [info] Updating {file:/Users/dhg/newproject/project/}default-178c5b...
    [info] Resolving org.scala-sbt#precompiled-2_10_1;0.12.4 ...
    [info] Done updating.
    [info] Set current project to <project-name> (in build file:/Users/dhg/newproject/)
    [info] Updating {file:/Users/dhg/newproject/}default-bd5345...
    [info] Resolving org.scala-tools.testing#test-interface;0.5 ...
    [info] Done updating.
    [info] Compiling 1 Scala source to /Users/dhg/newproject/target/scala-2.10/classes...
    [info] Running mypkg.First
    Running the First application.
    [success] Total time: 2 s, completed Aug 30, 2013 4:10:50 PM

When you run an SBT command for the first time, you will see that it downloads all the dependencies that are used by the project.  It will keep these files in `~/.ivy2` so that it will not have to download them again.


## IDEs

You can use any text editor you want to develop your Scala code.  Use whatever you are most comfortable with.

If you want to use Eclipse, here are some instructions:

### Eclipse Scala-IDE

The Scala developers have created an official plugin for Eclipse that has a lot of nice features.  This is what I use for Scala development.  If you are interested in using this, read on.  If you don't want to use it, that's fine too.

1. Download Eclipse 3.7 (Indigo) package called "Eclipse IDE for Java Developers": [eclipse.org/downloads/packages/release/indigo/sr2](http://www.eclipse.org/downloads/packages/release/indigo/sr2) (the plugin doesn't work so well with the newest version of Eclipse).  Install it.
2. Edit the file `eclipse.ini` in the installation directory (or in the application bundle if you are on a Mac) to increase the amount memory allocated to Eclipse.  Choose an decent amount.
    {% highlight text %}-Xmx4096m{% endhighlight %}
3. Start Eclipse
4. Go to `Help` `->` `Install New Software...` and add the following site (as seen here: [scala-ide.org/download/current.html](http://scala-ide.org/download/current.html)).  Then hit `Next` and follow the instruction to install the plugin.
    {% highlight text %}http://download.scala-ide.org/sdk/e37/scala210/stable/site{% endhighlight %}
5. Install the SBT plugin for generating Eclipse projects. Add the following to `~/.sbt/plugins/plugins.sbt`:
    {% highlight text %}
resolvers += Classpaths.typesafeResolver
     
addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "2.1.2"){% endhighlight %}
6. In your project directory, generate an Eclipse project:
    {% highlight text %}
$ sbt "eclipse with-source=true"{% endhighlight %}
7. In Eclipse, do `File` `->` `Import...` `->` `Existing Project into Workspace` and find your project directory.  Import it.
8. IF YOUR PROJECT IS ON GITHUB.  Right-click on the project on the left side of the window.  Do `Team` `->` `Share Project` `->` `Git` and check `Use or create repository in parent folder of project` and click `Finish`.

