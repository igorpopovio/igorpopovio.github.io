---
title: Artifactory, Gradle and publishing
layout: post
tags : [artifactory, gradle, maven, publish]

---

In this post I'll show you how to setup Artifactory from scratch and how to
publish a jar from gradle.

![Conveyor belt packets](/images/artifactory-gradle-publishing.jpg)


# Downloading and running

Go to [JFrog's Artifactory download
page](http://www.jfrog.com/home/v_artifactory_opensource_download) and get the
zip archive. Extract it somewhere and then start `bin/artifactory.sh` (or
    `bin/artifactory.bat` in case you're using Windows).  

You should now be able to access Artifactory at: `http://localhost:8081/artifactory`.
The default credentials are: `admin/password`.


# Creating a java project with gradle

Gradle is a build tool for java projects. It's more powerful than maven and
ant. We'll use it to create a very simple java project. But first you need to
download the latest version and add it to your `PATH`.  

After you got gradle, run these commands from the command line:

    mkdir gradle-artifactory-publishing
    cd gradle-artifactory-publishing
    gradle init --type java-library
    gradlew build

Now you should have a fully functional java project.


# Using Artifactory

After running `gradle init --type java-library` you get to use the central
maven repository. Let's change the build to use our own Artifactory.

The default (in case you don't use any custom maven server) is:

{% highlight groovy %}
    repositories {
      mavenCentral()
    }
{% endhighlight %}

We need to change this to:

{% highlight groovy %}
    repositories {
      maven { url 'http://localhost:8081/artifactory/repo' }
    }
{% endhighlight %}

In case you're wondering where does that `repo` at the end of the URL come
from: it's [the default virtual
repository](http://www.jfrog.com/confluence/display/RTF/Virtual+Repositories).

Quote from their documentation:

> Artifactory defines a default global virtual repository which effectively
aggregates all other repositories at the following URL:
`<host>:<port>/artifactory/repo`.

> By configuring Maven with this URL, any request for an artifact will go through
Artifactory which will search through all of the local and remote repositories
defined in the system.

# Publishing artifacts

Most of the times you need to publish a library to your Artifactory server.
Let's see how we can achieve this using gradle.

Apply [the new maven publishing
plugin](http://www.gradle.org/docs/current/userguide/publishing_maven.html):
`apply plugin: 'maven-publish'`.

Next configure what you want to publish and where:

{% highlight groovy %}
group = 'io.igorpopov'
version = '1.0'

publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
    repositories {
        maven {
            credentials {
              username 'admin'
              password 'password'
            }
            url "http://localhost:8081/artifactory/libs-release-local"
        }
    }
}
{% endhighlight %}

Here's the output you should get:

     ➜  ./gradlew publish
    :generatePomFileForMavenJavaPublication
    :compileJava
    :processResources UP-TO-DATE
    :classes
    :jar
    :publishMavenJavaPublicationToMavenRepository
    Uploading: io/igorpopov/gradle-artifactory-publishing/1.0/gradle-artifactory-publishing-1.0.jar to repository remote at http://localhost:8081/artifactory/libs-release-local
    Transferring 1K from remote
    Uploaded 1K
    :publish

    BUILD SUCCESSFUL

    Total time: 5.472 secs


Let's review the most important parts of the publication. You need to have the
three maven standard identifiers for a library:  
- the `groupId` (`group` in gradle)  
- the `artifactId` (`name` in gradle)  
- the `version` (same in gradle)  

The next most important is the `publishing` gradle closure that defines exactly
what we want to publish:

{% highlight groovy %}
publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
}
{% endhighlight %}

The `mavenJava(MavenPublication)` can be changed to reflect the name of your
library. You could for example use: `mylibrary(MavenPublication)`.

Next, you need to specify what files you want to publish (the `from` part).
You should probably check the [gradle documentation for publishing to
maven](http://www.gradle.org/docs/current/dsl/org.gradle.api.publish.maven.MavenPublication.html)
for full details and examples.

You also need to tell gradle **where** you want to publish your artifact. For
this we need to review what type of repositories Artifactory provides us:
local, remote and virtual.

**Local repositories** are physical, locally-managed repositories into which
**you can deploy artifacts**.

**Remote repositories** serve as a **caching proxy** for other repositories.
Artifactory is deployed with a number of pre-configured, remote repositories
which are in common use, but **you can add new repositories** based on your
needs.

**Virtual repositories** aggregate several repositories under a common URL.
The repository is virtual in that **you can resolve and retrieve artifacts from
it** but **you cannot deploy artifacts to it**.

This means that **you cannot use the default virtual repository**
(`http://localhost:8081/artifactory/repo`) to deploy artifacts (or any other
virtual repository for that matter).

For this reason **you must use one of the local repositories**:  

![Local Artifactory repositories](/images/local-artifactory-repositories.png)

For full details you can check [Artifactory's documentation on
repositories](https://www.jfrog.com/confluence/display/RTF/Configuring+Repositories).

Here's how to configure gradle to use the snapshots repository when you have
`SNAPSHOT` versions:

{% highlight groovy %}
  group = 'io.igorpopov'
  version = '1.0-SNAPSHOT'

  publishing {
    publications { ... }
    repositories {
      maven {
        credentials { ... }

        if (project.version.endsWith('-SNAPSHOT'))
          url "http://localhost:8081/artifactory/libs-snapshot-local"
        else
          url "http://localhost:8081/artifactory/libs-release-local"
      }
    }
  }
{% endhighlight %}

# Debugging

In case something goes wrong and you want to follow more closely what is
happening you can use these simple tricks:

## Gradle Info

Use `./gradlew build --info` to get the download URLs for each jar dependency.

    // the important part of the logs
    :compileJava
    Download http://localhost:8081/artifactory/repo/org/slf4j/slf4j-api/1.7.5/slf4j-api-1.7.5.pom
    Download http://localhost:8081/artifactory/repo/org/slf4j/slf4j-parent/1.7.5/slf4j-parent-1.7.5.pom
    Resource missing. [HTTP HEAD: http://localhost:8081/artifactory/repo/org/slf4j/slf4j-parent/1.7.5/slf4j-parent-1.7.5.jar]

The `Resource missing` you get above might be an issue and it can be further
investigated. These kind of issues are pretty hidden and you need to use the
`--info` flag to find them. Left unfixed, these issues can lead over time to a
greatly increased build time.

## Artifactory trace

Take the URL that gives "resource missing" from the previous step, paste it in
your browser and append: `?trace`.

    http://localhost:8081/artifactory/repo/org/slf4j/slf4j-parent/1.7.5/slf4j-parent-1.7.5.jar?trace

This will show a page with Artifactory's full log of what happens when it tries
to get that dependency: what repositories it tries first, does it find the
dependency in one of the local repositories or does it fetch it from some
remote repositories? This can be very useful when debugging why some dependency
is not found or when you have performance issues.

You can check [Artifactory's documentation on this
topic](http://www.jfrog.com/confluence/display/RTF/Artifactory+REST+API#ArtifactoryRESTAPI-TraceArtifactRetrieval).


This should be enough to get you up and running with Gradle, Artifactory and
publishing new artifacts.


# Considerations

This blog post just shows the basics how to get started with Gradle,
     Artifactory and publishing. When taken into production use you obviously
     need to change the default users and to install Artifactory as a service
     (not just start it from the command line like I did in this blog post).
     You also need to define the security permissions and inclusion and
     exclusions rules for artifact repositories.

You should check Artifactory's documentation to know exactly what to configure
and where to be careful regarding the security.

# Resources

You can find the full example project on GitHub and the documentation I used to
learn these things.

- [Full example on Github](https://github.com/igorpopovio/gradle-artifactory-publishing)
- [Configuring repositories](https://www.jfrog.com/confluence/display/RTF/Configuring+Repositories)
- [What are virtual repositories?](http://www.jfrog.com/confluence/display/RTF/Virtual+Repositories)
- [Publishing artifacts from Gradle](http://www.gradle.org/docs/current/userguide/publishing_maven.html)
- [More details on how to publish artifacts](http://www.gradle.org/docs/current/dsl/org.gradle.api.publish.maven.MavenPublication.html)

