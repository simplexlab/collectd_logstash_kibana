# Server performance stats

***

## Overview

Collect and analyze server performance statistics using:

collectd, logstash, elasticsearch, and kibana.

***

## Installation Guide

### Required software
* Ubuntu 12.04.4 LTS (probably works on 14.04 but this has not been tested)
* Logstash 1.4.2 to receive data from collectd and store that into elasticsearch
* Java 1.7.0_65
* Elasticsearch 1.2.2+
* optional: use Nginx as a proxy for Kibana

### install java:
```
sudo apt-get install -y python-software-properties
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install -y oracle-java7-installer
sudo update-alternatives --config java
sudo update-alternatives --config javac
sudo update-alternatives --config javaws
java -version
```

### install elasticsearch:
```
wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.2.2.deb
sudo dpkg -i elasticsearch-1.2.2.deb
sudo dpkg -l elasticsearch
verify it's not running:
sudo service elasticsearch stop
ps aux | grep -i elasticsearch
```

### install logstash:
```
java -version ... ensure java is installed before installing logstash
wget https://download.elasticsearch.org/logstash/logstash/packages/debian/logstash_1.4.2-1-2c0f5a1_all.deb
sudo dpkg -i logstash_1.4.2-1-2c0f5a1_all.deb
sudo dpkg -l logstash
ps aux | grep -i logstash
... if started then stop it, as it needs to be configured properly:
sudo service logstash-web stop ... we don't need this
... to stop logstash-web from starting on boot:
sudo mv /etc/init/logstash-web.conf /etc/init/logstash-web.conf.ORIGINAL
sudo service logstash stop
```

### install collectd:
```
sudo apt-get install collectd
... it starts after install so stop it in order to configure it:
sudo service collectd stop
sudo nano /etc/collectd/collectd.conf
- change these lines as appropriate for the installation server:
Hostname "????"
<Plugin network>
	<Server "x.x.x.x" "25826">
	</Server>
</Plugin>

<Plugin interface>
	Interface "eth0"
	IgnoreSelected false
</Plugin>
... this file is big and there are lots of options, but the
included file in collectd/etc/collectd/collectd.conf is a good start
```

### install kibana:
```
wget https://download.elasticsearch.org/kibana/kibana/kibana-3.1.0.tar.gz
tar -xvzf kibana-3.1.0.tar.gz
sudo cp -R kibana-3.1.0/ /var/www/kibana
sudo nano /var/www/kibana/config.js
	- change this line to the correct IP/hostname and port:
	elasticsearch: "http://192.168.0.2:9200",
... simple test web server:
cd /var/www
python -m SimpleHTTPServer
... in web browser:
http://192.168.0.2:9200/ ... or whatever was set in /var/www/kibana/config.js
... use ctrl+c to the python web server after doing the simple test
```

At this point, it's best to install nginx/apache/other as a web server for kibana.

Also, there are two kibana dashboard json files included in kibana/*.json, but only 
the kibana/system_resources.json works properly as seen in this screenshot:
![kibana](collectd_kibana_dashboard.png)


### optional: install a proxy between Kibana and Elasticsearch:
>   * https://github.com/elasticsearch/kibana
>   * https://github.com/elasticsearch/kibana/blob/master/sample/nginx.conf
>   * http://www.ragingcomputer.com/2014/02/securing-elasticsearch-kibana-with-nginx
>   * or use NodeJS + Express instead of Nginx:
>     * https://github.com/hmalphettes/kibana-proxy
>     * https://github.com/fangli/kibana-authentication-proxy

***
