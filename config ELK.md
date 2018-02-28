#Apres installation de la suite ELK#

* Allez dans etc/httpd/conf.d
* Creer un fichier de conf "kibana.conf"
* Y coller :

	<VirtualHost *:80>
        ServerName kibana.mysite.com
        ServerAdmin admin@mysite.com

	  #
	  # Proxy
	  #
	  ProxyRequests Off
	  ProxyPass / http://127.0.0.1:5601
	  ProxyPassReverse / http://127.0.0.1:5601
	  RewriteEngine on
	  RewriteCond %{DOCUMENT_ROOT}/%{REQUEST_FILENAME} !-f
	  RewriteRule .* http://127.0.0.1:5601%{REQUEST_URI} [P,QSA]

	</VirtualHost>

* Entrer ensuite la commande pour SELinux : 
	* /usr/sbin/setsebool httpd_can_network_connect 1 