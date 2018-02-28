#Apres installation de la suite ELK#

* Dans /etc/elasticsearch et /etc/logstash modifier les valeurs de ram dans jvm.options
* Dans /etc/elasticsearch modifier elasticsearch.yml :
	* Decommenter et modifier le "network.host:" en localhost

###Reverse-proxy httpd###

* Allez dans etc/httpd/conf.d
* Creer un fichier de conf "kibana.conf"
	* Y coller :

	<VirtualHost *:80>
        ServerName kibana.mysite.com
        ServerAdmin admin@mysite.com

	  ProxyRequests Off
	  ProxyPass / http://127.0.0.1:5601
	  ProxyPassReverse / http://127.0.0.1:5601

	  RewriteEngine on
	  RewriteCond %{DOCUMENT_ROOT}/%{REQUEST_FILENAME} !-f
	  RewriteRule .* http://127.0.0.1:5601%{REQUEST_URI} [P,QSA]
	</VirtualHost>

* Entrer ensuite la commande pour SELinux : 
	* /usr/sbin/setsebool httpd_can_network_connect 1

###Config logstash###

* Voir video de GrafiKArt sur Elastic Stack
* Allez dans /etc/logstash/conf.d
* Creer un fichier de conf "logstash-apache.conf"
* Y coller :

	input {
	  file {
	    path => "/var/log/httpd/*_log"
	    start_position => "beginning"
	  }
	}

	filter {
	  if [path] =~ "access" {
	    mutate { replace => { "type" => "apache_access" } }
	    grok {
	      match => { "message" => "%{COMBINEDAPACHELOG}" }
	    }
	  }
	  date {
	    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
	  }
	}

	output {
	  elasticsearch {
	    hosts => ["localhost:9200"]
	  }
	  stdout { codec => rubydebug }
	}

* Entrer la commande suivante en ettent dans /etc/logstash/conf.d:
  * /usr/share/logstash/bin/logstash -f logstash-apache.conf