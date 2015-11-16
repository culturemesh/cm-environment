# Installing CultureMesh

## ALL THE THINGS

+++ Note +++
This tutorial kinda assumes you're working in some kind of terminal set up
If you're not, that's cool. You can probably figure out how to translate this stuff
into mouseclicks and dragdrops.

+++ Note +++ 
Except for the parts that absolutely require you to work
in a terminal. And those parts exist. If you're working on Linux
or Mac, you should have a perfectly good terminal for all kinds of
whatever-you-wanna-do. If you're on Windows, might I recommend ConEmu?
It's functional, it's light, and it looks damn fine on your desktop.

Another alternative is the git shell that comes packaged with every
git installation. You've got options.

+++ Note +++
If you're doing this on a laptop with an intel processor, check to see
if you have vt-x enabled, if not, you won't be able to run a 64 bit
server. Don't worry, though, we're working on fixing that.

+++ Note +++ 
If you're confused about anything, stop. You're not a bad
person. Don't hesitate to contact me and ask about stuff.
I've tried to cover everything in this guide but it's all too
possible that I've missed something important.

Oh, and welcome to the team.

Here's all the software you'll need:

- Install Git (http://git-scm.com/)
- Install Vagrant (https://www.vagrantup.com/)
- Install VirtualBox (https://www.virtualbox.org/wiki/Downloads)
- Install Ruby (https://www.ruby-lang.org/en/downloads/)
	+ I'm on version 2.0.0p598 (haven't had any trouble with different versions, but haven't tried)
- Install PHP 5.4 if you want to be able to debug (Hint: Yes you do!)

-- may have to play around with different version configurations for the vagrant and virtualbox installs
-- for me, Windows 8.1 ( Vagrant 1.6.5, VirtualBox 4.3.20 r96967, I believe a higher version of vagrant would have had problems )

## THE ENVIRONMENT

1. Clone this repository into whatever folder you want to

	git clone https://github.com/culturemesh/cm-environment.git

2. Create a folder in this directory called htdocs

	mkdir htdocs

The htdocs folder is our root. It's not the ABSOLUTE ABSOLUTE
root, but it is our domain root (where www.culturemesh.com - or
www.culturemesh.build, in your case - points to on the server). In
other words, it's not the root of all evil, but it's certainly the
root of some of it.

3. Start the virtual machine

	vagrant up

This step accomplishes several things. If it's your first time setting
up the server, vagrant will attempt to download the appropriate box
(CentOS 6.5 x64, in our case). Next, vagrant will boot up the virtual
machine. And finally, vagrant will attempt to provision (download and
configure) the machine, basically molding your vm into the bubbly and
fun OS that we all know and love.

Time to check if things were successful.

SSH yourself on into the VM's terminal with the command:

	vagrant ssh

And then run:

	sudo netstat -tlnp

Should give you this as output:

![Working out image bugs] (images/netstat-output.jpg)

If you see a connection for httpd on port 80, congratulations! That's our apache server.

And if you look in your browser, the url www.culturemesh.build should present
a nifty little apache directory listing.

4. Create a folder in htdocs called user_images

5. Get the code

If you're running this from the terminal, you'll want to navigate into htdocs, and then 
execute the following command:

	git clone https://github.com/culturemesh/cm-environment.git culturemesh-live

This command will clone all the code from the master branch into 

If you're not running from a terminal, you can hit up:

<https://github.com/culturemesh/culturemeshdevteam/>

And download the zip file of all the goodness. Then unzip it into htdocs/culturemesh-live.

6. Create an htaccess file in the htdocs folder

	touch .htaccess

The htaccess file allows us to arrange our public_html folder in a very
specific way. Basically it's telling the server that when a client requests
www.culturemesh.build to direct the request to the folder that contains all
the culturemesh code.

The code should be as follows:
	
	RewriteEngine On
	Options +FollowSymLinks

	# make this main domain
	RewriteCond %{HTTP_HOST} ^www\.culturemesh\.build$ [NC]
	# change subdirectory
	RewriteCond %{REQUEST_URI} !^/culturemesh\-live/htdocs/$
	# Don't change these lines
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteCond %{REQUEST_FILENAME} !-d
	# Change culturemesh live to main directory domain
	RewriteRule ^(.*)$ culturemesh\-live/htdocs/$1

	# Change culturemesh.com to main domain
	# Change 'subdirectory' to be directory for main domain
	# followed by / then the main file
	RewriteCond %{HTTP_HOST} ^www.culturemesh.build$ [NC]
	RewriteRule ^(/)?$ culturemesh\-live/htdocs/index.php [L]

7. Contact me for the configuration file (and further instructions, WOOOOOOOO)

8. Setup your bash

From SSH, navigate to the home folder:

	cd ~

And open up the .bashrc file
	
	vim .bashrc

And copy in this little passage:

	if [ -d "$HOME/bin" ] ; then
	  PATH="$HOME/bin:$PATH"
	fi

And, of course, save.

## THINGS YOU MAY NEED TO CHECK

1. If your browser isn't showing the culturemesh home page when you
give it www.culturemesh.build, it's likely that apache is ignoring the
htaccess file.

Traipse on over to ssh land again with vagrant ssh and then open up
the server's configuration file with:

	sudo nano /etc/httpd/conf.d/10-default_vhost_80.conf

Find this little passage:

	  ## Vhost docroot
	  DocumentRoot "/var/www/html"

	  ## Directories, there should at least be a declaration for /var/www/html

	  <Directory "/var/www/html">
	    Options Indexes FollowSymLinks MultiViews
	    AllowOverride None
	    Order allow,deny
	    Allow from all
	  </Directory>

And change it to

	  ## Vhost docroot
	  DocumentRoot "/home3/culturp7/public_html"

	  ## Directories, there should at least be a declaration for /home3/culturp7/public_html

	  <Directory "/home3/culturp7/public_html">
	    Options Indexes FollowSymLinks MultiViews
	    AllowOverride All
	    Order allow,deny
	    Allow from all
	  </Directory>

Hit Ctrl-X to exit the document, and press Y to save the changes

And then restart the server with

	sudo service httpd restart

And try accessing www.culturemesh.build again...fingers crossed

2. If session variables aren't being saved (when you login, you get an error message, or nothing at all)

Breathe.

Are you breathing?

You're dealing with a permissions issue. It's not a big deal.

All you have to do is change the group of php's specified session folder. That task is handled with this command:

	sudo chgrp www-data /var/lib/php/session

Then restart the server with

	sudo service httpd restart

And continue breathing. That remains important. But don't get cocky
about it. It's unprofessional. But...nobody will complain if - in a moment
of weakness - you go ahead and breathe yourself a nice big gulp of
victory oxygen.

3. You'll also need to know where the php.ini file is located.

It should be at /etc/php.d/zzzz_custom.ini
