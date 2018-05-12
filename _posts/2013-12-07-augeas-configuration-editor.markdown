---
title: Augeas, the configuration editor
layout: post
tags : [configuration, editor, augeas]

---

I just found a new interesting tool that can be used to edit configuration
files. It's so good that you can change some options without parsing the file.
You just need to give it the file name and tell it to change a specific option.

If you're on ubuntu you need to install it using the following command: 

    sudo apt-get install augeas-tools augeas-lenses


## Printing properties

I recently wanted to enable the `display_errors` in `php.ini` in an automated
way. To make things easier we're just going to display the contents of the
file. This is done using:

    augtool print /files/etc/php5/apache2/php.ini

Notice I used the `/files` prefix before putting the full path of the file I
want to access. That's how it's done in augeas.

Now, to show the property we're interested in we need to run the following
command:

    augtool print /files/etc/php5/apache2/php.ini/PHP/display_errors

Here's the output returned: 

    /files/etc/php5/apache2/php.ini/PHP/display_errors = "On"


## Debugging

I tried the above command without luck for a while before I realized that my
`php.ini` file had a mistake from a previous manual editing I've done.  In this
case augeas didn't show any errors. It just didn't show anything. So you have
to be careful with this type of things. To actually see what was wrong with the
file you're trying to parse run this command:

    augtool print /augeas//error

This will at least show the line number where the problem is.

## Changing properties

Now that we know how to show the value of a property let's see how can we change it, shall we?
Run the augtool as root and set the `display_errors` configuration as shown below:

    $ sudo augtool
    augtool> set /files/etc/php5/apache2/php.ini/PHP/display_errors "Off"
    augtool> save
    Saved 1 file(s)
    augtool> exit

If you get an error or something doesn't work remember to run `augtool print
/augeas//error`.


## Lenses

Till now you probably have the impression that this `augtool` works only with
php configuration files... Well, it actually works with any configuration file
and here's why... Augeas actually reads the file you give it and saves it into
a tree based on a *lense*. This lense is actually a file containing regular
expressions that identify each option for a specific configuration file. So we
can have lenses for pretty much anything: cron, grub, hosts, mysql, passwd,
    ssh, xml, json, ini. Here's [*a list*](http://augeas.net/stock_lenses.html)
    with all the lenses that augeas comes with by default.


## Puppet

If you happen to use puppet, here's how to do it:

{% highlight ruby %}
    augeas { 'display_errors':
      context => '/files/etc/php5/apache2/php.ini',
      changes => [
        'set PHP/display_errors On',
      ]
    }
{% endhighlight %}

## More information

You can read more about augeas on the official website (although I didn't find
    it particularly informative) and on this page that actually contains really
good information: [*Augeas puppet
documentation*](http://projects.puppetlabs.com/projects/1/wiki/puppet_augeas).

