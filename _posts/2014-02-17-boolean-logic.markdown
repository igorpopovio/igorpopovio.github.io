---
title: Boolean Logic
layout: post
tags : [C#, boolean, logic, refactoring, clean code]

---

![Less code matters](/images/boolean-logic-less-code-matters.png)

While learning WPF I encountered [this example on Microsoft's
website](http://code.msdn.microsoft.com/windowsdesktop/Data-Binding-Demo-82a17c83/sourcecode?fileId=17219&pathId=822232916).
You can either scroll down or download the C# version of the code.

It's about this code:

{% highlight csharp %}
    // MainWindow.xaml.cs
    private void ShowOnlyBargainsFilter(object sender, FilterEventArgs e)
    {
        AuctionItem product = e.Item as AuctionItem;
        if (product != null)
        {
            // Filter out products with price 25 or above
            if (product.CurrentPrice < 25)
            {
                e.Accepted = true;
            }
            else
            {
                e.Accepted = false;
            }
        }
    }
{% endhighlight %}

The entire inner if statement can be reduced to:

{% highlight csharp %}
    e.Accepted = product.CurrentPrice < 25;
{% endhighlight %}

This reads very well in *plain english* (unlike the original version which is
    also longer):  
"*accepted means that the product's current price is smaller than 25*".

Actually, the whole method can be rewritten as:

{% highlight csharp %}
    // MainWindow.xaml.cs
    private void ShowOnlyBargainsFilter(object sender, FilterEventArgs e)
    {
        AuctionItem product = e.Item as AuctionItem;
        if (product != null) e.Accepted = product.CurrentPrice < 25;
    }
{% endhighlight %}

This is only 5 lines of code, unlike the original version which is 16 lines of
code, 11 lines of code waste! The final is *just one third*! Imagine if this
would be happening in a real project!

I really hate when people don't understand boolean logic and write billions of
lines of code that others have to read and maintain!

Regarding the example above: I would go even further. If we look at the method
name we see that it shows *bargains*. This concept can be expressed more
clearly in the code by creating a function `isBargain` in `AuctionItem`.  
This is called the [*tell, don't ask! principle*](http://martinfowler.com/bliki/TellDontAsk.html).

Here's how I would do it:

{% highlight csharp %}
    // MainWindow.xaml.cs
    private void ShowOnlyBargainsFilter(object sender, FilterEventArgs e)
    {
        AuctionItem product = e.Item as AuctionItem;
        if (product != null) e.Accepted = product.IsBargain();
    }

    // AuctionItem.cs
    public bool IsBargain()
    {
        return CurrentPrice < 25;
    }
{% endhighlight %}

This type of logic should be in the *domain model*, not in the user interface as
it was in the original code.

# Dealing with boolean expressions

Here are some of the bad examples I encountered and how you can correct them.
The examples are in java, but it shouldn't matter.

[*Don't compare a boolean with true!*](http://programmers.stackexchange.com/a/12828/3704) It is already true!
{% highlight java %}
    // bad way
    if (something == true) doThis();

    // good way
    if (something) doThis();

    // even better: rename the boolean so it reads like English
    if (isSomething) doThis();

    // good examples: isBargain, isValid, isFile, exists, shouldReceiveBonus
{% endhighlight %}

Same goes for false:
{% highlight java %}
    // bad way
    if (something == false) doThat();

    // good way
    if (!something) doThat();
{% endhighlight %}

Other programmers have an urge to check the boolean when returning from a function.
{% highlight java %}
    // bad way
    if (condition)
        return true;
    else
        return false;

    // good way
    return condition;
{% endhighlight %}

Some other programmers use the ternary operator:
{% highlight java %}
    // bad way
    boolean active = userDisabled? false : true;

    // good way
    boolean active = !userDisabled;
{% endhighlight %}

And finally: don't *EVER* use booleans as parameters to a function. This
violates the [Single Responsibility
Principle](http://www.codinghorror.com/blog/2007/03/curlys-law-do-one-thing.html).
If you need a boolean parameter you are doing *2 things*: one for true and the
other for false. If you didn't know about the Single Responsibility Principle,
      maybe you should check the full stack which is called the [*SOLID
      principles*](https://en.wikipedia.org/wiki/SOLID).

Here are some examples:
{% highlight java %}
    // bad way
    setVisible(true);
    setVisible(false);

    // good way
    show();
    hide();

    // other good examples: enable/disable, switchOn/switchOff
{% endhighlight %}

# Tips and Tricks

How can you avoid writing code as in the original example? Well, here are some
tips:

- [use guard
clauses](http://sourcemaking.com/refactoring/replace-nested-conditional-with-guard-clauses),
- [remove stupid comments that just repeat code](http://www.codinghorror.com/blog/2006/12/code-tells-you-how-comments-tell-you-why.html),
- [remember that the "stuff" you put between parens is a boolean expression and
  that you can assign it directly to a variable...](http://programmers.stackexchange.com/q/199939/3704)
- [remove all those stupid brackets when you have single statements. And you
SHOULD always have only one statement (extract functions for each branch)](http://stackoverflow.com/q/359732/354009)

# The moral of the story

The C# code I showed in the beginning of the article is an example, a demo...
no wonder why new programmers (and not only) write *shitty code*! When they start
learning a new technology they only see *piles of crap all over the Internet*.

So, one moral is for software professionals to be more careful what type of
code they use as examples (unlike Microsoft in this case).

*What type of code YOU want to promote?*
Don't forget that it always backfires and you end up working with people that
will write the same kind of code.

