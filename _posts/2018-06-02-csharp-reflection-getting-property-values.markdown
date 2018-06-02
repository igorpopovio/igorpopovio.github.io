---
title: C# Reflection API
layout: post
tags : [C#, reflection, api, properties, tips, tricks]

---

In this post we're going to see how to dynamically get the value of a property by using only its name. This started from me trying to understand how data binding works in XAML and implementing it myself. For this, I needed a view model which implements the `INotifyPropertyChanged` and then subscribe to its `PropertyChanged` event.

# The view model
{% highlight csharp %}
using System.ComponentModel;

namespace CSharpEventsReflectionAndDataBinding {
    public class Robot : INotifyPropertyChanged {
        public string FirstName { get; set; }
        public string LastName { get; set; }

        private int years;
        public int Years {
            get { return years; }
            set {
                years = value;
                OnPropertyChanged("Years");
            }
        }

        public override string ToString() => $"{FirstName} {LastName}";

        public event PropertyChangedEventHandler PropertyChanged;
        protected virtual void OnPropertyChanged(string propertyName) {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }
    }
}
{% endhighlight %}

# The main program
Just for fun I used **R. Daneel Olivaw** as an example...

{% highlight csharp %}
using System;
using System.Timers;

namespace CSharpEventsReflectionAndDataBinding {
    public class Program {
        static void Main(string[] mainArgs) {
            new Program().Run();
            Console.Read();
        }

        private void Run() {
            var rDaneelOlivaw = new Robot {
                FirstName = "Daneel",
                LastName = "Olivaw",
                Years = 19200,
            };

            PrintYearsWhenItIsChanged(robot: rDaneelOlivaw);
            CreateTimerToIncreaseYears(seconds: 3, robot: rDaneelOlivaw).Start();
        }

        private void PrintYearsWhenItIsChanged(Robot robot) {
            robot.PropertyChanged += (sender, args) => {
                var propertyName = args.PropertyName;
                var propertyValue = typeof(Robot)
                    .GetProperty(propertyName)
                    .GetValue(robot);

                Console.WriteLine($"{robot} now has {propertyValue} {propertyName}");
            };
        }

        private Timer CreateTimerToIncreaseYears(int seconds, Robot robot) {
            var timer = new Timer(seconds * 1000) { AutoReset = true };
            timer.Elapsed += (sender, args) => { robot.Years++; };
            return timer;
        }
    }
}
{% endhighlight %}


# Explanation
To get the property value we first need to know on what `Type` it resides, then the property name and finally the target object.

{% highlight csharp %}
var propertyValue = typeof(Robot)
    .GetProperty(propertyName)
    .GetValue(robot);
{% endhighlight %}

This is a simple concept and in case I ever need it I know where to look for a quick refresher.
