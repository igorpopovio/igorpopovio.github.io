---
title: Asynchronous programming in C# with WPF
layout: post
tags : [async, await, asynchronous, programming]

---

I know nothing about the Task Parallel Library and writing a blog post about this is a great opportunity to learn about it. This post won't be exhaustive: I'll just explore Microsoft's recommended way of implementing asynchronous programming.


# Way of working

I'm more focused now **only on the specifics of asynchronous programming** so I won't write the app using best practices like MVVM. For that you should check my previous posts about it.

# Starting out: the unresponsive app

The app is very simple: just 2 buttons - one for a long running task (which we are going to simulate using `Thread.Sleep`) and another, quicker one that just updates a counter. We're going to see how different approaches work.

The first approach is the dumb implementation which makes the app unresponsive while the long task executes. We're doing this just so we have something to compare to.

Here's the source code for the long task button click. As you can see, we're just simulating that it takes long with `Thread.Sleep` and then updating the interface to signal that we're done.

{% highlight csharp %}
private void LongTaskButton_Click(object sender, RoutedEventArgs e)
{
    Thread.Sleep(2000);
    LongTaskTextBlock.Text = "Completed long task";
}
{% endhighlight %}

And here's how the application looks like. While the task is executing in background the interface freezes and you can't do anything else (like clicking the other button).

