---
title: Rocking with Jira's Script Runner
layout: post
tags : [atlassian, jira, script-runner, groovy]

---

In this post I will show you how to get up to speed with scripting in Jira. You
will learn how to develop scripts, test them and various tips and tricks on how
to develop in a productive way. I will show you a lot of examples and also
provide a separate git repository which you can clone and play with.

![Description](/images/rocking-with-jira-script-runner-logo.png)

I found out that there are **a lot of great tools** out there, but there's no
documentation on **how to make them all work equally great together**. This
post aims to change that and show you how to use Jira, Jira's API, the Script
Runner add-on, the Groovy language, the Gradle build system and the IntelliJ
IDEA IDE **the best way possible**.

## How to develop your scripts

Install *IntelliJ IDEA* - the Community Edition which is *free to use* and has
*built-in support for Groovy*.

Run `git clone https://github.com/igorpopovio/jira-scripting.git`.

Run `gradlew` or `./gradlew` (depending on your operating system: Windows or
    Linux) to generate the IntelliJ IDEA projects... this will take a while
since it will download all the required libraries from Atlassian's maven
repository.

After the previous step finishes, open the `ipr` file in IntelliJ IDEA then
open any script from the project and begin experimenting. When you finish
writing the script just copy paste it into the Script Runner Console and run
it.


## Code completion with IntelliJ IDEA

After you generate and import the IntelliJ IDEA project you should get code
completion automatically when typing a variable and `.` (dot). If the popup
doesn't show up like in the image below then you can force it with `CTRL +
SPACE`.

![Description](/images/rocking-with-jira-script-runner-code-completion.png)


## How to test your scripts

After you write your scripts you might want to test them on a test server just
to make sure you don't mess up the production server... To set up a testing
server locally you need to install the *Atlassian SDK*.

After installing it you can start an instance of Jira from the command line:  

    atlas-run-standalone --product jira

You might also want to specify which version you want to start:

    atlas-run-standalone --product jira --version 6.3.10

You might want to grab a coffee or something since this command will take a bit
of time to download everything it needs.
    
When the previous command finishes go to the URL printed in the command line
(should be `http://localhost:2990/jira` or something similar). Login with
user/password: `admin/admin` and *create a new demo project* (see the next
    section). This project will already have some issues created so you can go
on directly to the testing part.  

Install the *Script Runner* plugin from the *Atlassian Marketplace* and go to
`Administration -> Add-ons -> Script Console` (see the image below).

![Description](/images/rocking-with-jira-script-runner-where-to-paste-the-script.png)


## Creating a test project

Luckily, Atlassian made a demo project with several issues. Here's how to
create it:

![Description](/images/rocking-with-jira-script-runner-create-new-project-1.png)

![Description](/images/rocking-with-jira-script-runner-create-new-project-2.png)


## The build script

The build script is done in Gradle because it is much easier to use than other
tools: you don't even need to have it installed. You just need to have the Java
Development Kit installed which I assume you have because you installed the
Atlassian SDK which requires it.

The build script will download the `jira-core` libraries from Atlassian's maven
repository and will also generate the IntelliJ IDEA project which you'll later
use. All that you have to change in this script is the Jira version:

    com.atlassian.jira:jira-core:<REPLACE WITH YOUR JIRA VERSION>


Now run `gradlew` from the command line to generate the project and then import
it in IntelliJ.

Here is the full Gradle build script:

{% highlight groovy %}
apply plugin: 'groovy'
apply plugin: 'idea'

defaultTasks 'idea'

repositories {
    mavenCentral()
    maven { url = 'https://maven.atlassian.com/repository/public' }
}

dependencies {
    compile 'org.codehaus.groovy:groovy-all:2.3.6'
    compile 'com.atlassian.jira:jira-core:6.3.6'
}
{% endhighlight %}


## Jira's `ComponentAccessor`

