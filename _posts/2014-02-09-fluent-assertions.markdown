---
title: Fluent Assertions
layout: post
tags : [test, tdd, fluent, junit, assert]

---

One of the tricks I learned regarding writing unit tests is that sometimes it's
better to write your own assertion classes.  Say you need to test the
conversion of measurements. 

![Fluent Assertions](/images/fluent-assertions.jpg)

# Code we want to test

In this example we're interested just to see how the tests end up looking so
we're not going to write any implementation code; we'll just use a library for
the actual conversion (jscience).                                   

{% highlight java %}
private double convert(double value, String fromUnit, String toUnit) {
    return Unit.valueOf(fromUnit)
            .getConverterTo(Unit.valueOf(toUnit))
            .convert(value);
}
{% endhighlight %}

# Normal way

This is how we would normally write the tests. Just use the standard junit
asserts.

{% highlight java %}
public static final double DELTA = .01;

@Test
public void normalAsserts() throws Exception {
    double actualKilometres = convert(1000, "m", "km");
    assertEquals(1, actualKilometres, DELTA);

    double actualPounds = convert(1, "t", "lb");
    assertEquals(2204.62, actualPounds, DELTA);

    double actualGallons = convert(10, "L", "gal");
    assertEquals(2.64, actualGallons, DELTA);
}
{% endhighlight %}

# Improved way

The normal way is ok in most cases, but we want to increase their readability
and maintainability. One method of doing this is to make the tests describe as
close as possible to English the behaviour we want to test.

{% highlight java %}
@Test
public void improvedAsserts() throws Exception {
    assertThat(1000, "m").equals(1, "km");
    assertThat(1, "t").equals(2204.62, "lb");
    assertThat(10, "L").equals(2.64, "gal");
}
{% endhighlight %}

Let's take a closer look at one of the asserts:

{% highlight java %}
assertThat(1000, "m").equals(1, "km");
{% endhighlight %}

![Readable tests](/images/fluent-assertions-readable-tests.png)

It reads almost as English (if you ignore the punctuation): "Assert that 1000
  metres equals 1 kilometre". Another thing to notice here is that the value
  and the unit are grouped together as a single concept which is not so clear
  in the first example. Of course, the unit and the value might be better in a
  separate class called Measurement or something like that.

# How to do it?

The basic idea is to create a method `assertThat` that returns a new
ConversionAssert and contains a method that does the actual verification.

{% highlight java %}
private ConversionAssert assertThat(double value, String fromUnit) {
    return new ConversionAssert(value, fromUnit);
}

private class ConversionAssert {
    private final double value;
    private final String fromUnit;

    public ConversionAssert(double value, String fromUnit) {
        this.value = value;
        this.fromUnit = fromUnit;
    }

    public void equals(double expected, String toUnit) {
        double actual = convert(value, fromUnit, toUnit);
        String message = String.format("conversion from %f %s to %s", value, fromUnit, toUnit);
        assertEquals(message, expected, actual, DELTA);
    }
}
{% endhighlight %}

# Resources

- [Full example on Github](https://github.com/sensui/fluent-assertions-example)
- [Write Maintainable Unit Tests That Will Save You Time And Tears](http://msdn.microsoft.com/en-us/magazine/cc163665.aspx)
- [FEST Assertions](https://github.com/alexruiz/fest-assert-2.x/wiki/One-minute-starting-guide)
