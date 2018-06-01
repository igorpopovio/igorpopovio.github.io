---
title: StringFormat in XAML
layout: post
tags : [stringformat, xaml, tips, tricks]

---

In an usual application sometimes you need to "adapt" the values from the view model. This is normally done using `StringFormat`, but we'll see some other options as well.

# Simple StringFormat with binding escape
Let's say that you need to display a temperature in degrees. In the view model you have just the numerical value and in the interface you want to append the `°C` string to make it clear what type of degrees are displayed. Here's how that's done:

{% highlight xml %}
<TextBlock Text="{Binding CelsiusTemp, StringFormat={}{0}°C}" />
{% endhighlight %}

# MultiBinding
The zero from XAML binding is actually the first binding. In the next example `Name` is the `{0}` part and `ID` is the `{1}` part:

{% highlight xml %}
<TextBlock>
    <TextBlock.Text>
        <MultiBinding StringFormat="{}{0} + {1}">
            <Binding Path="Name" />
            <Binding Path="ID" />
        </MultiBinding>
    </TextBlock.Text>
</TextBlock>
{% endhighlight %}

There are however some other ways you can concatenate string values in XAML. Let's review them quickly:
- TextBlock with Run text
- Using StackPanel to group
- Using Converters

# TextBlock with Run text
`TextBlock` supports an inner element called `Run` which can be helpful when you want to concatenate more things.

{% highlight xml %}
<TextBlock>
  <Run Text="Temperature is " />
  <Run Text="{Binding CelsiusTemp}" />
  <Run Text="°C" />
</TextBlock>
{% endhighlight %}

# Using StackPanel to group
In this case you can just dump everything in a `StackPanel` having the `Orientation` set to `Horizontal`.

{% highlight xml %}
<StackPanel Orientation="Horizontal">
    <TextBlock Text="Temperature is "/>
    <TextBlock Text="{Binding CelsiusTemp}"/>
    <TextBlock Text="°C"/>
</StackPanel>
{% endhighlight %}

# Using Converter
And the last example, although I wouldn't really use it in this case (but shown nonetheless just for completeness sake):

{% highlight csharp %}
public class TemperatureConverter : IValueConverter {
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture) {
        return $"Temperature is {value} °C";
    }

    public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture) {
        throw new NotImplementedException();
    }
}
{% endhighlight %}

And here's how to use it in XAML:
{% highlight xml %}
<Grid>
    <Grid.Resources>
        <local:TemperatureConverter x:Key="temperatureConverter"/>
    </Grid.Resources>
    <TextBlock Text="{Binding CelsiusTemp, Converter={StaticResource temperatureConverter}}"/>
</Grid>
{% endhighlight %}


# Most common formatting specifiers
**Formatting numbers using 2 decimal points** is done using `F2` - F means floating point and the following digit is the number of decimal digits. In this case I used 2, but it can be any value. If you want to show only the integral part then use `F0`.

If you also want to **display the thousands separator** you can use `N2`.

{% highlight xml %}
<!-- Consider CelsiusTemp = 1234.5678 -->
<!-- Also take note that the values will be rounded! -->

<!-- This will be: 1234.57 -->
<TextBlock Text="{Binding CelsiusTemp, StringFormat={}{0:F2}}"/>

<!-- This will be: 1235 -->
<TextBlock Text="{Binding CelsiusTemp, StringFormat={}{0:F0}}"/>

<!-- This will be: 1,234.57 -->
<TextBlock Text="{Binding CelsiusTemp, StringFormat={}{0:N2}}"/>

<!-- This will be: 1,235 -->
<TextBlock Text="{Binding CelsiusTemp, StringFormat={}{0:N0}}"/>
{% endhighlight %}


Here are some more examples showing how to **display currency and dates**:

{% highlight xml %}
<Window x:Class="PlayingWithXAML.MainWindow"
        x:Name="wnd"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:system="clr-namespace:System;assembly=mscorlib"
        Width="300"
        Height="200">
    <StackPanel Margin="10">
        <TextBlock Text="{Binding ElementName=wnd, Path=ActualWidth, StringFormat=Window width: {0:#,#.0}}" />
        <TextBlock Text="{Binding ElementName=wnd, Path=ActualHeight, StringFormat=Window height: {0:C}}" />
        <TextBlock Text="{Binding Source={x:Static system:DateTime.Now}, StringFormat=Date: {0:dddd, MMMM dd}}" />
        <TextBlock Text="{Binding Source={x:Static system:DateTime.Now}, StringFormat=Time: {0:HH:mm}}" />
    </StackPanel>
</Window>
{% endhighlight %}

## Resources
- [{} Escape Sequence / Markup Extension](https://docs.microsoft.com/en-us/dotnet/framework/xaml-services/escape-sequence-markup-extension)
- [Standard Numeric Format Strings](https://docs.microsoft.com/en-us/dotnet/standard/base-types/standard-numeric-format-strings)
- [Use StringFormat to add a string to a WPF XAML binding](https://stackoverflow.com/questions/19278515/use-stringformat-to-add-a-string-to-a-wpf-xaml-binding)
- [The StringFormat property](http://www.wpf-tutorial.com/data-binding/the-stringformat-property)
- [How to bind multiple values to a single WPF TextBlock?](https://stackoverflow.com/questions/2552853/how-to-bind-multiple-values-to-a-single-wpf-textblock)
