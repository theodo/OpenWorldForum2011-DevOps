!SLIDE subsection

# Rapid and repeatable server setup#
## Configuration management with Puppet ##

!SLIDE incremental

# What is configuration management? #

<br />
<br />
* Writing the system configuration of your servers in files
* Applying these files automatically
* That's it!

!SLIDE

# Why do configuration management? #

<br />
<br />
* To do fast cluster deployment in the cloud: who wants to manually setup 50 EC2 servers?
* To do fast crash-recovery: configuration management is the best documentation for a server's setup
* To have consistent environments for development and production


!SLIDE

# Puppet or Chef#
## Configuration management tools ##

* Two popular recent tools for configuration management: Puppet and Chef
* A master server contains different "recipes" describing system configurations
* Client servers connect to the master server, read their recipe, and apply the configuration

!SLIDE center

# Puppet #

![Puppet](img/puppet_diagram.png)

!SLIDE center

# Puppet references #

![Puppet](img/puppet-references.png)


!SLIDE center

# Let us create a Lighttpd+FastCGI+PHP+MySQL server with Puppet #
## Introduction to Puppet manifests ##


!SLIDE smaller

	class lighttpd
	{

	  package { "apache2.2-bin":
	    ensure => absent,
	  }

	  package { "lighttpd":
	    ensure => present,
	  }

	  service { "lighttpd":
	    ensure => running,
	    require => Package["lighttpd", "apache2.2-bin"],
	  }

	}

!SLIDE smaller

	class lighttpd-phpmysql-fastcgi inherits lighttpd
	{

	  package { ["php5-cgi", "php5-cli", "php5-sqlite"]:
	    ensure => present,
            notify  => Service["lighttpd"],
	  }

	  package { "mysql-server":
	    ensure => present,
	  }

	  exec { "lighttpd-enable-mod fastcgi":
	    path    => "/usr/bin:/usr/sbin:/bin",
	    creates => "/etc/lighttpd/conf-enabled/10-fastcgi.conf",
	    require =>  Package["php5-cgi", "lighttpd"],
	  }

	}

!SLIDE smaller

	class web-server inherits lighttpd-phpmysql-fastcgi
	{

	  file { "/etc/lighttpd/conf-available/99-hosts.conf":
	    source => "/vagrant/files/conf/hosts.conf",
	    notify  => Service["lighttpd"],
	  }

	  exec { "lighttpd-enable-mod hosts":
	    path => "/usr/bin:/usr/sbin:/bin",
	    creates => "/etc/lighttpd/conf-enabled/99-hosts.conf",
	    require => File["/etc/lighttpd/conf-available/99-hosts.conf"],
	    notify  => Service["lighttpd"],
	  }

	}

	include web-server


!SLIDE

# Why not use shell scripts? #

<br />
<br />
<br />
<br />


* Shell scripts are for ops, not for devs. We want both to have control.
* Even on the ops side, Puppet and Chef recipes are more readable
* Puppet and Chef make inheritance and modules easy
* Puppet and Chef are built to be <strong>idempotent</strong>: running them twice in a row will not break your system


!SLIDE subsection

# Same environment for development, testing and production #
## VM provisioning with Vagrant ##


!SLIDE incremental

# Develop on local Virtual Machines #
## Vagrant ##

* Vagrant is a tool to create local VirtualBox VMs, configured automatically by your Chef recipe or Puppet manifest
* It ensures you test on the same environment as your production server
* It is VERY easy


!SLIDE small

# All you need is: #
## Vagrant ##


<br />
<br />
<br />
<br />

* A Puppet manifest
* A few system config files
* A Vagrant conf file

!SLIDE small

# Demonstration #
## Vagrant ##

<br />
<br />
<br />
<br />


    $ git clone git://github.com/fabriceb/sflive2011vm.git .
    $ git clone git://github.com/fabriceb/sflive2011.git
    $ vagrant up

<br />
<br />
<br />
<br />
<br />
<br />
    http://127.0.0.1:2011/


!SLIDE

# Vagrant is the ultimate level of consistency between development and production #
## Vagrant use case ##

* Level 1: same OS flavour<br/><br/>Development on Windows is <strong>forbidden</strong> if your production runs on Debian! This solves encoding/path problems and other useless headaches inside your code.
* Level 2: fixed library versions<br/><br/>Python - virtualenv+pip / Ruby - rvm. This solves most external conflicts between libraries and language versions.
* Level 3: same OS+architecture<br/><br/>Vagrant. This solves architecture consistency for compiled libraries. For example: https://bugs.launchpad.net/ubuntu/+source/matplotlib/+bug/295719