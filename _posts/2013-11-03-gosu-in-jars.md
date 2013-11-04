---
layout: post
title: Make JAR of Gosu classes using Vark build tool
categories: [programming, gosu]
tags:
 - gosu
 - vark
 - ant
 - build
 - jar
 - packaging
---

When your Gosu scripts become more complicated, you might want to split some of the classes into separate modules. In other case you might need to reuse part of the Gosu code across several applications.

One of the options is to just copy reused sources across all applications that rely on them, which quickly leads to maintenance nightmare.

Alternative and elegant solution is to package the group of the Gosu sources as JAR file and refer to them from Gosu script just as you would do with plain Java JARs. I didn't find any guide on how to do that, so here is short explanation on how to achieve that.

Prerequisites:

- Installed Java SDK 1.7+, my version was 1.7.0_17
- Installed latest version of [Gosu](http://gosu-lang.org/) language, my version was 0.10.2
- Installed latest version of [Aardvark](http://vark.github.io/) - Gosu build tool, my version was 0.4
- Installed Ant - Java build tool, my version was 1.7.1

Steps to create Vark build file that will make JAR with your Gosu sources:

1. Create a separate project directory for the group of sources you want to reuse across several scripts or applications, ie. /mylib
1. Put your Gosu files to be packaged in JAR into one directory, ie. /mylib/src
1. Create Vark build file /mylib/build.vark containing following instructions:

<pre>
<code class="gosu">
var libName = "mylibrary"
var libVersion = "0.1"

var srcDir = file("src")
var distDir = file("dist")

function dist() {
 Ant.mkdir(:dir = distDir)
 Ant.jar(
 :destfile = distDir.file("${libName}-${libVersion}.jar"),
 :basedir = srcDir
 )
}

function clean() {
 Ant.delete(:dir = distDir)
}
</code>
</pre>

Now you should be able to call Vark from /mylib directory:
 - vark dist - will create /mylib/dist directory with mylibrary-0.1.jar file
 - vark clean - will delete /mylib/dist directory

 To use your newly created JAR in Gosu apps just put it on classpath of your Gosu application.

> **WARNING:** Gosu files were not compiled by our Vark script , so you won't be able to use Gosu classes directly from Java code.

If you want to refer to mylib.jar from any Gosu program (.gsp), just place mylibrary-0.1.jar (or however you named it) in /lib directory of the program and don't forget to point to it in your classpath declaration.

See top section of an example program myprogram.gsp using JAR with your Gosu classes below.

<pre>
<code class="gosu">
classpath "lib/mylibrary-0.1.jar"

uses mylibrary.*

// etc...
</code>
</pre>
