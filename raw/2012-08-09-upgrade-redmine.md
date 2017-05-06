---

post_title: Upgrade Redmine
author: ytjohn
post_date: 2012-08-09 02:00:00
post_excerpt: ""
layout: post
permalink: >
  http://devblog.yourtech.us/2012/08/09/upgrade-redmine/
published: true
---
Currently, I have Redmine version redmine 1.3.3 installed via the
<a href="https://launchpad.net/~ondrej/+archive/redmine" title="ppa:ondrej/redmine">ondrej/redmine PPA</a>. I have been wanting to upgrade to the 2.x series
of redmine, but no PPA currently exists for it. Redmine is officially
provided by Ubuntu, but the version for Precise is 1.3.2, and Ondřej's
PPA is on 1.4.3. While I usually prefer to have my software installation
and updates handled by packages, it looks like to get to the 2.x series,
I'll have to go back to source.

I will be following the <a href="http://www.redmine.org/projects/redmine/wiki/RedmineUpgrade" title="Upgrade to 2.x">official upgrade guide</a> closely, but with a
few variations.

<ol>
<li>The apt-get/ppa version uses multiple file locations for source code
    and configuration. I'll have to consolidate to one place.</li>
<li>My ruby and passenger modules were installed and modified to work
    with the ppa version of redmine. Adjustments will be needed.</li>
</ol>

My ruby version is 1.8.7 (1.8.7 min), rails 2.3.14 (2.3.6 min) and gem
1.8.15 (1.8 min. Already having the minimum requirements makes this a
bit easier.

After performing a mysql backup (hint: database.yml is in
/etc/redmine/default), I downloaded redmine to /usr/local/redmine-2.0. I
also decided to stop Apache so that Passenger wouldn't be running the
existing redmine instance. If I had other sites running on this server,
I would have disabled this virtual host or put up a maintenance page.

<pre><code>cp /etc/default/database.yml /usr/local/redmine-2.0/config
cp /var/lib/redmine/default/files/* /usr/local/redmine-2.0/files
</code></pre>

I didn't have any plugins, but if I did they would either be in
/usr/share/redmine/vendor/plugins or /usr/share/redmine/lib/plugins. I
do intend to install a couple plugins when I get into 2.x though.

I found in step 3 that the rake commands didn't work. This is probably
because I wasn't working from an existing rails directory. I went to the
<a href="http://www.redmine.org/projects/redmine/wiki/RedmineInstall" title="Install Guide">Redmine Installer</a> page, which gave me the answer. "Since 1.4.0,
Redmine uses Bundler to manage gems dependencies. You need to install
<a href="http://gembundler.com/">Bundler</a> first.".

<pre><code>cp /etc/default/database.yml /usr/local/redmine-2.0/config
cp /var/lib/redmine/default/files/* /usr/local/redmine-2.0/files
</code></pre>

I ran into an error when bundle was installing json 1.7.4.

<pre><code>gem install bundler
# run the next command from /usr/local/redmine-2.0
# it reads the file "Gemfile"
bundle install --without development test
</code></pre>

According to an <a href="http://ruby.about.com/od/faqs/qt/Extconf-Rb-1-In-Require-No-Such-File-To-Load-Mkmf-Loaderror.htm">about.com page</a>, I need build-essentials,
libopenssl-ruby, and ruby1.8-dev installed. The one I was missing was
ruby1.8-dev. This is easily fixed with an <code>apt-get install ruby1.8-dev</code>.

I had to install the following other packages for certain gems. The
Gemfile includes items for postgresql and sqlite, even if you don't use
it. The <a href="http://www.redmine.org/projects/redmine/wiki/RedmineInstall" title="Install Guide">install guide</a> lets you know that you can
skip these with the --without option. You would just add "pg sqlite
rmagick" to the end of your bundle install line (above).

<ul>
<li>json: build-essentials, libopenssl-ruby, and ruby1.8-dev</li>
<li>mysql: libmysqlclient-dev</li>
<li>pg: libpq-dev (alternatively: add pg to the --without list)</li>
<li>rmagick: libmagickcore-dev, libmagickwand-dev (alternatively: add
    rmagick to the --without list)</li>
<li>sqlite: libsqlite3-dev</li>
</ul>

Once we got Bundler installed and all the required gems, we switch back
to the <a href="http://www.redmine.org/projects/redmine/wiki/RedmineUpgrade" title="Upgrade to 2.x">Upgrade Guide</a> to update our session
store and migrate the database. I had no plugins, so I'm skipping that
step.

<pre><code>/usr/bin/ruby1.8 extconf.rb
extconf.rb:1:in `require': no such file to load -- mkmf (LoadError)
from extconf.rb:1
</code></pre>

Let's start this locally before we mess with passenger or apache (be
sure to allow port 3000 via iptables or ufw).

<pre><code>rake generate_secret_token
rake db:migrate RAILS_ENV=production 
# unecessary, as this is a new directory, but why not clean up?
rake tmp:cache:clear
rake tmp:sessions:clear
</code></pre>

This worked without a hitch for me. Now on to my passenger setup. I
already has this configured and installed previously, so all I have to
do is change my VirtualHost directory.

<pre><code>ServerName projects.example.com
DocumentRoot /usr/local/redmine-2.0/public
RailsSpawnMethod smart
# Keep the application instances alive longer. Default is 300 (seconds)
PassengerPoolIdleTime 1000
RailsAppSpawnerIdleTime 0
RailsFrameworkSpawnerIdleTime 0
PassengerMaxRequests 5000
PassengerStatThrottleRate 5

        AllowOverride all
        Options -MultiViews
</code></pre>

I did have to change a few permisssions (all files installed as owned by
root)

<pre><code>chgrp -R www-data config
chown -R www-data files
chown -R www-data log
</code></pre>
