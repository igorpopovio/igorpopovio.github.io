---
title: Reducing boilerplate with Fody
layout: post
tags : [wpf, inotifypropertychanged, boilerplate, fody]

---

This post is regarding reducing boilerplate code when implementing the MVVM pattern for WPF applications. One of the great things about MVVM is that you can bind properties to GUI elements and when you change one of them the other will update automatically. To get this to work you need to implement the `INotifyPropertyChanged` interface.

## The classic way

Normally you'd create an `ObservableObject` similar to this:

{% highlight csharp %}
public class ObservableObject : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged = delegate { };
    public void OnPropertyChanged(string propertyName)
    {
        PropertyChanged(this, new PropertyChangedEventArgs(propertyName));
    }
}
{% endhighlight %}

Let's say that you have an `Item` class and you want to bind it's properties to the GUI. Here's the class:  

{% highlight csharp %}
public class Item : ObservableObject
{
    public Item()
    {
        Name = "Testing Item";
        Description = "Just an item used for testing";
    }

    private string _name;
    public string Name
    {
        get { return _name; }
        set
        {
            _name = value;
            OnPropertyChanged("Name");
        }
    }

    private string _description;
    public string Description
    {
        get { return _description; }
        set
        {
            _description = value;
            OnPropertyChanged("Description");
        }
    }
}
{% endhighlight %}

And here's the GUI:  

{% highlight xml %}
<Window
        ...
        xmlns:src="clr-namespace:MyApp"
        >

    <Window.DataContext>
        <src:Item/>
    </Window.DataContext>

    <StackPanel Orientation="Vertical">
        <TextBlock Text="{Binding Name}"/>
        <TextBlock Text="{Binding Description}"/>
    </StackPanel>
</Window>
{% endhighlight %}

When you want to use binding you cannot use automatic properties so you have to provide similar implementations for all of them. Add a private field, make the getter return it and the setter call the `OnPropertyChanged` method from `ObservableObject`. It's not difficult, but **it's code that I don't want to write and I don't want to see** - because it's code that has no (business) logic in it and it's highly repetitive.

Of course, you might argue that if I don't want to see them then I could use those `#region` directives, but I feel like I'm sweeping them under the carpet. It doesn't really solve the problem. Some people (me being one of them) [recommend that you shouldn't use regions](http://programmers.stackexchange.com/a/53114/3704).

## The Fody way

[Fody](https://github.com/Fody/Fody) is a "tool for weaving .net assemblies". Probably this short intro doesn't tell you much so here's my explanation: Fody adds new attributes (based on the plugins you use) that generate (Intermediate Language) code during the build. That IL code is code that you don't see and you don't write, but does the job. It's basically eliminating boilerplate code - really similar code that you must always write that doesn't achieve much.


### How to use it

In Visual Studio add the Nuget package for `PropertyChanged.Fody`:

![Go to Nuget Package Manager](/images/fody-dot-net-nuget.png)

![Install the PropertyChanged.Fody package](/images/fody-dot-net-nuget-install.png)

After installing it just add the `[ImplementPropertyChanged]` attribute and the automatic properties. Here's the `Item` class again:

{% highlight csharp %}
[ImplementPropertyChanged]
public class Item
{
    public string Name { get; set; }
    public string Description { get; set; }

    public Item()
    {
        Name = "Testing Item";
        Description = "Just an item used for testing";
    }
}
{% endhighlight %}

I know that this isn't much, but considering that in a normal project you can have a lot more properties this is pretty nice.

**Classic way**: 10 lines for each property (braces included, spacing removed)  

{% highlight csharp %}
private string _name;
public string Name
{
    get { return _name; }
    set
    {
        _name = value;
        OnPropertyChanged("Name");
    }
}
{% endhighlight %}

**Fody way**: 1 line for each property (disregarding the attribute line, since it's once per class)  

{% highlight csharp %}
public string Name { get; set; }
{% endhighlight %}

This looks to me like a **major win: 10 times less code!**

Before Fody I found out about [PostSharp](http://www.postsharp.net/) which seems to do the same thing (regarding `INotifyPropertyChanged` - it also does a lot more), but they use the `NotifyPropertyChanged` attribute instead.

There are a lot of other cool plugins for Fody like: `Equals` and `ToString` (among many others). It's worth checking them out.

The full code that I used for this blog post is available on GitHub:  
- [without Fody (classic way)](https://github.com/igorpopovio/FodyWithPropertyChanged/tree/82d1ab934e2867705709bc3a2109c4eb9063904f)  
- [with Fody](https://github.com/igorpopovio/FodyWithPropertyChanged/tree/7768dfafb63c53b3174afc9915dd3ad69daab474)
