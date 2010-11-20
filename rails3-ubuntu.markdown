rails-nginx-passenger-debian
============================

Instructions on how to get up and running with nginx and passenger on Debian Lenny.

* This is an adptation of the rails-nginx-passenger-ubuntu notes
* This was tested on a [Gandi VPS server](https://www.gandi.net/hebergement/) 

Setting SSH keys
----------------

Sur le serveur

    mkdir ~/.ssh/
    chmod 740  ~/.ssh/

Sur le client:

    cat ~/.ssh/id_rsa.pub | ssh myuser@serversip "cat - >> ~/.ssh/authorized_keys"
    
Serveur

    chmod 640  ~/.ssh/authorized_keys
    
Sshd (rightly) refuses to log in if authorized_keys is writable by others. This also applies to containing directories, e.g., ~.
So, if necessary

        chmod 740 /home/myuser

SU
---

    cd /root


Update and upgrade the system
-------------------------------

    apt-get install nano
    apt-get update && apt-get upgrade
    
Add backports to sources.list
-----------------------------
#sqlite won't get the proper version to work with new rails and gem

      nano /etc/apt/sources.list
      add this link at the end : 
      deb http://backports.debian.org/debian-backports lenny-backports main

Install checkrestart ????
--------------------

    apt-get install debian-goodies
    checkrestart #

Configure timezone
-------------------

    # http://nuxwin.com/article-165-debian-etch-lenny-synchronisation-horaire
    
    dpkg-reconfigure tzdata
    apt-get install ntp ntpdate
    nano /etc/default/ntpdate
    # and change to
        NTPDATE_USE_NTP_CONF=no
        
    
Verify that you have to correct date and time with

    date

Adding a cron for automatic update

    crontab -e
    30 04 * * 1-5 /usr/sbin/ntpdate-debian

Configure hostname
-------------------

(pas besoin chez gandi)

    sudo hostname your-hostname

Add 127.0.0.1 your-hostname

    sudo vim /etc/hosts
    
Write your-hostname in 
    
    sudo vim /etc/hostname
    
Verify that hostname is set
    
    hostname

Install mysql
---------------

This should be installed before Ruby Enterprise Edition becouse that will install the mysql gem.

    apt-get install mysql-server libmysqlclient15-dev

SQlite 3
--------

    apt-get install sqlite3 libsqlite3-dev
    
Gemrc
-------

Add the following lines to ~/.gemrc, this will speed up gem installation and prevent rdoc and ri from being generated, this is not nessesary in the production environment.

    ---
    :sources:
    - http://gems.rubyforge.org
    - http://gems.github.com
    gem: --no-ri --no-rdoc


Ruby Enterprise Edition
------------------------

Check for newer version at http://www.rubyenterpriseedition.com/download.html

Install package required by ruby enterprise, C compiler, Zlib development headers, OpenSSL development headers, GNU Readline development headers

    apt-get install build-essential zlib1g-dev libssl-dev libreadline5-dev

Download and install Ruby Enterprise Edition

    http://rubyforge.org/frs/download.php/71096/ruby-enterprise-1.8.7-2010.02.tar.gz

    wget http://rubyforge.org/frs/download.php/66162/ruby-enterprise-X.X.X-ZZZZ.ZZ.tar.gz
    tar xvfz ruby-enterprise-X.X.X-ZZZZ.ZZ.tar.gz 
    rm ruby-enterprise-X.X.X-ZZZZ.ZZ.tar.gz 
    cd ruby-enterprise-X.X.X-ZZZZ.ZZ/
    ./installer
    
    
Change target folder to /opt/ruby for easier upgrade later on

Add Ruby Enterprise bin to PATH Ajout du chemin dans /etc/environment

Pour recharger l'env

    source /etc/environment #to be sure
    
Verify the ruby installation

    ruby -v
    ruby 1.8.7 (2009-06-12 patchlevel 174) [x86_64-linux], MBARI 0x6770, Ruby Enterprise Edition 20090928


Installing git
----------------

    apt-get install git-core

Nginx
-------

    /opt/ruby/bin/passenger-install-nginx-module

Select option 1. Yes: download, compile and install Nginx for me. (recommended)

When finished, verify nginx source code is located under /tmp

    $ ll /tmp/
    drwxr-xr-x 8 deploy deploy    4096 2009-04-18 17:48 nginx-0.6.36
    -rw-r--r-- 1 root   root    528425 2009-04-02 08:49 nginx-0.6.36.tar.gz
    drwxrwxrwx 7   1169   1169    4096 2009-04-18 17:56 pcre-7.8
    -rw-r--r-- 1 root   root   1168513 2009-04-18 17:51 pcre-7.8.tar.gz
    
Run the passenger-install-nginx-module once more if you want to add --with-http_ssl_module 

    $ /opt/ruby/bin/passenger-install-nginx-module
    
Select option 2. No: I want to customize my Nginx installation. (for advanced users)

When installation script ask, "Where is your Nginx source code located?" Enter:

    /tmp/nginx-0.6.36

On, extra arguments to pass to configure script add

     --with-http_ssl_module
     
     
 installé dans /opt/nginx

 user www-data (existe déjà)

 Dans /opt/nginx/conf/nginx.conf ajouter la ligne suivante

     user www-data www-data;

     
     
Nginx init script
-------------------

More information on http://wiki.nginx.org/Nginx-init-ubuntu

    cd
    git clone git://github.com/jnstq/rails-nginx-passenger-ubuntu.git
    mv rails-nginx-passenger-ubuntu/nginx/nginx /etc/init.d/nginx
    chown root:root /etc/init.d/nginx
    chmod +x /etc/init.d/nginx
    
Verify that you can start and stop nginx with init script

    /etc/init.d/nginx start
    
      * Starting Nginx Server...
      ...done.
    
    /etc/init.d/nginx status
    
      nginx found running with processes:  11511 11510
    
    /etc/init.d/nginx stop
    
      * Stopping Nginx Server...
      ...done.
    
    /etc/init.d/nginx syntax
    
    /usr/sbin/update-rc.d -f nginx defaults
    
If you want, reboot and see so the webserver is starting as it should.

Installing ImageMagick and RMagick
-----------------------------------

If you want to install the latest version of ImageMagick. I used MiniMagick that shell-out to the mogrify command, worked really well for me.

    # If you already installed imagemagick from apt-get
    #useless on gandi
    apt-get remove imagemagick

    apt-get install libperl-dev gcc libjpeg62-dev libbz2-dev libtiff4-dev libwmf-dev libz-dev libpng12-dev libx11-dev libxt-dev libxext-dev libxml2-dev libfreetype6-dev liblcms1-dev libexif-dev perl libjasper-dev libltdl3-dev graphviz gs-gpl pkg-config

Use wget to grab the source from ImageMagick.org.

    wget http://image_magick.veidrodis.com/image_magick/ImageMagick.tar.gz

Once the source is downloaded, uncompress it:


    tar xvfz ImageMagick.tar.gz


Now configure and make:

    rm ImageMagick.tar.gz
    cd ImageMagick-6.6.4-7
    ./configure --disable-static --with-modules --without-perl --without-magick-plus-plus --with-quantum-depth=8 --with-gs-font-dir=$FONTS
    make && make install

To avoid an error such as:

convert: error while loading shared libraries: libMagickCore.so.2: cannot open shared object file: No such file or directory

    ldconfig

Install RMagick
 
    gem install rmagick

Sur gandi
---------

lien symbolique vers dossier de datas
    mkdir /srv/d_tib-metal/www

    ln -s /srv/d_tib-metal/www www

Test a rails applicaton with nginx
----------------------------------

    rails testapp
    chown -R myuser:myuser testapp
    cd testapp
    
    ruby script/generate scaffold post title:string body:text
    rake db:migrate RAILS_ENV=production
    
Check so the rails app start as normal
    
    ruby script/server -e production

    nano /opt/nginx/conf/nginx.conf
    
Add a new virutal host

    server {
        listen 8000;
        # server_name www.mycook.com;
        root /home/deploy/www/testapp/public;
        passenger_enabled on;
    }
    
Restart nginx

    sudo /etc/init.d/nginx restart
    
Check you ipaddress and see if you can acess the rails application

On ajoutera www-data au groupe de l'utilisateur auquel appartient l'appli pour éviter certains problèmes de permissions (à tester mais pas franchement utile pour des applis rails via passenger puisqu'il lance l'appli sous les droits du user concerné)

    adduser www-data tolix_sas        

Si problèmes droits d'accès (403…), bien vérifier que pas de Suid sur le dossier home du user. Sinon

    chmod -s /home/myuser
    # pour permettre aux users du group d'acceder au dossier /home/myuser
    chmod 751 /home/myuser
    
## Sending mails ##

<http://www.khayspace.info/index.php/installer-lenvoi-de-mails-depuis-son-site-la-fonction-php-mail/>

Très simple en installant Exim4

	apt-get install exim4

et répondre au questions... notamment type de server = envoi sur le web

Si besoin d'une reconfiguration:

	dpkg-reconfigure exim4-config

Et comme on utilise exim à la place de sendmail

	ln -s /usr/sbin/exim /usr/sbin/sendmail

Testing in rails: créer un fichier mail.rb dans initilizers/

    ActionMailer::Base.raise_delivery_errors = true
    ActionMailer::Base.delivery_method = :sendmail
    ActionMailer::Base.perform_deliveries = true

Dans la console rails

    >> class MyMailer < ActionMailer::Base
    >>   def test_email
    >>     @recipients = "receiver at somedomain.com"
    >>     @from = "contact at yourdomain.com"
    >>     @subject = "test from the Rails Console"
    >>     @body = "This is a test email"
    >>   end
    >> end

    puis

    >> MyMailer::deliver_test_email
    
## Adding log rotatation for nginx ##

(from <https://wincent.com/wiki/nginx_log_rotation>)

Make a /etc/logrotate.d/nginx file (see file)

Tester la rotation: logrotate -d /etc/logrotate.d/nginx
Real (forced) rotation with verbose: logrotate -f -v /etc/logrotate.d/nginx

## Adding log rotatation for rails app ##

(http://overstimulate.com/articles/logrotate-rails-passenger)

Make a /etc/logrotate.d/my-rails-app file (see file)

Tester la rotation: logrotate -d /etc/logrotate.d/my-rails-app
Real (forced) rotation with verbose: logrotate -f -v /etc/logrotate.d/my-rails-app

## Nginx conf ##

Adding gzip (http://brainspl.at/nginx.conf.txt)
Adding cache for public assets (http://weblog.redlinesoftware.com/2009/1/25/nginx-expiry-for-ruby-on-rails)

## Denyhosts ##

http://www.howtoforge.com/preventing_ssh_dictionary_attacks_with_denyhosts

    apt-get install denyhosts
    nano /etc/denyhosts.conf # make changes (alert email recipient, …)
    
    PURGE_DENY = 1w
    ADMIN_EMAIL = tech@yourdomain.com
    SMTP_FROM = DenyHosts <denyhosts@yourwebsite.com>
    

## Monit ##
http://nyrodev.info/index.php/2009/03/28/248-monitoring-monit-munin-serveur-web-sur-debian-lenny

    apt-get install monit
    mv /etc/monit/monitrc /etc/monit/monitrc_default
    nano /etc/monit/monitrc # see example file
    mkdir /etc/monit/conf.d
    nano /etc/monit/conf.d/cron  # see example file
    # ... see all examples files in /etc/monit/conf.d/
    
    nano /etc/default/monit # see example file
    /etc/init.d/monit syntax # to be sure
    /etc/init.d/monit force-reload
    monit summary
    
## Apticron ##
http://nyrodev.info/index.php/2009/03/30/249-autres-ntpdate-apticron-etc-serveur-web-sur-debian-lenny

    apt-get install apticron
     # see example file
    
## Other ##

Keep your passenger apps awaken and ready after logrotation and nginx restarts

In your cron, access your website every hour by adding a line like this for each app.

30 *    * * *   curl --silent http://www.mywbesite.org/ >& /dev/null

    
