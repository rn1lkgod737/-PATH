# Installing Pony Mail on Ubuntu 14.04 or 16.04: #
Start by installing the following Ubuntu packages:

- apache2
- git
- liblua5.2-dev
- lua-sec
- lua-cjson
- lua-socket
- python3
- python3-pip
- subversion

~~~
sudo apt-get install apache2 git liblua5.2-dev lua-cjson lua-sec lua-socket python3 python3-pip subversion
~~~

Install the required Python 3 modules:
~~~
sudo pip3 install elasticsearch formatflowed netaddr
~~~

Install ElasticSearch:
~~~
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list
sudo apt-get update && sudo apt-get install elasticsearch default-jre-headless
~~~

Compile and install mod_lua if necessary (httpd < 2.4.17 on Ubuntu):
~~~
sudo apt-get install apache2-dev
svn co https://svn.apache.org/repos/asf/httpd/httpd/branches/2.4.x/modules/lua/
cd lua/
sudo apxs -I/usr/include/lua5.2 -cia mod_lua.c lua_*.c -lm -llua5.2
~~~

Check out a copy of Pony Mail:
~~~
sudo git clone https://github.com/apache/incubator-ponymail.git /var/www/ponymail
~~~

Configure Elasticsearch to automatically start during bootup. For Ubuntu <= 14.10:
~~~
sudo update-rc.d elasticsearch defaults 95 10
~~~

For Ubuntu >= 15.04:
~~~
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
~~~

Start up ElasticSearch:

~~~
sudo service elasticsearch start
~~~

Set up Pony Mail:
~~~
cd /var/www/ponymail/tools
sudo python3 setup.py
[... answer questions asked by the setup script ...]
~~~


Set up Apache httpd by adding, for example, the following virtual host configuration (e.g. in `/etc/apache2/sites-enabled/000-default.conf`):
~~~
<VirtualHost *:80>
    ServerName mylists.foo.tld
    DocumentRoot /var/www/ponymail/site
    AddHandler      lua-script .lua
    LuaScope        thread
    LuaCodeCache    stat
    AcceptPathInfo  On
</VirtualHost>
~~~

Enable mod_lua and start apache, if not already enabled:

~~~
sudo a2enmod lua
sudo service apache start
~~~

Once this is done, you should now have a *working copy* of Pony Mail!

You may wish to tweak the settings in `site/js/config.js` and your
elasticsearch settings once Pony mail is up and running.

Refer to the [General installation documentation](INSTALLING.md) for
detailed information about archiving messages, OAuth, mail settings and
much more.
