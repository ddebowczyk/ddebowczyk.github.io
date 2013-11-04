---
layout: post
title: Call Gosu code from Java
categories: [programming, gosu]
tags:
 - gosu
 - java
 - interop
 - vark
 - intellij idea
---

# Why

Java is trivial to call from [Gosu](http://gosu-lang.org/). Just put Java JARs on the Gosu classpath, declare Java class usage declaration and you are ready to go.

It is not so straightforward to do opposite, as Gosu relies on its own classloader which makes its magical tricks like [Open Type System](http://devblog.guidewire.com/2010/11/18/gosus-secret-sauce-the-open-type-system/) possible.

As Gosu's syntax is more powerful and less explicit than Java, wouldn't it be nice to be able to write at least parts of your application in it? Here is how you can do it.

> Original idea of the solution comes from: https://gist.github.com/am2605/1577272

# Solution overview

General idea is to:

- define Gosu class(es) API(s) as Java interface(s),
- implement this interface in Gosu,
- use [Gosu Reflection API](http://gosu-lang.org/doc/Gosu%20Reference%20Guide/wwhelp/wwhimpl/common/html/wwhelp.htm#href=typesystem.26.4.html&single=true) to create an instance of Gosu class.

Both Gosu provider and Java consumer will share dependency on the interface defined in Java, which may be distributed as a separate JAR file.

# How to set up IntelliJ Idea

1. Create new project - choose "Empty".
1. Project structure dialog will be displayed.
1. Add new Java module "java-api". It will contain Java code defining interface implemented by Gosu service provider.
1. Add new Gosu module "gosu-provider".
- It will contain Gosu code implementing interface defined in "java-api" module.
- Select "Dependencies" tab and use green "+" icon on the right size of dependencies list.
- Select "Module dependency" and choose "java-api" from the list.
1. Add new Java module "java-consumer".
- It will contain Java code consuming services implemented in Gosu, defined in "gosu-provider" module.
- Select "Dependencies" tab and use green "+" icon on the right size of dependencies list.
- Select "Module dependency" and choose "java-api" and "gosu-provider" from the list.

Make sure your module SDKs are pointing to:

- java-api - Java SDK (1.7 in my case)
- java-consumer, gosu-provider - Gosu SDK

> *WARNING:* I've been experiencing problems with IntelliJ Idea 12.1.6 while trying to make it run Try your luck with IJ 12.1.4. This is what I used for this example.

# Code

## ServiceApi.java

This will contain definition of interface offered by our Gosu class. This will be shared between our Gosu provider and Java consumer code.

<pre>
<code class="java">
package sample.api;

public interface ServiceApi {
    String execute();
}
</code>
</pre>

## ServiceProvider.gs

Here we implement our logic in Gosu.

<pre>
<code class="gosu">
package sample.provider

uses sample.api.ServiceApi

class ServiceProvider implements ServiceApi {
  construct() {
  }

  override function execute() : String {
    return "Hello from Gosu service"
  }
}
</code>
</pre>

## ServiceConsumer.java

And this how we will initialize Gosu and then instantiate and call Gosu class providing implementation of known interface.

<pre>
<code class="java">
package sample.consumer;

import sample.api.ServiceApi;

import gw.lang.reflect.ReflectUtil;
import gw.lang.Gosu;

import java.io.File;
import java.net.URISyntaxException;
import java.util.ArrayList;
import java.util.List;

public class ServiceConsumer {
    public ServiceConsumer() {
    }

    public String consume() {
        List<File> gosuPath = new ArrayList<File>();
        // path to libraries
        gosuPath.add(new File("c:/work/projects/sandbox/java-gosu-interop/lib/"));
        // path to gosu sources
        gosuPath.add(new File("c:/work/projects/sandbox/java-gosu-interop/gosu-provider/src/"));
        Gosu.init(gosuPath);
        ServiceApi service = (ServiceApi) ReflectUtil.construct("sample.provider.ServiceProvider");
        return service.execute();
    }

    public static void main(String[] args) throws URISyntaxException {
        ServiceConsumer consumer = new ServiceConsumer();
        System.out.println(consumer.consume());
    }
}
</code>
</pre>

You should end up with something like this:

![Project set up](http://farm8.staticflickr.com/7385/10678432704_08b99631fa_o.png)

To run your Java ServiceConsumer which executes Gosu code:

- open configurations (Run > Edit configurations),
- add new Application,
- select ServiceConsumer as main class,
- set "Use classpath of module" to "java-consumer" module,
- hit "Run".

You will find source code for this at [GitHub](https://github.com/ddebowczyk/java-gosu-interop).

Have fun.
