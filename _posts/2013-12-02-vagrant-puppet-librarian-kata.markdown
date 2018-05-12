---
title: Vagrant and puppet kata
layout: post
category: katas
tags : [kata, vagrant, puppet]

---

OK, so maybe you’re wondering what’s all this post about... Well, kata means a
kind of exercise used to learn something new.  Vagrant is a command line
wrapper for VirtualBox which is a tool to manage virtual machines.

Software development is pretty hard in general... one of the reasons why it’s
so hard is because each project has a lot of external dependencies: the
operating system, specific versions of libraries, the network, the database
etc. One of the consequences of these dependencies is the so called *works on
my machine* type of issues that almost always happens.

So, there’s a lot of stuff that can go wrong... How can we solve this, at least
on the development side? Well, this is where vagrant comes into play! You
create a virtual machine for your application and then automatically install
all the dependencies. Now you test and develop only on that virtual machine.
This has the advantage that it pins down everything you need for the project
and that everyone uses the same environment. When you fix an issue on your
machine you can be sure as hell that it’s solved once and for all!!! Besides
this, if you’re working on multiple projects with conflicting dependencies (for
example 2 projects requiring different versions of the same library or
framework) you’ll have them both nicely separated and totally independent.


## Requirements

Download and install: git, VirtualBox, vagrant and librarian-puppet.  You
should have at least basic knowledge of the command line (bash/zsh), although I
still explain some of the commands and tricks I use.

In this kata, I'll create a virtual machine with all the required software to
run *a basic php website* (and I'll also play with some of the puppet's
    features).

OK, enough talk... let’s get to work!


## The Vagrantfile

Create a git repository called `vagrant-puppet-librarian-kata`. Now `cd` into it
(there’s a cool trick for this: type `cd` then press `ALT+.` and the previous
command line argument will autocomplete so you don’t need to type it twice).
Create a directory called `provisioning` so everything stays nicely grouped
together. Create a `Vagrantfile` with the following contents:

{% highlight ruby %}
  Vagrant.configure('2') do |config|
    config.vm.box = 'precise32'
    config.vm.box_url = 'http://files.vagrantup.com/precise32.box'

    config.vm.define :node do |node|
      node.vm.hostname = 'node'
      node.vm.network :private_network, ip: '192.168.33.10'
      node.vm.synced_folder '../', '/var/www/vagrant'
      node.vm.provision :puppet do |puppet|
        puppet.module_path = [
          'modules/third-party',
          'modules/main',
        ]
        puppet.manifests_path = 'manifests'
        puppet.manifest_file = 'node.pp'
        puppet.facter = {
          'fqdn' => 'node.igorpopov.me'
        }
      end
    end
  end
{% endhighlight %}

The `Vagrantfile` is just a ruby script with some DSL (Domain Specific
    Language) on top of it. This means that anything written in ruby will also
