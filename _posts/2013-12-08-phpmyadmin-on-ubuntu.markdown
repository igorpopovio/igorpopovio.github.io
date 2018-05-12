---
title: phpmyadmin on ubuntu
layout: post
tags : [phpmyadmin, ubuntu, linux]

---

Installing `phpmyadmin` on Ubuntu can be a bit difficult for someone that
doesn't know how to configure apache (like me till now).  I just executed the
following command:

    sudo apt-get update
    sudo apt-get install phpmyadmin

and expected that when I navigate to `http://localhost/phpmyadmin` it would
simply work. Actually, what I got was *page not found* :(


## Proper way to configure apache

After a bit of googling here's what I found out:
- with the above 2 commands I just installed phpmyadmin,
- what this means is that I only have the phpmyadmin files on my server,
- I need to configure apache to tell it about the new phpmyadmin site location.

Apache has multiple ways of accomplishing this:
- use the `conf.d` directory,
- use the `apache2.conf`,
- use the `httpd.conf`,
- use the `sites-available` directory and `a2ensite`,
- use the `sites-enabled` directory.

Not knowing what is the proper way of doing this I began looking for "best
practices". It seems that the `conf.d` directory is used for global apache
configuration, the `apache2.conf` and `httpd.conf` provide defaults and they
shouldn't be normally touched and the `sites-enabled` directory should contain
only symbolic links to files from the `sites-available` directory.


## Activating phpmyadmin in apache

Ok, so now that we've eliminated all the "wrong" ways to do it only one way
remains: using the `sites-available` and the `a2ensite` command.

But first: phpmyadmin already comes with an apache configuration and we need to
just use it. This configuration is available in:

    /etc/phpmyadmin/apache.conf

Now we need to put it in:

    /etc/apache2/sites-available

To do this we'll create a symbolic link:

    sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/sites-available/phpmyadmin.conf

The next step is to activate this new site using:

    sudo a2ensite phpmyadmin.conf

This command will actually create a link from the `sites-available` to the
`sites-enabled` directory.

The final step is to restart apache with the following command:

    sudo /etc/init.d/apache2 restart

Navigate to `http://localhost/phpmyadmin` and enjoy!


## Quick commands

    sudo ln -s /etc/phpmyadmin/apache.conf
               /etc/apache2/sites-available/phpmyadmin.conf
    sudo a2ensite phpmyadmin.conf
    sudo /etc/init.d/apache2 restart


## Vagrant and puppet

If you need a `phpmyadmin` server just execute these commands:

    git clone https://github.com/sensui/vagrant-phpmyadmin.git
    vagrant up

In a few moments you'll have a phpmyadmin server running on: `http://192.168.33.10/phpmyadmin/`.

Here's the puppet script that accomplishes what this post describes:

{% highlight ruby %}
Exec { path => [ '/bin', '/sbin', '/usr/bin', '/usr/sbin', ] }

exec { 'system-update':
  command => 'sudo apt-get update',
}

Exec['system-update'] -> Package <| |>

package { 'phpmyadmin':
  ensure => present,
}

# linux way: ln -s /etc/phpmyadmin/apache.conf /etc/apache2/sites-available/phpmyadmin.conf
file { '/etc/apache2/sites-available/phpmyadmin.conf':
  ensure => link,
  target => '/etc/phpmyadmin/apache.conf',
  require => Package['phpmyadmin'],
}

exec { 'enable-phpmyadmin':
  command => 'sudo a2ensite phpmyadmin.conf',
  require => File['/etc/apache2/sites-available/phpmyadmin.conf'],
}

exec { 'restart-apache':
  command => 'sudo /etc/init.d/apache2 restart',
  require => Exec['enable-phpmyadmin'],
}
{% endhighlight %}


## Resources

I found these links very helpful in my research:
- [stackoverflow answer](http://stackoverflow.com/a/15467127/354009)
- [apache configuration section from ubuntu manuals](https://help.ubuntu.com/12.04/serverguide/httpd.html#http-configuration)

