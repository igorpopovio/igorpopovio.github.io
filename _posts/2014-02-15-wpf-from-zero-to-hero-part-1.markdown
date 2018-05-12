---
title: WPF - From Zero to Hero - Part I
layout: post
tags : [WPF, Visual Studio, C#]

---


I’d like to learn WPF to create a tiny application (something like a TODO list
    application). This is my first time ever touching Microsoft technologies.
That’s why I will document all the steps I take while learning.

Let’s begin... The first thing I need is Visual Studio. After a quick search on
google I found [Microsoft Visual Studio Express for Windows
Desktop](http://www.microsoft.com/en-us/download/details.aspx?id=40787).

Here’s the GUI I want to end up with:
![GUI sketch](/images/wpf-from-zero-to-hero-gui-sketch.png)

Pretty simple, huh? 

I created a new *Visual C# WPF Application* project and began playing a bit with
the Designer. Obviously, since this was my first attempt it didn’t look very
good. Besides, I want to add each item (line with image, small description and
    recommendations) dynamically. This means I have to create some kind of
component or a template which I can refer to from code.


After a bit of googling and looking at sample projects I was a bit disappointed
with what I found... The most relevant information I have found is in [this
stackoverflow answer](http://stackoverflow.com/a/9262452/354009) which provides
some very good links for [data
binding](http://msdn.microsoft.com/en-us/library/ms752347.aspx) and [data
templating](http://msdn.microsoft.com/en-us/library/ms742521.aspx).

# Nice looking interface

While the information I found is very good, I still don’t know how to make good
looking items. That’s why I began googling for some professional “WPF
components” and I found Telerik (a company that seems to specialize in creating
    various GUI components). After looking a bit on their site I found a [demo
to download](http://demos.telerik.com/wpf).

Playing with their demos, I noticed they have some examples for a custom
ListBox and they also have *code samples*. The ListBox I found looks somewhat
similar to what I want to do:

![Telerik Demo](/images/wpf-from-zero-to-hero-telerik-demo.png)

After checking the source code I saw that they obviously use their own
components, but I also noticed that some parts could be used in a standalone
project (without Telerik).

![Telerik ListBox](/images/wpf-from-zero-to-hero-telerik-listbox.png)

But, of course, I had to actually try and see if the components are as
independent as I thought they were. That’s why I replaced the empty Grid I got
when I created the WPF Application with the one from Telerik examples (section
    3. in the image above).

After doing so, I noticed that there were some issues (code underlined with
    blue wiggles) which are related to some resources I haven’t copied. This
and the fact that the window rendering from the upper half of Visual Studio is
quite empty and I couldn’t figure out the layout easily, made me remember a
*useful technique for debugging layouts*. The basic idea is to *add a bright
and noticeable background color* to the component you’re debugging so that you
can better notice overflows and floating issues.

![Visual Studio Telerik
Example](/images/wpf-from-zero-to-hero-telerik-visual-studio.png)

This is what I got after replacing all those wiggly underlines with the debug background colors:

![Visual Studio Debugging
Layouts](/images/wpf-from-zero-to-hero-visual-studio-debugging-layouts.png)

This kinda looks similar to one of the lines from the Telerik demo (it’s just a
    bit stretched on height as you can notice from the huge green rectangle):
  circle on left side, then a title and a small description, and then another
  image:

![Telerik ListBox item](/images/wpf-from-zero-to-hero-telerik-listbox-item.png)

Now, at least, I know I got the layout right and it works outside of Telerik.

# Code

Here's the part that creates the (future) item template:

{% highlight xml %}
    <StackPanel Orientation="Vertical" Background="Green" Margin="10">
        <TextBlock Text="{Binding Name}" FontSize="18" Background="Yellow"/>
        <TextBlock Text="{Binding Description}" FontSize="12" Background="Orange"/>
    </StackPanel>
{% endhighlight %}

This code also contains the data binding part which seems to use it's [own
little language](http://msdn.microsoft.com/en-us/library/ms752300.aspx).

I found that [these how-to
articles](http://msdn.microsoft.com/en-us/library/ms752039.aspx) and in
particular [this one about how to make data available for binding in
XAML](http://msdn.microsoft.com/en-us/library/ms748857.aspx) were quite useful
in understanding how everything fits together.

The next step is to create the data template for one item.

{% highlight xml %}
    <Window.Resources>
        <DataTemplate x:Key="itemTemplate" DataType="src:Item">
            <StackPanel Orientation="Vertical" Background="Green" Margin="10">
                <TextBlock Text="{Binding Name}" FontSize="18" Background="Yellow"/>
                <TextBlock Text="{Binding Description}" FontSize="12" Background="Orange"/>
            </StackPanel>
        </DataTemplate>
    </Window.Resources>
{% endhighlight %}

The basic idea is to put the DataTemplate in the resources part of the
application. I saw in some examples that people put these in a separate file,
  but I'll do it in the same file for simplicity (and this is just a learning
      project anyway).

{% highlight xml %}
    <DataTemplate x:Key="itemTemplate" DataType="src:Item">
{% endhighlight %}

The `x:key` part is used to give an identifier for the DataTemplate so you can
later reference it and the `DataType` sets the object types that can be
represented using this template.

This is where I stumbled I bit since there were different examples on the
Internet: `src:Something` or `local:Something` or even `c:Something`. I
couldn't understand how and where are those defined.

I figured it out in the end after a bit of swearing. It looks like you should
define it using *XML namespaces* (the *xmlns* part) like this:
`xmlns:src="clr-namespace:DiagnoseTry1"`. Only after you do this you can
reference classes inside that namespace using this syntax: `src:Item`.

Here are all the puzzle pieces:

{% highlight xml %}
   <!-- MainWindow.xaml -->
   <Window x:Class="DiagnoseTry1.MainWindow"
          xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
          xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
          xmlns:src="clr-namespace:DiagnoseTry1"
          Title="MainWindow" Height="350" Width="525">

      <!-- other code -->

  </Window>
{% endhighlight %}

{% highlight csharp %}
    // Item.cs
    namespace DiagnoseTry1
    {
        public class Item
        {
          // more code
        }
    }
{% endhighlight %}

That's why the `DataType="src:Item"` part works:

{% highlight xml %}
    <DataTemplate x:Key="itemTemplate" DataType="src:Item">
{% endhighlight %}

Oh, by the way, in case you're wondering: *CLR* is the [Common Language
Runtime](https://en.wikipedia.org/wiki/Common_Language_Runtime), some kind of a
virtual machine used to run .NET programs and it's similar to the JVM (Java
    Virtual Machine).

So, probably I can extract a *good practice* here: to use the same XML
namespace as CLR namespace. That means in my case I would need to use
`xmlns:DiagnoseTry1="clr-namespace:DiagnoseTry1"` (besides coming up with
    better names for namespaces; at least for this project it doesn't matter
    this much since I'm just learning).

And now here's the part that uses the `DataTemplate`:
`ItemTemplate="{StaticResource itemTemplate}"`:

{% highlight xml %}
    <ListBox x:Name="ItemListBox" Width="400" Margin="10"
             ItemsSource="{Binding items}"
             ItemTemplate="{StaticResource itemTemplate}">
    </ListBox>
{% endhighlight %}

Now follows the part where I couldn't find more walls where to bang my head.
The binding to a list of items is simply done in XAML, just:
`ItemsSource="{Binding items}"`. It's just that the binding doesn't work
without a *DataContext*. What follows is a *hack* due to not having enough
knowledge to do it the right way in this post.

{% highlight csharp %}
    public partial class MainWindow : Window
    {
        public List<Item> items { get; set; }

        public MainWindow()
        {
            InitializeComponent();
            items = GetItems();
            DataContext = this;
        }
    }
{% endhighlight %}

Normally the `DataContext` should be set in XAML code, but this doesn't work in
`MainWindow.xaml` (it [causes a StackOverflow exception](http://stackoverflow.com/a/15205381/354009)):

{% highlight xml %}
    <Window.DataContext>
        <src:MainWindow/>
    </Window.DataContext>
{% endhighlight %}

If I set the `DataContext` from the C# code then it works:

{% highlight csharp %}
    DataContext = this;
{% endhighlight %}

After reading the explanation from the above mentioned StackOverflow post I
understood that I have to use the MVVM pattern. So, the next part will focus on
the [MVVM pattern](https://en.wikipedia.org/wiki/Model_View_ViewModel) and how
to use it in WPF.

# Result for the first part

I know that this is pretty *laughable*, but this is what I got working for this first part:

![First attempt](/images/wpf-from-zero-to-hero-first-attempt.png)

# Tips

Use *CTRL + E, D* to format the XAML and C# code.

# Resources

- [my laughable attempts on GitHub](https://github.com/sensui/DiagnoseTry1)
- [Microsoft Visual Studio Express for Windows Desktop](http://www.microsoft.com/en-us/download/details.aspx?id=40787)
- [the site I used to create the gui sketch](https://gomockingbird.com/mockingbird/)
- [the StackOverflow answer which gave me the push in the right direction](http://stackoverflow.com/a/9262452/354009)
- [Microsoft Data Binding Documentation](http://msdn.microsoft.com/en-us/library/ms752347.aspx)
- [Microsoft Data Templating Documentation](http://msdn.microsoft.com/en-us/library/ms742521.aspx)
- [Telerik Demo I used as example](http://demos.telerik.com/wpf/)
- [.gitignore for all languages and tools](https://github.com/github/gitignore)
- [.gitignore for Visual Studio](https://github.com/github/gitignore/blob/master/VisualStudio.gitignore)
- [WPF Grid tutorial](http://wpftutorial.net/GridLayout.html)
- [Solution and Project Basics](http://msdn.microsoft.com/en-us/library/b142f8e7.aspx)
- [WPF video tutorial](http://www.youtube.com/watch?v=ZCVIYDREWsk)
- [the StackOverflow answer explaining the DataContext](http://stackoverflow.com/a/5517400/354009)
- [binding fails with MainWindow (StackOverflow)](http://stackoverflow.com/a/15205381/354009)

