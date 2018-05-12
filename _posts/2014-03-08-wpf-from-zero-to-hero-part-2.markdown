---
title: WPF - From Zero to Hero - Part II
layout: post
tags : [WPF, Visual Studio, C#]

---

In the previous post I just looked at some examples and started banging code. I
learned quite a lot of stuff. Now I'll learn how to use the MVVM pattern. This
pattern was first introduced by Martin Fowler and then adopted by Microsoft as
part of the WPF framework.

This pattern is used to separate the business logic from the gui logic. So in
this case you'll have a data model, which deals only with the business logic,
     and a view model, which deals only with display logic, for example
     formatting and getting the data from the data model into the user
     interface.


## INotifyPropertyChanged

The most important things are using data binding and implementing the
`INotifyPropertyChanged` interface or using the `ObservableCollection`.

I saw that Telerik, in their examples, have extracted this in a separate class
called `ViewModelBase`. This class is only available if you're using Telerik
which doesn't apply in this case. I thought that I would extract my own, but I
didn't quite like the name. It's not very consistent with what WPF offers. So,
  we have an `ObservableCollection` already available in WPF... what would be a
  proper name for just one object? It still needs to be "observable"... Maybe
  `ObservableObject`? Actually I was quite surprised it's not already provided
  by WPF...

Here's the code:

{% highlight csharp %}
    public class ObservableObject : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged = delegate { };
        public void RaiseOnPropertyChanged(string propertyName)
        {
            PropertyChanged(this, new PropertyChangedEventArgs(propertyName));
        }
    }
{% endhighlight %}

## The ViewModel

The next thing is the `ViewModel` which needs to extend the `ObservableObject`:

{% highlight csharp %}
    namespace DiagnoseTry2_WithMvc
    {
        public class ViewModel : ObservableObject
        {
            private string message;
            public string Message
            {
                get { return message; }
                set
                {
                    message = value;
                    RaiseOnPropertyChanged("Message");
                }
            }
            // other code
        }
    }
{% endhighlight %}

Here's how we connect the `ViewModel` to the view:

{% highlight xml %}
    <Window 
            ...
            xmlns:src="clr-namespace:DiagnoseTry2_WithMvc"
            >

        <Window.DataContext>
            <src:ViewModel/>
        </Window.DataContext>
    </Window>
{% endhighlight %}

When someone updates the `Message` property, the subscribers (in this case the
    View: `MainWindow.xaml`) will be notified using the
`RaiseOnPropertyChanged()` method. You may wonder who sets those subscribers,
  since we didn't do it anywhere... Well, in fact we did it using data binding
  on this line:

{% highlight xml %}
    <TextBlock Text="{Binding Message}"/>
{% endhighlight %}

I won't go into details on how the `Items` are implemented since they are very
similar to the normal `Message`. The only differences are:
- `Items` is an `ObservableCollection`
- `Item` should be an `ObservableObject`
- the binding is done like this:

{% highlight xml %}
    <ListBox ItemsSource="{Binding Items}" />
{% endhighlight %}

The next step is to actually show something in the GUI. For this we just add
some items in the constructor. I know this is not the normal way, but I prefer
using simpler methods while I'm learning till I get all the moving parts.

{% highlight csharp %}
    public ViewModel()
    {
        Items = new ObservableCollection<Item>();
        Items.Add(new Item() { Name = "first item name", Description = "first item description" });
        Items.Add(new Item() { Name = "second item name", Description = "second item description" });
        Message = "There are " + items.Count + " items";
    }
{% endhighlight %}

# Resources

- [my laughable attempts on GitHub](https://github.com/sensui/DiagnoseTry2-WithMvc)
- [an extremely helpful tutorial on CodeProject by Barry Lapthorn](http://www.codeproject.com/Articles/165368/WPF-MVVM-Quick-Start-Tutorial)

