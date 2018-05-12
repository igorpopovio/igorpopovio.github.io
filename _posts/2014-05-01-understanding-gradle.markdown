---
title: Understanding Gradle
layout: post
tags : [groovy, gradle, closure]

---

In this blog post I'll review a bit how Gradle works behind the scenes and how
certain parts may seem magical if you don't know Groovy. And, of course, I'll
explain how that DSL magic works.

![Gradle Nirvana](/images/understanding-gradle-nirvana.jpg)

When I started learning Gradle this was one of the major blockers in doing
something with it. If you don't understand that magic then you're limited to
the examples in the documentation which aren't that revealing and don't help
you when you have similar issues.

**Warning!** this blog post just tries to bring you closer to understanding
Gradle, but it's not an exact overview of how Gradle works.


# Gradle basics

Let's say that you have a fairly standard java project. You'll need the JUnit
library, because, well, you do test your code, riiiight? We'll add that as a
dependency for the test code.

This is how the build script would look like:

{% highlight groovy %}
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    testCompile 'junit:junit:4.11'
}
{% endhighlight %}

Basically, what's happening here is that you add the java plugin that
automatically adds some defaults for building java projects.

Then you configure the repositories that tell Gradle where to download it's
dependencies from.

After that you tell that in order to compile the tests you need an external
library that will be downloaded from the above defined repository.


# A billion Gradle questions for a total noob

Now, if you have a java background that syntax looks fairly alien. It seems
like a dedicated language for specifying builds. But how does it work? What is
`apply` or `repositories` or `dependencies`? How do I know what's available so
I can do my work? What's that curly braces thing?! It looks like a code
block... What about that `testCompile`? I don't see where it is defined... I
don't even know what it is and what I can do with it...

I had all these questions initially and although I could create a basic Gradle
script I couldn't undestand it or make it do something that wasn't described in
the Gradle user guide.


# Groovy closures and Gradle Nirvana

After reading the documentation, Gradle DSLs, the Gradle forums and lots of
Googling I achieved nirvana. That is: the Gradle nirvana.

First, those `apply`, `repositories`, and `dependencies` thingys are just
normal methods. You may ask: "Wooaah! Methods! But if they are methods then
they must belong to an object, right?" Yes, they are defined in the [Project
class](http://www.gradle.org/docs/current/dsl/org.gradle.api.Project.html).
Once you know this, more things start to get clearer. Now if there's something
you don't understand you can look it up in the `Project` class documentation:
method definition, etc.

Now, the way you call these methods may seem a bit strange. For example, what
is that curly brace thing? It's a Groovy closure! It's something like an
anonymous method that can be passed as a parameter to other methods. It's
actually implemented as an anonymous java class with just one method.

Here's how to make that `testCompile` thing - add the following code to
`build.gradle` and run it with `./gradle -q`:

{% highlight groovy %}
class DependencySpec {
    def testCompile(String libraryIdentifier) {
        println "Adding the $libraryIdentifier library..."
    }
}

def dependencies2(Closure configurationClosure) {
    def dependencySpec = new DependencySpec()
    configurationClosure.delegate = dependencySpec
    configurationClosure.resolveStrategy = Closure.DELEGATE_FIRST
    configurationClosure()
}

dependencies2 {
    testCompile 'junit:junit:4.11'
}
{% endhighlight %}

Ok, so I'll take you through all the above code, since it's quite a lot to take
at once. First, I used `dependencies2` so my code doesn't conflict with what's
already provided in Gradle and it shouldn't affect your understanding anyway
:).

Then, the `DependencySpec` class and the `dependencies2` method are present in
the `build.gradle` script, but the `dependencies` method (from Gradle) is
available in the `Project` class which is compiled and put in the class path of
the build script. So, all that's left for you to see is only the `dependencies`
method call, which seems kind of magical if you don't know how it's
implemented.

I left the supporting code in the same script so you can see and understand how
everything works.

Let's start from the `dependencies2` method call:

{% highlight groovy %}
dependencies2 {
    testCompile 'junit:junit:4.11'
}
{% endhighlight %}

It may not seem a method call, since in java you have to put parenthesis like
this: `dependencies2( /* method arguments */ )`. Well, in Groovy (which Gradle
    builds on top), the parenthesis are optional if you have at least one
argument (if there's no argument you **MUST** use them).

Now, things get confusing again: there are no arguments for the `dependencies2`
method call (or so it seems). And again, that certainly doesn't look like a
method call! How can a method **call** have a "definition code block"?!

Well, let's rewrite that section of the code without using Groovy
magic/syntactic sugar so it's a bit more clear.

{% highlight groovy %}
Closure configurationClosure = { testCompile 'junit:junit:4.11' }
dependencies2(configurationClosure)
{% endhighlight %}

Now you can clearly see that we have a method call. What about the
`testCompile`? Well, that's a method call too. Just add the paranthesis and
everything becomes clear again: `testCompile('junit:junit:4.11')`.

Now, to understand how it's possible to call the `testCompile` method even
though it's in another class let's review the first part:

{% highlight groovy %}
class DependencySpec {
    def testCompile(String libraryIdentifier) {
        println "Adding the $libraryIdentifier library..."
    }
}
{% endhighlight %}

First, you define a new Groovy class that has the code you need, `testCompile`
in our case. Then you define a new `dependencies2` method that receives a
closure as an argument.

{% highlight groovy %}
def dependencies2(Closure configurationClosure) {
    def dependencySpec = new DependencySpec()
    configurationClosure.delegate = dependencySpec
    configurationClosure.resolveStrategy = Closure.DELEGATE_FIRST
    configurationClosure()
}
{% endhighlight %}

Now you need to understand one cool thing about closures: you can "delegate"
method calls that are not defined in them to an object. In our case this object
is of type: `DependencySpec`. So this means that when you call a method in the
configuration closure it will be "forwarded" to a `DependencySpec`.

To achieve this you have to set the closure's `delegate` to our object **AND**
set the `resolveStrategy` to `Closure.DELEGATE_FIRST` so Groovy/Gradle knows
that it should first look for any undefined thingy in our own object.

The Groovy closures practically hold the (Gradle) Universe together. They are
the key to understanding Gradle and getting to a productive level with it.

You can read more about [Groovy closures](http://groovy.codehaus.org/Closures)
to get a better understanding.
