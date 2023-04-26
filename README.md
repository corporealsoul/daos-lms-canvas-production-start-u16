**REFERENCES,** I strongly recoment https://github.com/instructure/canvas-lms/wiki/Production-Start but if you want to know what I did on my setup please find below,

### Prerequisites

Familiar with, website configuration and administration - specifically, [Apache](http://httpd.apache.org/) and/or generic [Ruby on Rails](http://rubyonrails.org/) setups. It also doesn't hurt to have a small working knowledge of [Git](http://git-scm.com/), [Postgres](http://www.postgresql.org/), and [Passenger](http://www.modrails.com/).

Secondly, this tutorial is targeting POSIX-based systems (like Mac OS X and Linux).

    sysadmin@dbserver:~$ cat /etc/os-release

Finally, It's recommended a server with at least 8GB RAM, especially if everything is being run on one server.

    sysadmin@dbserver:~$ free -h
    
    sysadmin@dbserver:~$ htop

Here, Installing the database on the same server as the one Canvas is hosted from.

<br>

### Database installation and configuration

### Installing Postgres

Be sure you're running at least Postgres version 9.5

    sysadmin@dbserver:~$ sudo apt-get update
    
    sysadmin@dbserver:~$ sudo apt-get upgrade
    
    sysadmin@dbserver:~$ sudo apt install postgresql postgresql-contrib
    
    sysadmin@dbserver:~$ psql -V

<br>

    sysadmin@dbserver:~$ psql -V (Version)
    
    sysadmin@dbserver:~$ ifconfig (Cluster)
    
    sysadmin@dbserver:~$ sudo lsof -i -P -n | grep LISTEN (Port)
    
    sysadmin@dbserver:~$ sudo netstat -plunt | grep postgres (Port)
    
    sysadmin@dbserver:~$ sudo systemctl status postgresql (Status)
    
    sysadmin@dbserver:~$ whereis postgresql (Location)
    
    sysadmin@dbserver:~$ sudo ls -ltr /var/lib/postgresql/12/main (Data directory)
    
    sysadmin@dbserver:~$ sudo ls -ltr /var/log/postgresql/ (Log file)
    
    sysadmin@dbserver:~$ sudo cat /etc/postgresql/12/main/pg_hba.conf

<br>

### Configuring Postgres

Set up a Canvas user, createuser will prompt for a password 3ri=rL@4dRL9E for the database user,

    sysadmin@dbserver:~$ sudo -u postgres createuser canvas --no-createdb \
       --no-superuser --no-createrole --pwprompt
       
    sysadmin@dbserver:~$ sudo -u postgres createdb canvas_production --owner=canvas

<br>

    sysadmin@dbserver:~$ sudo -i -u postgres

    postgres@dbserver:~$ psql

    postgres-# \conninfo

    postgres-# \du

    postgres=# \list

    postgres=# \c canvas_production 

    canvas_production=# \dn

    canvas_production=# \q

    postgres@dbserver:~$ exit

<br>

**Installing Git,**

    sysadmin@appserver:~$ sudo apt-get install git-core

    sysadmin@appserver:~$ git --version

Once you have a copy of Git installed on your system, getting the latest source for Canvas is as simple as checking out code from the repo,

    sysadmin@appserver:~$ ls -ltr

    sysadmin@appserver:~$ git clone https://github.com/instructure/canvas-lms.git canvas

    sysadmin@appserver:~$ ls -ltr

    sysadmin@appserver:~$ cd canvas

    sysadmin@appserver:~/canvas$ git branch -a

    sysadmin@appserver:~/canvas$ git checkout stable

    sysadmin@appserver:~/canvas$ git branch -a

We need to put the Canvas code in the location where it will run from. We'll refer to /var/canvas (or whatever you chose) as your Rails application root.

    /var/canvas

Take your checkout and make sure you move the contents to this directory you've chosen, We'll be referring to /var/canvas,

    sysadmin@appserver:~$ sudo mkdir -p /var/canvas
    sysadmin@appserver:~$ sudo chown -R anup /var/canvas
    
    sysadmin@appserver:~$ cd canvas
    sysadmin@appserver:~/canvas$ ls
    sysadmin@appserver:~/canvas$ cp -av . /var/canvas
    
    sysadmin@appserver:~/canvas$ cd /var/canvas
    sysadmin@appserver:/var/canvas$ ls

<br>

### External Dependency Installation

We now need to install the Ruby libraries and packages that Canvas needs. Installing dependencies for compiling Ruby along with Node.js and Yarn

    $ sudo apt-get install software-properties-common
    
    $ sudo add-apt-repository ppa:brightbox/ruby-ng
    $ sudo apt-get update

**Now, We install Ruby,**

    $ sudo apt-get install ruby2.7 ruby2.7-dev zlib1g-dev libxml2-dev \
                           libsqlite3-dev postgresql libpq-dev \
                           libxmlsec1-dev curl make g++

<br>

    $ ruby -v
    $ ruby --version
    $ gem -v
    $ gem query --local
    $ sudo gem list
    
    anup@ubuntu-20042:~$ nano program.rb
    
    puts("Ruby code!")
    
    anup@ubuntu-20042:~$ ruby program.rb

**Installing SQLite**

    $ sudo apt-get install sqlite3
    $ sqlite3 --version
    $ sudo apt-get install sqlitebrowser
    $ sudo apt-get install libsqlite3-dev

**Node.js installation**

    $ curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
    $ sudo apt-get install nodejs
    $ node -v
    $ sudo npm install -g npm@latest
    $ npm version

After installing Postgres, you will need to set your system username as a Postgres superuser,

    sudo -u postgres createuser $USER
    sudo -u postgres psql -c "alter user $USER with superuser" postgres
    
    sysadmin@dbserver:~$ sudo -i -u postgres
    postgres@dbserver:~$ psql
    postgres-# \du

<br>

### Ruby Gems

Most of Canvas' dependencies are Ruby Gems.

**Bundler and Canvas dependencies**

Canvas uses Bundler as an additional layer on top of Ruby Gems to manage versioned dependencies.

    sysadmin@dbserver:~$ ruby -v
    sysadmin@dbserver:~$ cd /var/canvas
    sysadmin@appserver:/var/canvas$ sudo gem install bundler --version 2.2.19
    sysadmin@appserver:/var/canvas$ bundle version
    sysadmin@appserver:/var/canvas$ gem query --local | grep -i "bundler"
    sysadmin@appserver:/var/canvas$ sudo gem list | grep -i "bundler"
    sysadmin@appserver:/var/canvas$ bundle _2.2.19_ install --without pulsar --path vendor/bundle

<br>

Canvas now prefers yarn instead of npm.

    sysadmin@appserver:/var/canvas$ curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
    sysadmin@appserver:/var/canvas$ echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
    sysadmin@appserver:/var/canvas$ sudo apt-get update && sudo apt-get install yarn=1.19.1-1
    sysadmin@appserver:/var/canvas$ yarn --version

Also, make sure Python is installed (needed for the Contextify package),

    sysadmin@appserver:/var/canvas$ sudo apt-get install python
    sysadmin@appserver:/var/canvas$ python --version

Then install the node modules,

    sysadmin@appserver:/var/canvas$ sudo yarn install
    # If doesn't work, try
    sysadmin@appserver:/var/canvas$ yarn install
    
    sysadmin@appserver:/var/canvas$ yarn list
    sysadmin@appserver:/var/canvas$ npm list
    sysadmin@appserver:/var/canvas$ npm list --depth=0 (Without dependencies)
    sysadmin@appserver:/var/canvas$ npm -g list (Global)
    sysadmin@appserver:/var/canvas$ npm list grunt

<br>

### Canvas default configuration

Before we set up all the tables in your database, our Rails code depends on a small few configuration files,

    sysadmin@appserver:/var/canvas$ for config in amazon_s3 database \
      delayed_jobs domain file_store outgoing_mail security external_migration; \
      do cp config/$config.yml.example config/$config.yml; done

<br>

### Dynamic settings configuration

Most of the configurations reside,

    sysadmin@appserver:/var/canvas$ ls -ltr config

This config file is useful if you don't want to run a consul cluster with Canvas,

    sysadmin@appserver:/var/canvas$ cp config/dynamic_settings.yml.example config/dynamic_settings.yml
    sysadmin@appserver:/var/canvas$ nano config/dynamic_settings.yml

<br>

**Database configuration**

Open the file config/database.yml, and find the production environment section,

This is the place you will put the password and database name, along with anything else we set up, from the Postgres setup steps.

    sysadmin@appserver:/var/canvas$ cp config/database.yml.example config/database.yml
    sysadmin@appserver:/var/canvas$ nano config/database.yml

The configuration should look like this,

production:
  adapter: postgresql
  encoding: utf8
  database: canvas_production
  host: localhost
  username: canvas
  password: 3ri=rL@4dRL9E
  timeout: 5000

**Outgoing mail configuration**

Find the production section and configure it to match your SMTP provider's settings,

    sysadmin@appserver:/var/canvas$ cp config/outgoing_mail.yml.example config/outgoing_mail.yml
    sysadmin@appserver:/var/canvas$ nano config/outgoing_mail.yml

The configuration should look like this,

    production:
      address: "email-smtp.us-east-1.amazonaws.com"
      port: "25"
      user_name: "user"
      password: "password"
      authentication: "plain" # plain, login, or cram_md5
      domain: "example.com"
      outgoing_address: "canvas@example.com"
      default_name: "Instructure Canvas"

**URL Configuration**

Please edit the **production **section of config/domain.yml to be the appropriate domain name for your Canvas installation,

    sysadmin@appserver:/var/canvas$ cp config/domain.yml.example config/domain.yml
    sysadmin@appserver:/var/canvas$ nano config/domain.yml

The configuration should look like this,

    production:
      domain: 192.168.40.132
      # Whether this instance of the canvas is served over SSL (HTTPS) or not
      # defaults to true for production, false for test/development
      ssl: true
      # files_domain: "canvasfiles.example.com"

**Security configuration**

You must insert randomized strings of at least 20 characters in this file,

    sysadmin@appserver:/var/canvas$ cp config/security.yml.example config/security.yml
    sysadmin@appserver:/var/canvas$ nano config/security.yml

<br>

### Generate Assets

First, create the directories that will store the generated files.

    sysadmin@appserver:~$ cd /var/canvas
    sysadmin@appserver:/var/canvas$ mkdir -p log tmp/pids public/assets app/stylesheets/brandable_css_brands
    sysadmin@appserver:/var/canvas$ touch app/stylesheets/_brandable_variables_defaults_autogenerated.scss
    sysadmin@appserver:/var/canvas$ touch Gemfile.lock
    sysadmin@appserver:/var/canvas$ touch log/production.log
    sysadmin@appserver:/var/canvas$ sudo chown -R anup config/environment.rb log tmp public/assets \
                                  app/stylesheets/_brandable_variables_defaults_autogenerated.scss \
                                  app/stylesheets/brandable_css_brands Gemfile.lock config.ru

Then will need to run,

    sysadmin@appserver:/var/canvas$ sudo yarn install
    If doesn't works, try
    sysadmin@appserver:/var/canvas$ yarn install
    sysadmin@appserver:/var/canvas$ sudo RAILS_ENV=production bundle exec rake canvas:compile_assets
    sysadmin@appserver:/var/canvas$ sudo chown -R anup public/dist/brandable_css

<br>

### Database population

Once your database is configured, and assets are installed, we need to actually fill the database with tables and initial data. You can do this by running our rake migration and initialization tasks from your application's root:

    sysadmin@appserver:/var/canvas$ RAILS_ENV=production bundle exec rake db:initial_setup

<br>

### Canvas ownership

These are the .yml files inside the config directory, and we want to make them readable only by the canvasuser user "anup",

    sysadmin@appserver:/var/canvas$ sudo chown anup config/*.yml
    sysadmin@appserver:/var/canvas$ sudo chmod 400 config/*.yml

<br>

### Apache configuration

**Installation**

You're now going to need to set up the webserver. We're going to use [Apache](http://httpd.apache.org/) and [Passenger](https://www.phusionpassenger.com/) to serve the Canvas content.

Before proceeding, you'll need to [add the Phusion Passenger APT repository](https://www.phusionpassenger.com/library/install/apache/install/), which contains the passenger package.

Install Passenger packages,

    sysadmin@appserver:/var/canvas$ sudo apt-get install -y dirmngr gnupg                                                        
    
    sysadmin@appserver:/var/canvas$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7      
    
    sysadmin@appserver:/var/canvas$ sudo apt-get install -y apt-transport-https ca-certificates     
                             
    
    sysadmin@appserver:/var/canvas$ sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger bionic main > /etc/apt/sources.list.d/passenger.list'
    
    sysadmin@appserver:/var/canvas$ sudo apt-get update
    
    
    sysadmin@appserver:/var/canvas$ sudo apt-get install -y libapache2-mod-passenger
    
    sysadmin@appserver:/var/canvas$ sudo apt-get install -y passenger
    
    sysadmin@appserver:/var/canvas$ passenger -v
    
    sysadmin@appserver:/var/canvas$ sudo apt-get install -y apache2
    
    sysadmin@appserver:/var/canvas$ apache2 -v
    
    sysadmin@appserver:/var/canvas$ sudo systemctl status apache2

Enable the Passenger Apache module and restart Apache,

    sysadmin@appserver:/var/canvas$ sudo a2enmod passenger
    
    sysadmin@appserver:/var/canvas$ sudo systemctl restart apache2
    
    sysadmin@appserver:/var/canvas$ sudo systemctl status apache2

Check installation,

    sysadmin@appserver:/var/canvas$ sudo /usr/bin/passenger-config validate-install
    
    sysadmin@appserver:/var/canvas$ sudo /usr/sbin/passenger-memory-stats
    
    sysadmin@appserver:/var/canvas$ sudo apt-get update

We'll be using mod_rewrite, so you'll want to enable that,

    sysadmin@appserver:/var/canvas$ sudo a2enmod rewrite
    
    sysadmin@appserver:/var/canvas$ sudo systemctl restart apache2

<br>

### Configure Passenger with Apache

If it didn't enable for Apache configuration or they are disabled somehow,

    sysadmin@appserver:/var/canvas$ sudo a2enmod passenger

In other setups, you just need to make sure you add the following lines to your Apache configuration,

    LoadModule passenger_module /usr/lib/apache2/modules/mod_passenger.so
    PassengerRoot /usr
    PassengerRuby /usr/bin/ruby

If you have trouble starting the application because of permissions problems, you might need to add this line to your passenger.conf, site configuration file, or httpd.conf,

    PassengerDefaultUser canvasuser

How passenger.conf looks,

    anup@anuniqs:/var/canvas$ sudo nano /etc/apache2/mods-enabled/passenger.conf
<br>
    ### Begin automatically installed Phusion Passenger config snippet ###
    <IfModule mod_passenger.c>
      PassengerRoot /usr/lib/ruby/vendor_ruby/phusion_passenger/locations.ini
      PassengerDefaultRuby /usr/bin/passenger_free_ruby
      PassengerRuby /usr/bin/ruby
      PassengerDefaultUser anup
    </IfModule>
    ### End automatically installed Phusion Passenger config snippet ###

How passenger.load looks,

    anup@anuniqs:/var/canvas$ sudo nano /etc/apache2/mods-enabled/passenger.load
    
    ### Begin automatically installed Phusion Passenger load snippet ###
    LoadModule passenger_module /usr/lib/apache2/modules/mod_passenger.so
    ### End automatically installed Phusion Passenger load snippet ###

<br>

### Configure SSL with Apache

Next, need to make sure your Apache configuration supports SSL.

    sysadmin@appserver:/var/canvas$ sudo a2enmod ssl
    sysadmin@appserver:/var/canvas$ sudo systemctl restart apache2

On other systems, you need to make sure something like the below is in the config,

    LoadModule ssl_module /usr/lib/apache2/modules/mod_ssl.so
    SSLRandomSeed startup builtin
    SSLRandomSeed startup file:/dev/urandom 512
    SSLRandomSeed connect builtin
    SSLRandomSeed connect file:/dev/urandom 512
    SSLSessionCache        shmcb:/var/run/apache2/ssl_scache(512000)
    SSLSessionCacheTimeout  300
    SSLMutex  file:/var/run/apache2/ssl_mutex
    SSLCipherSuite HIGH:MEDIUM:!ADH
    SSLProtocol all -SSLv2

### A note about SSL

If you want to get a certificate for your Canvas installation that will be accepted automatically by your user's browsers, you will need to contact a certificate authority and generate one. For the sake of example, Digicert, PositiveSSL (for pay) and Let's Encrypt (free) Verisign are commonly used certificate authorities.

After downloading certificates, place them to "/etc/ssl/certs/" this location

      #SSLCertificateChainFile /etc/ssl/certs/dns.com.ca-bundle
      #SSLCertificateKeyFile /etc/ssl/certs/dns.key
      SSLCertificateFile /etc/ssl/certs/ssl-cert-snakeoil.pem

<br>

### Configure Canvas with Apache

Now need to tell Passenger about particular Rails application, unlink any of the symlinks in the /etc/apache2/sites-enabled subdirectory aren't required,

    sysadmin@appserver:/var/canvas$ sudo unlink /etc/apache2/sites-enabled/000-default.conf

Now, need to make a VirtualHost for app,

    sysadmin@appserver:/etc/apache2/sites-enabled$ sudo nano /etc/apache2/sites-available/canvas.conf


In the new file, or new spot, depending, you want to place the following snippet. You will want to modify the lines designated ServerName(2), ServerAdmin(2), DocumentRoot(2), SetEnv(2), Directory(2), and probably SSLCertificateFile(1) and SSLCertificateKeyFile(1), discussed below in the "Note about SSL Certificates".

    <VirtualHost *:80>
      ServerName localhost
      ServerAlias canvasfiles.localhost
      ServerAdmin uniqs.anup@gmail.com
      DocumentRoot /var/canvas/public
      RewriteEngine On
      RewriteCond %{HTTP:X-Forwarded-Proto} !=https
      RewriteCond %{REQUEST_URI} !^/health_check
      RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URI} [L]
      ErrorLog /var/log/apache2/canvas_errors.log
      LogLevel warn
      CustomLog /var/log/apache2/canvas_access.log combined
      SetEnv RAILS_ENV production
      XSendFile On
      XSendFilePath /var/canvas
      <Directory /var/canvas/public>
        Options All
        AllowOverride All
        Require all granted
      </Directory>
    </VirtualHost>
    <VirtualHost *:443>
      ServerName localhost
      ServerAlias canvasfiles.localhost
      ServerAdmin uniqs.anup@gmail.com
      DocumentRoot /var/canvas/public
      ErrorLog /var/log/apache2/canvas_errors.log
      LogLevel warn
      CustomLog /var/log/apache2/canvas_ssl_access.log combined
      SSLEngine on
      BrowserMatch "MSIE [17-9]" ssl-unclean-shutdown
      # the following ssl certificate files are generated for you from the ssl-cert package.
      SSLCertificateFile /etc/ssl/certs/ssl-cert-snakeoil.pem
      SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
      SetEnv RAILS_ENV production
      XSendFile On
      XSendFilePath /var/canvas
      <Directory /var/canvas/public>
        Options All
        AllowOverride All
        Require all granted
      </Directory>
    </VirtualHost>

Apache 2.4 users: the allow/options configuration inside the <Directory /var/canvas/public> have changed in Apache 2.4. You'll likely want something like this:

      <Directory /var/canvas/public>
        Options All
        AllowOverride All
        Require all granted
      </Directory>

Test Apache configuration,

    sysadmin@appserver:/var/canvas$ sudo apachectl configtest

Add server name,

    anup@anup-VirtualBox:/var/canvas$ sudo nano /etc/apache2/apache2.conf

<br>

    ServerName 192.168.56.118

And finally, if you created this as its own file inside /etc/apache2/sites-available, we'll need to make it an enabled site.

    sysadmin@appserver:/etc/apache2/sites-enabled$ sudo a2ensite canvas
    sysadmin@appserver:/etc/apache2/sites-enabled$ sudo systemctl reload apache2

Once the site gets enabled, get your IP,

    sysadmin@appserver:/var/canvas$ hostname -I

**Open your favorite browser,** https://192.168.40.132/

<br>

Optimizing File Downloads,

If you are storing uploaded files locally, make sure that Apache has mod_xsendfile installed and enabled,

    sysadmin@appserver:/var/canvas$ sudo apt-get install libapache2-mod-xsendfile

This command installs and enables the module. To ensure properly running the module,

    sysadmin@appserver:/var/canvas$ sudo apachectl -M | sort

Module xsendfile_module (shared) should be in the list.

    anup@megatron:/var/canvas$ sudo apachectl -M | grep -i "xsendfile_module"

In config/environments/production.rb you'll find the necessary config.action_dispatch.x_sendfile_header line, but commented out. We recommend that you create a config/environments/production-local.rb file and add the uncommented line to that file, to avoid future merge conflicts.

config/environments/production.rb

    anup@anuniqs:/var/canvas$ sudo nano config/environments/production.rb
    
    # If you have mod_xsendfile enabled in apache:
    # config.action_dispatch.x_sendfile_header = 'X-Sendfile'

config/environments/production-local.rb

    anup@anuniqs:/var/canvas$ sudo nano config/environments/production-local.rb
    
    # If you have mod_xsendfile enabled in apache:
    config.action_dispatch.x_sendfile_header = 'X-Sendfile'

In your canvas virtual host at /etc/apache2/sites-available/canvas.conf , add the following two directives:

    anup@anuniqs:/var/canvas$ sudo nano /etc/apache2/sites-available/canvas.conf
    
        XSendFile On
        XSendFilePath /var/canvas

Test Apache configuration,

    sysadmin@appserver:/var/canvas$ sudo apachectl configtest

Restart apache,

    sysadmin@appserver:/var/canvas$ sudo systemctl restart apache2
    
    sysadmin@appserver:/var/canvas$ sudo systemctl status apache2

<br>

### Cache configuration

Canvas supports two different methods of caching: Memcache and redis. However, there are some features of Canvas that require redis to use, such as OAuth2, so it's recommended that you use redis for caching as well to keep things simple.

Below are instructions for setting up redis.

**Redis**

Required version: redis 2.6.x or above.

**Note: Ubuntu installs an older version by default. See [Download](http://redis.io/download)  for instructions on how to manually install redis 2.6.x or above manually or use the PPA below.**

For Ubuntu, you can use the redis-server package. However, on trusty, it's not new enough, so you'll want to use a backport PPA to provide it: [redis-server : chris lea](https://launchpad.net/~chris-lea/+archive/redis-server) .

    sysadmin@appserver:/var/canvas$ sudo add-apt-repository ppa:chris-lea/redis-server
    sysadmin@appserver:/var/canvas$ sudo apt-get update
    sysadmin@appserver:/var/canvas$ sudo apt-get install redis-server
    sysadmin@appserver:/var/canvas$ redis-server -v

Check if Redis is working,

    sysadmin@appserver:/var/canvas$ redis-cli ping

After installing Redis, start the server. There are multiple options for doing this. You can set it up so it runs automatically when the server boots or you can run it manually.

    sysadmin@appserver:/var/canvas$ redis-server &

Now we need to go back to your canvas-lms directory and edit the configuration. Inside the config folder, we're going to copy [cache_store.yml.example](https://github.com/instructure/canvas-lms/blob/stable/config/cache_store.yml.example) and edit it:

    sysadmin@appserver:/var/canvas$ cd /var/canvas/
    sysadmin@appserver:/var/canvas$ sudo cp config/cache_store.yml.example config/cache_store.yml
    sysadmin@appserver:/var/canvas$ sudo nano config/cache_store.yml

The file may start with all caching methods commented out. Match your config file to the entries below:

    test:
      cache_store: redis_store
    development:
      cache_store: redis_store
    production:
      cache_store: redis_store

Change ownership and permission,

    sysadmin@appserver:/var/canvas$ sudo chown anup config/cache_store.yml
    sysadmin@appserver:/var/canvas$ sudo chmod 400 config/cache_store.yml

Then specify your redis instance information in redis.yml, by coping and editing [redis.yml.example](https://github.com/instructure/canvas-lms/blob/stable/config/redis.yml.example):

    sysadmin@appserver:/var/canvas$ cd /var/canvas/
    sysadmin@appserver:/var/canvas$ sudo cp config/redis.yml.example config/redis.yml
    sysadmin@appserver:/var/canvas$ sudo nano config/redis.yml
    
    production:
      servers:
        - redis://localhost

Change ownership and permission,

    sysadmin@appserver:/var/canvas$ sudo chown anup config/redis.yml
    sysadmin@appserver:/var/canvas$ sudo chmod 400 config/redis.yml

In our example, redis is running on the same server as Canvas.

<br>

### QTIMigrationTool

The QTIMigrationTool needs to be installed to enable copying or importing quiz content. Instructions are at [Home](https://github.com/instructure/QTIMigrationTool/wiki)

**Canvas Setup Instructions**

This tool has been modified to be used with the Instructure Canvas Learning Management System.

Running the tool requires the lxml python library. We can install with your local package manager: apt-get install python-lxml, You need to have Python 2.5 or newer (but not Python 3 or newer) ,

    sysadmin@appserver:/var/canvas$ sudo apt-get install python-lxml

Setting up the migration tool with git:

    sysadmin@appserver:anup@anuniqs:/var/canvas$ cd vendor/
    sysadmin@appserver:anup@anuniqs:/var/canvas/vendor$ git clone https://github.com/instructure/QTIMigrationTool.git QTIMigrationTool
    sysadmin@appserver:anup@anuniqs:/var/canvas/vendor$ cd QTIMigrationTool
    sysadmin@appserver:anup@anuniqs:/var/canvas/vendor/QTIMigrationTool$ chmod +x migrate.py

Once the tool is in the vendor directory you will need to restart (or start) the delayed jobs daemons:

    sysadmin@appserver:anup@anuniqs:/var/canvas/vendor/QTIMigrationTool$ cd /var/canvas/
    sysadmin@appserver:anup@anuniqs:/var/canvas$ script/delayed_job restart

In case we face any issues with gem compatibility,

    anup@anup-VirtualBox:/var/canvas$ gem list | grep "bundle"
    anup@anup-VirtualBox:/var/canvas$ sudo rails --version
    anup@anup-VirtualBox:/var/canvas$ sudo gem install bundler --version 2.2.19 --default
    anup@anup-VirtualBox:/var/canvas$ gem list | grep "bundle"

<br>

### Automated jobs

Canvas has some automated jobs that need to run at occasional intervals, such as email reports, statistics gathering, and a few other things. Your Canvas installation will not function properly without support for automated jobs, so we'll need to set that up as well.

**Installation**

    sysadmin@appserver:/var/canvas$ sudo ln -s /var/canvas/script/canvas_init /etc/init.d/canvas_init
    sysadmin@appserver:/var/canvas$ sudo update-rc.d canvas_init defaults
    sysadmin@appserver:/var/canvas$ sudo /etc/init.d/canvas_init start
    sysadmin@appserver:/var/canvas$ sudo /etc/init.d/canvas_init status

Ready, set, go !

Restart Apache (sudo /etc/init.d/apache2 restart), and point your browser to your new Canvas installation! Log in with the administrator credentials you set up during database configuration, and you should be ready to use Canvas.

    sysadmin@appserver:/var/canvas$ sudo /etc/init.d/apache2 restart
    
    sysadmin@appserver:/var/canvas$ sudo systemctl status apache2.service

<br>

**Open your favorite browser, https://192.168.40.132/**

<br>

### Troubleshooting

    sysadmin@appserver:~$ sudo /etc/init.d/apache2 restart
    sysadmin@appserver:~$ sudo systemctl status apache2.service
    
    sysadmin@appserver:~$ sudo /etc/init.d/canvas_init restart
    sysadmin@appserver:~$ sudo /etc/init.d/canvas_init status

<br>

**LIST JOBS, https://192.168.40.133/jobs**

<br>

**Try checking the error_reports table,**

    anup@megatron:/var/canvas$ psql canvas_production -c "select message, backtrace from error_reports order by id desc limit 1;"

<br>

### Logs

Logs are important and most of the logs reside here ,

    sysadmin@appserver:/var/log/apache2$ pwd
    
    sysadmin@appserver:/var/log/apache2$ multitail error.log

<br>