work. `Vagrant.configure('2')` selects what version of the vagrant API you want
to use. The next 2 lines (with `config.vm.box*`) describe what base operating
system image we should use. We can choose from the 
[VagrantBoxes](http://vagrantbox.es) website.

The next line defines a new node called `:node` and configures it in a ruby
block.  One particularly interesting feature is that we can set an ip address
to this virtual machine. This means that when you start it you could use its
ip address in your browser (if you have a web server installed).

Another interesting feature is that we can specify some folders to be
synchronized between the host and the guest machine. For example you can set
the project directory to be mapped to some directory on the virtual machine and
in this case you can edit the project files on your machine using any editors
you like and then have those files automatically syncronized on the guest
machine.


## The provisioning

The next lines configure the *provisioning* of the virtual machine. This is
just a complicated word for *automatic installation* of software. For this
kata, I'm using puppet. Puppet requires a manifest file that describes what we
want installed on the machine. In our case, this file is `manifests/node.pp`.
By the way, the puppet script is, just like the Vagrantfile, a ruby script
(with a DSL on top).


## Third party modules

Normally, in a programming language, you have libraries/jars/gems/dlls etc.
that you want to use in your project. You also have them in puppet, but they
are called *modules*. You can find them on the [Puppet
Forge](http://forge.puppetlabs.com). Let's see how we can specify what
third-party modules we need for our project...

Create a `Puppetfile` with the following contents:

{% highlight ruby %}
    forge 'http://forge.puppetlabs.com'

    mod 'puppetlabs/apache'
    mod 'example42/php'
{% endhighlight %}

We will use *librarian-puppet* to manage all the third party modules.
Historically, adding third party puppet modules was done using git submodules
which aren’t the best solution. First, because they are pretty complicated to
use and second, because they just sit there in your repository. The most
desirable solution would be to just add them as dependencies somewhere, but not
include the actual source code in your repository. Librarian-puppet will do
this for you. All you have to do is give it a Puppetfile that tells it what
modules you need and librarian-puppet will go and download them for you. 

Run the following command to change the directory where librarian-puppet puts
the third-party modules (by default it’s `modules`):

    librarian-puppet config --local path modules/third-party

This will save the configuration in the `.librarian` directory that you should
add to source control.  Then, to actually get the third party modules you
declared in the Puppetfile you need to run:

    librarian-puppet install --verbose

The `--verbose` flag is needed so that librarian-puppet actually tells it’s
progress (by default you’ll only see that librarian-puppet hangs for 15 seconds
without showing anything). This command will put all the downloaded modules
in the `modules/third-party` directory (the one you’ve configured above).

Now, we need to create the required directories and we’ll use a very cool trick
for this, called *shell expansion*. OK, so, this is the directory structure we
need:

    .
    `-- provision
        |-- manifests
        `-- modules
            |-- main
            `-- third-party

Normally, we’d just do:

    mkdir provision
    mkdir provision/manifests
    mkdir provision/modules
    mkdir provision/modules/main
    mkdir provision/modules/third-party

But there are a few tricks we could use. The mkdir command accepts a command
line switch: `-p` that creates all the directories till the last one. So the
above list of commands becomes:

    mkdir -p provision/manifests
    mkdir -p provision/modules/main
    mkdir -p provision/modules/third-party

And there’s one more trick we can use: shell expansion! If we want to create
multiple directories on the same level in the hierarchy we can use curly
brackets and the shell will automatically expand them. The above example
becomes:

    mkdir -p provision/manifests
    mkdir -p provision/modules/{main,third-party}

Next we need to create the `provision/manifests/node.pp` file:

{% highlight ruby %}
  Exec { path => [ '/bin', '/sbin', '/usr/bin', '/usr/sbin' ] }

  exec { 'system-update':
    command => 'sudo apt-get update'
  }

  Exec['system-update'] -> Package <| |>

  package {
    [ 'tree', 'git', 'vim', 'augeas-tools' ]:
    ensure => present,
  }

  user { 'igor': 
    ensure => present,
  }

  class { 'apache':
    mpm_module => 'prefork',
  }

  apache::vhost { $fqdn:
    priority => 14,
    port => 80,
    docroot => '/var/www/vagrant/labs',
  }

  include apache::mod::php
  include php
{% endhighlight %}


## Command paths

The first Exec statement tells puppet where to look on the disk when running
commands. If we don’t do it here, we’ll need to prefix each command with it’s
path and that’s not so nice...

{% highlight ruby %}
  Exec { path => ['/bin', '/sbin', '/usr/bin', '/usr/sbin'] }
{% endhighlight %}


## Puppet resources

Before we advance, you should know about “puppet resources”. Among the most
important ones are exec, package and service. All the resources have the
following format:

{% highlight ruby %}
  resource-type { 'custom-resource-name':
    key1 => 'value1',
    key2 => 'value2',
  }
{% endhighlight %}


## Updating the system

The exec resource is used to tell puppet to execute a shell command on the
virtual machine.

{% highlight ruby %}
  exec { 'system-update':
    command => 'sudo apt-get update'
  }
{% endhighlight %}

The next thing tells puppet that before installing any package it should first
update the list of packages available:

{% highlight ruby %}
  Exec['system-update'] -> Package <| |>
{% endhighlight %}


## Messing around

Now we tell puppet that we want a list of packages installed:

{% highlight ruby %}
  package {
    [ 'tree', 'git', 'vim', 'augeas-tools' ]:
    ensure => present,
  }
{% endhighlight %}

Let's also create a new user, just for fun :)

{% highlight ruby %}
  user { 'igor': 
    ensure => present,
  }
{% endhighlight %}

See? It was pretty simple, huh?


## Configuring apache and php

Now, for the apache and php installation it's a bit more complex. If we want
both apache and php we need to activate the `prefork` option of the
`mpm_module`. At least that's how it was documented on the [module's main
page](http://forge.puppetlabs.com/puppetlabs/apache). Next, we need to define a
vhost. This will be used so apache knows what files it should use when we
navigate to it. One thing I had difficulties with is that you should set a
*priority less than 15* if you want to use the port 80 or else your files won't
be taken into use. Now we also need to specify where are those files located.
Remember that we configured a `synced_folder` in the Vagrantfile? This is where
we'll use it. As the docroot for the current vhost.

{% highlight ruby %}
  class { 'apache':
    mpm_module => 'prefork',
  }

  apache::vhost { $fqdn:
    priority => 14,
    port => 80,
    docroot => '/var/www/vagrant/labs',
  }

{% endhighlight %}

Next, we need to activate the php module in apache and also include the php puppet module in the current script:

{% highlight ruby %}
  include apache::mod::php
  include php
{% endhighlight %}


## Starting the virtual machine

To start it, just run this command from the directory where the Vagrantfile is
located: `vagrant up`. After vagrant starts the virtual machine it will
provision it using the `node.pp` script I have described.


## Check if apache and php are installed

Ok, now add a `index.php` file with the following content:

{% highlight php %}
  <?php
  phpinfo();
  ?>
{% endhighlight %}

To check if everything is alright (in case you didn't have errors from vagrant
    and/or puppet) you can use ping and curl in this way:

    ping 192.168.33.10
    curl 192.168.33.10


## Resources

I have pushed the complete example (which may be a bit different) [on
github](https://github.com/sensui/vagrant-puppet-librarian-kata).


## Motivation

I have done this kata more than 20 times (over the course of about 2 weeks),
  each time recording myself and reviewing my mistakes. In the beginning I made
  lots of mistakes and I researched a lot on the Internet even the trivial
  stuff.
  
Each time I have progressed bit by bit and now I can do this kata in about 17
minutes almost perfectly using only vim and the command line. I still make
small typos which I fix pretty quickly. The first time I did it in about 40
minutes and I made lots of mistakes and did lots of research on the Internet.

This post will serve me as a reminder in case I forget some steps and why
they are done... and maybe, just maybe, it will also help someone else.

I will also post somewhere the final video with me doing the kata.

Well, if you're still reading, this concludes the explanations. If you have
questions you can comment and ask clarifications.