![The unresponsive app](https://i.imgur.com/nCvloFl.gif)

# Introducing the Task Parallel Library

Now let's use the Task Parallel Library to put the long running task in a background thread and also make a few improvements to the user experience like disabling the button while the task executes and finally updating the interface when the task completes.

{% highlight csharp %}
private void LongTaskButton_Click(object sender, RoutedEventArgs e)
{
    LongTaskButton.IsEnabled = false;
    LongTaskTextBlock.Text = "Starting long task";

    var task = Task.Run(() =>
    {
        Thread.Sleep(2000);
    });

    task.ContinueWith((t) =>
    {
        Dispatcher.Invoke(() =>
        {
            LongTaskButton.IsEnabled = true;
            LongTaskTextBlock.Text = "Completed long task";
        });
    });
}
{% endhighlight %}

The `Task.Run` will queue the operation to run on the thread pool and will return a `Task` object.
In order to update the interface when the task is complete we're using `Task.ContinueWith`. An important thing to notice here is the use of a `Dispatcher`. WPF requires all interface updates to be done from a single thread: the UI thread, which queues work items inside an object called a `Dispatcher`.

If you try to update the interface from a different thread then the UI thread you will get an `System.InvalidOperationException`: **The calling thread cannot access this object because a different thread owns it.**

Here's how the application works now:

![Task Parallel Library](https://i.imgur.com/Ytj82d9.gif)

As you can notice, **the app no longer freezes** after the long task starts: we can still click on the second button to increase the count.

In case the task fails, we have to check if the task `IsFaulted` and if so, then show a message to the user:

{% highlight csharp %}
private void LongTaskButton_Click(object sender, RoutedEventArgs e)
{
    LongTaskButton.IsEnabled = false;
    LongTaskTextBlock.Text = "Starting long task";

    var task = Task.Run(() =>
    {
        throw new Exception("Something crashed!");
    });

    task.ContinueWith((t) =>
    {
        Dispatcher.Invoke(() =>
        {
            LongTaskButton.IsEnabled = true;
            LongTaskTextBlock.Text = t.IsFaulted ?
                t.Exception.InnerException.Message :
                "Completed long task";
        });
    });
}
{% endhighlight %}

You can also return a status result from the task which you can later access in `ContinueWith` by checking the task `Result` property: 

{% highlight csharp %}
private void LongTaskButton_Click(object sender, RoutedEventArgs e)
{
    LongTaskButton.IsEnabled = false;
    LongTaskTextBlock.Text = "Starting long task";

    var task = Task.Run(() =>
    {
        Thread.Sleep(2000);
        return "Completed long task";
    });

    task.ContinueWith((t) =>
    {
        Dispatcher.Invoke(() =>
        {
            LongTaskButton.IsEnabled = true;
            LongTaskTextBlock.Text = t.IsFaulted ?
                t.Exception.InnerException.Message :
                t.Result;
        });
    });
}
{% endhighlight %}


# Handling exceptions in tasks

Here's what Microsoft has to say about [handling exceptions in tasks](https://docs.microsoft.com/en-us/dotnet/standard/parallel-programming/task-based-asynchronous-programming#handling-exceptions-in-tasks):

> When a task throws one or more exceptions, the exceptions are wrapped in an `AggregateException` exception. That exception is propagated back to the thread that joins with the task, which is typically the thread that is waiting for the task to finish or the thread that accesses the `Result` property. This behavior serves to enforce the .NET Framework policy that all unhandled exceptions by default should terminate the process. The calling code can handle the exceptions by using any of the following in a `try/catch` block:
- The `Wait` method
- The `WaitAll` method
- The `WaitAny` method
- The `Result` property


# async and await

These keywords simplify work with Task Parallel Library since you no longer need to use explicit continuations. You simply need to `await` a long running task and the current thread will be suspended until that task completes. Everything after await will be executed when the task completes so we no longer need to call `ContinueWith` on the returned task.

{% highlight csharp %}
private async void LongTaskButton_Click(object sender, RoutedEventArgs e)
{
    LongTaskButton.IsEnabled = false;
    LongTaskTextBlock.Text = "Starting long task";

    var result = await Task.Run(() =>
    {
        Thread.Sleep(2000);
        return "Completed long task";
    });

    LongTaskButton.IsEnabled = true;
    LongTaskTextBlock.Text = result;
}
{% endhighlight %}


And here's how we handle exceptions in this case:

{% highlight csharp %}
private async void LongTaskButton_Click(object sender, RoutedEventArgs e)
{
    LongTaskButton.IsEnabled = false;
    LongTaskTextBlock.Text = "Starting long task";

    string result = "";
    try
    {
        result = await Task.Run(() =>
        {
            Thread.Sleep(2000);
            throw new Exception("Something crashed!");
            return "Completed long task";
        });
    }
    catch (Exception ex)
    {
        result = ex.Message;
    }

    LongTaskButton.IsEnabled = true;
    LongTaskTextBlock.Text = result;
}
{% endhighlight %}


# Awaiting on multiple tasks

Besides declaring each task, you need to call `Tasks.WhenAll` with all the tasks you want to `await` upon:

{% highlight csharp %}
private async void LongTaskButton_Click(object sender, RoutedEventArgs e)
{
    LongTaskButton.IsEnabled = false;
    LongTaskTextBlock.Text = "Starting long tasks";

    var firstLongTask = Task.Run(() =>
    {
        Thread.Sleep(1000);
        return "first";
    });

    var secondLongTask = Task.Run(() =>
    {
        Thread.Sleep(2000);
        return "second";
    });

    var thirdLongTask = Task.Run(() =>
    {
        Thread.Sleep(500);
        return "third";
    });

    var results = await Task.WhenAll(firstLongTask, secondLongTask, thirdLongTask);

    LongTaskButton.IsEnabled = true;
    LongTaskTextBlock.Text = "Completed " + string.Join(", ", results) + " tasks";
}
{% endhighlight %}


# Source Code

The full source code is available on [GitHub](https://github.com/igorpopovio/AsyncProgramming). You can clone the repo and try it locally. Just a note though: the examples are available all in the same file, but in different commits - you need to check the `git log`.


# Resources

- [Asynchronous programming](https://docs.microsoft.com/en-us/dotnet/csharp/async)
- [Async in depth](https://docs.microsoft.com/en-us/dotnet/standard/async-in-depth)
- [Parallel Class](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.parallel)
- [Async/await vs BackgroundWorker](https://stackoverflow.com/questions/12414601/async-await-vs-backgroundworker)
- [Asynchronous Programming Patterns](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/index)
- [Task-based Asynchronous Pattern (TAP)](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)
- [Six Essential Tips for Async from Channel9](https://channel9.msdn.com/Series/Three-Essential-Tips-for-Async)
- [Concurrency in C# Cookbook](https://www.amazon.com/Concurrency-Cookbook-Asynchronous-Multithreaded-Programming/dp/1449367569)
