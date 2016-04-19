# Deploy Ruby On Rails on Ubuntu 14.04

Server: `Nginx with Phusion Passenger`

Ruby Version: `2.1.3`

User System: `deploy`

## User System

The first thing we will do on our new server is create the user account we'll be using to run our applications and work from there.

	sudo adduser deploy
	sudo adduser deploy sudo
	su deploy

## Installing Ruby

The first step is to install some dependencies for Ruby.

	cd ~
	sudo apt-get update
	sudo apt-get install build-essential git-core curl zlib1g-dev libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties -y

### Install rbenv

Installing with rbenv is a simple two step process. First you install rbenv, and then ruby-build:

	cd ~
	git clone git://github.com/sstephenson/rbenv.git .rbenv
	echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
	echo 'eval "$(rbenv init -)"' >> ~/.bashrc
	exec $SHELL

	git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
	echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
	exec $SHELL

	rbenv install 2.1.3
	rbenv global 2.1.3
	ruby -v

The last step is to tell Rubygems not to install the documentation for each package locally

	echo "gem: --no-ri --no-rdoc" > ~/.gemrc

## Installing Nginx

	# Install Phusion's PGP key to verify packages
	gpg --keyserver keyserver.ubuntu.com --recv-keys 561F9B9CAC40B2F7
	gpg --armor --export 561F9B9CAC40B2F7 | sudo apt-key add -

	# Add HTTPS support to APT
	sudo apt-get install apt-transport-https

	# Add the passenger repository
	sudo sh -c "echo 'deb https://oss-binaries.phusionpassenger.com/apt/passenger trusty main' >> /etc/apt/sources.list.d/passenger.list"
	sudo chown root: /etc/apt/sources.list.d/passenger.list
	sudo chmod 600 /etc/apt/sources.list.d/passenger.list
	sudo apt-get update

	# Install nginx and passenger
	sudo apt-get install nginx-full passenger


So now we have Nginx and passenger installed. We can manage the Nginx webserver by using the service command:

	sudo service nginx start

Open up the server's IP address in your browser to make sure that nginx is up and running.


Next, we need to update the Nginx configuration to point Passenger to the version of Ruby that we're using. You'll want to open up /etc/nginx/nginx.conf in your favorite editor. I like to use vim, so I'd run this command:

	sudo vim /etc/nginx/nginx.conf


Find the following lines, and uncomment them:

	##
	# Phusion Passenger
	##
	# Uncomment it if you installed ruby-passenger or ruby-passenger-enterprise
	##

	passenger_root /usr/lib/ruby/vendor_ruby/phusion_passenger/locations.ini;

	passenger_ruby /usr/bin/ruby;
	# passenger_ruby /home/deploy/.rvm/wrappers/ruby-2.1.2/ruby; # If use use rvm, be sure to change the version number
	# passenger_ruby /home/deploy/.rbenv/shims/ruby; # If you use rbenv



## MySQL and PostgreSQL Database Setup

### Installing MySQL

	sudo apt-get install mysql-server mysql-client libmysqlclient-dev

### Installing PostgreSQL

	sudo apt-get install postgresql postgresql-contrib libpq-dev


## Adding The Nginx Host

In order to get Nginx to respond with the Rails app, we need to modify it's sites-enabled.

Open up `/etc/nginx/sites-enabled/default` in your text editor and we will replace the file's contents with the following:


	server {
	    listen 80 default_server;
	    listen [::]:80 default_server ipv6only=on;

	    server_name mydomain.com;
	    passenger_enabled on;
	    #passenger_min_instances 2;
	    #passenger_intercept_errors on;
	    rails_env    production;
	    root         /home/deploy/myapp/public;

	    # redirect server error pages to the static page /50x.html
	    error_page   500 502 503 504  /50x.html;
	    location = /50x.html {
	        root   html;
	    }
	}

This is our Nginx configuration for a server listening on port 80. You need to change the server_name values to match the domain you want to use and in root replace "myapp" with the name of your application.

## References

[Deploy Ruby On Rails on Ubuntu 14.04 Trusty Tahr](https://gorails.com/deploy/ubuntu/14.04)