To achieve anything when scripting for Jira you need some kind of manager for
the objects you want to change. Examples include (but are not limited) to:
ProjectManager, IssueManager, UserManager, PermissionManager, VersionManager,
  FieldManager, CommentManager, IssueEventManager and ServiceManager.

You can access them easily by typing `ComponentAccessor.manager` in IntelliJ
IDEA and select the one you need like in the image below:

![Description](/images/rocking-with-jira-script-runner-component-accessor-managers.png)

Please note that only some of the managers appear as properties on
`ComponentAccessor`. There are others as well which you can find about by
browsing the javadoc documentation.

Here are some examples on how to get those managers (some of them, like the
    `SearchProvider` don't even have "manager" in their names):

{% highlight groovy %}
def issueManager = ComponentAccessor.issueManager
def user = ComponentAccessor.jiraAuthenticationContext.user
def jqlQueryParser = ComponentAccessor.getComponent(JqlQueryParser.class)
def searchProvider = ComponentAccessor.getComponent(SearchProvider.class)
{% endhighlight %}

There are 2 ways of getting something. First you have some fields for the
really common stuff. For the uncommon stuff just use `getComponent`.

Also please use the Groovy way instead of classical getters. Use *just the
property name*: `ComponentAccessor.issueManager`, not
`ComponentAccessor.getIssueManager()`.

## Now, why would you need that `ComponentAccessor`?

Let's assume you need an `IssueManager` - it is the most used one: how would
you get it?

If you check the javadoc you would see that it is just an interface with 2
implementations: `DefaultIssueManager` and `MockIssueManager`. The
`MockIssueManager` is just a replacement class used in Atlassian's unit tests,
  so we're left with just one choice: the `DefaultIssueManager`.

Let's see how we can create it... here is the constructor taken from the
javadoc:

{% highlight groovy %}
DefaultIssueManager(OfBizDelegator ofBizDelegator, WorkflowManager workflowManager, NodeAssociationStore nodeAssociationStore, UserAssociationStore userAssociationStore, IssueUpdater issueUpdater, PermissionManager permissionManager, MovedIssueKeyStore movedIssueKeyStore, ProjectKeyStore projectKeyStore)
{% endhighlight %}

To create that issue manager you would first need to create all the objects
given as a parameter and before doing that you'd need to create all the objects
that those objects need and so on...

In case you're wondering how Jira manages all those objects: they use an
*Inversion of Control* container called *PicoContainer*. You can Google it for
more information...

## Finding issues using JQL

Probably the most common thing you'd want to change in Jira is an issue. To get
hold of it you need to use the *Jira Query Language* which is a kind of query
language designed for interrogating Jira about issues.

I have written a groovy method that can return a list of issues based on a
query that you give as a parameter. After obtaining the list of issues you can
change them however you want.

{% highlight groovy %}
def findIssues(String jqlQuery) {
    def issueManager = ComponentAccessor.issueManager
    def user = ComponentAccessor.jiraAuthenticationContext.user
    def jqlQueryParser = ComponentAccessor.getComponent(JqlQueryParser.class)
    def searchProvider = ComponentAccessor.getComponent(SearchProvider.class)

    def query = jqlQueryParser.parseQuery(jqlQuery)
    def results = searchProvider.search(query, user, PagerFilter.unlimitedFilter)
    results.issues.collect { issue -> issueManager.getIssueObject(issue.id) }
}

jqlQuery = "project = DEMO and text ~ 'shortcut'"
logImportantMessage "Executing query: <pre>${jqlQuery}</pre>"
def issues = findIssues(jqlQuery)
{% endhighlight %}


## I don't know JQL! What should I do?

Well, you can always learn by example: first of all search the issues in Jira,
  switch to advanced mode and copy paste the JQL query. See the screenshots
  below:  


![Description](/images/rocking-with-jira-script-runner-jql-1.png)

![Description](/images/rocking-with-jira-script-runner-jql-2.png)

![Description](/images/rocking-with-jira-script-runner-jql-3.png)

Besides the copy paste you can always check Atlassian's documentation on JQL
queries...


## Jira issue objects

IntelliJ IDEA gives you all the code completion suggestions but not all of them
are useful in Jira's case - some of the methods are internal and shouldn't be
touched (unless you know what you're doing). I recommend you stick to the
methods that have `Object` in name - these are the actual domain objects that
have all the properties easily accessible:  

![Description](/images/rocking-with-jira-script-runner-use-objects.png)

## Updating issues

Just changing the issue won't actually update it in Jira's internal database.
To do that you have multiple options. There doesn't seems to be a standard way
of doing it, so I'll just describe and provide examples for each method I
encountered.

This initial example is using the manager for labels. There are managers for
pretty much everything in Jira - you just need to get it using the
`ComponentAccessor`.	

{% highlight groovy %}
def addLabelToIssue(MutableIssue issue, String label) {
    def labelManager = ComponentAccessor.getComponent(LabelManager.class)
    def user = ComponentAccessor.jiraAuthenticationContext.user.directoryUser
    def sendEmailUpdates = false
    labelManager.addLabel(user, issue.id, label, sendEmailUpdates)
}

addLabelToIssue(issue, "productivity")
{% endhighlight %}


## Jira Bulk Change vs Script Runner

You can do almost the same thing using Jira's Bulk Change with a single
exception: you cannot *append* something to a list (of labels, of versions, of
    anything actually...). You can just overwrite the previous values which is
probably not what you expect... See the images below to get an idea...

![Description](/images/rocking-with-jira-script-runner-bulk-change-overwrites.png)

The change that appears at `12 minutes ago` was done using the Script Runner
and my example script... The change from `4 minutes ago` was done using Bulk
Change... Note the difference: when using Bulk Change you lose the previous
values!

![Description](/images/rocking-with-jira-script-runner-script-vs-bulk-change.png)


## Updating issues with `ModifiedValue`

This is an example on how to add and update the fix versions. This way of
updating issues is very coupled to the field name, much like the previous
example. In the next section I'll show a better and more general way of
updating.

{% highlight groovy %}
def addFixVersionToIssue(MutableIssue issue, Version fixVersion) {
    def oldFixVersions = issue.fixVersions
    def newFixVersions = oldFixVersions + fixVersion
    def value = new ModifiedValue(oldFixVersions, newFixVersions)

    def field = ComponentAccessor.fieldManager.getOrderableField("fixVersions")
    field.updateValue(null, issue, value, new DefaultIssueChangeHolder())

    issue.fixVersions = newFixVersions
}

def getVersion(String projectKey, String versionName) {
    ComponentAccessor.versionManager.allVersions.find { version ->
        version.projectObject.key == projectKey && version.name == versionName
    }
}

def projectKey = "DEMO"
def fixVersionName = "Version 1.0"
def fixVersion = getVersion(projectKey, fixVersionName)

addFixVersionToIssue(issue, fixVersion)
{% endhighlight %}


## Finding out dynamic fields

In some cases you'll need methods that receive field names or other similar
things. This is the case we had in the previous example:  

{% highlight groovy %}
def field = ComponentAccessor.fieldManager.getOrderableField("fixVersions")
{% endhighlight %}

How were we supposed to know what we have to use `"fixVersions"` as
parameter??? Well, you can use the Script Runner to help you find out the
`orderableFields` like this: 

![Description](/images/rocking-with-jira-script-runner-quick-documentation.png)

## Better way to update issues

This is the most general way to update something related to an issue. Normally
you should call it no matter what you update.

{% highlight groovy %}
def updateIssue(MutableIssue issue) {
    def user = ComponentAccessor.jiraAuthenticationContext.user
    def issueManager = ComponentAccessor.issueManager
    issueManager.updateIssue(user, issue, createIssueUpdateRequest())
}

def createIssueUpdateRequest() {
    new UpdateIssueRequest.UpdateIssueRequestBuilder()
            .eventDispatchOption(EventDispatchOption.DO_NOT_DISPATCH)
            .sendMail(false)
            .build()
}

updateIssue(issue)
{% endhighlight %}


## Indexing updated issues

This isn't always required, but here's how to do it anyway...

{% highlight groovy %}
def reindex(Collection<MutableIssue> issues) {
    def issueIndexManager = ComponentAccessor.issueIndexManager
    boolean wasIndexing = ImportUtils.isIndexIssues()
    ImportUtils.setIndexIssues(true)
    issues.each { issue -> issueIndexManager.reIndex(issue) }
    ImportUtils.setIndexIssues(wasIndexing)
}

reindex(issues)
{% endhighlight %}


## Providing feedback on the results

By default the logs will get buried somewhere in Jira's logs and while it is
possible to go hunt them down it would be too much hassle. Instead you can
return a string from the script and the Script Runner will show it after
running your script. I made several methods that help with the logging -
basically just appending stuff to a string and finally returning it.

{% highlight groovy %}
finalMessage = ""

def logMessage(Object message) {
    finalMessage += "${message}<br/>"
}

def logImportantMessage(Object message) {
    logMessage "<strong>${message}</strong>"
}

return finalMessage
{% endhighlight %}


## Finding the issue links

Although this is not required and doesn't actually change anything in Jira, I
prefer to have it. Basically instead of getting just a list of issues that can
be updated you get a list where you can click on any issue and get more
details. It saves me from having to copy paste the issue key from the list to
the browser's address bar.

{% highlight groovy %}
def getIssueLink(MutableIssue issue) {
    def properties = ComponentAccessor.applicationProperties
    def jiraBaseUrl = properties.getString(APKeys.JIRA_BASEURL)
    "${jiraBaseUrl}/browse/${issue.key}"
}

def formatIssue(MutableIssue issue) {
    def issueLink = getIssueLink(issue)
    def htmlLink = "<a href=\"${issueLink}\">${issue.key}</a>"
    "<strong>${htmlLink}</strong> - ${issue.summary}"
}

logImportantMessage "Found ${issues.size()} issues. Here they are:"
logIssues(issues)
{% endhighlight %}


## Script results

After running the script you should see the results in a nice format like in
the screen shot below:  

![Description](/images/rocking-with-jira-script-runner-script-result.png)

It can get even better: by using nicely styled tables from AUI (Atlassian User
    Interface):

  ![Description](/images/rocking-with-jira-script-runner-issue-table.png)

You can check the repository with examples and see how it's done.

## How can I do `this/that/whatever`?

From what I noticed, in Jira, there are quite a lot of managers and these are
available as directly accessible fields in the `ComponentAccessor` class. I
would first look if there's any manager that seems to have what I want  then
fumble up with IntelliJ IDEA's code assistance and then look it up in Jira's
javadoc. If that doesn't work you could check *Atlassian Answers* (either
    search or even ask questions) or Google.


## More resources

 - [Atlassian SDK](https://developer.atlassian.com/display/DOCS/Getting+Started)  
 - [IntelliJ IDEA](https://www.jetbrains.com/idea)  
 - [Script Runner plugin](https://marketplace.atlassian.com/plugins/com.onresolve.jira.groovy.groovyrunner)  
 - [Script Runner documentation](https://jamieechlin.atlassian.net/wiki/display/GRV/Script+Runner)  
 - [The Java API policy for Jira](https://developer.atlassian.com/display/JIRADEV/Java+API+Policy+for+JIRA)  
 - [Atlassian maven repositories](https://developer.atlassian.com/display/DOCS/Atlassian+Maven+Repositories)  
 - [Java API reference](https://developer.atlassian.com/display/JIRADEV/Java+API+Reference)  
 - [Jira Query Language - JQL](https://confluence.atlassian.com/display/JIRA/Advanced+Searching)  
 - [Atlassian Answers](https://answers.atlassian.com)  

