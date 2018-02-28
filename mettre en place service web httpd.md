***Installer httpd :***
	* yum install httpd

***Verifier que cela fonctionne:***
	* systemctl start httpd.service

***Httpd doit redemarrer automatiquement quand l vm redemarre:***
	* systemctl enable httpd.service
	
***Verifier que cela fonctionne:***
	* shutdown -r 
	* ps aux | grep apache

***Où se situe les fichiers de log de httpd:***
	* ls -al /etc/httpd = ../../var/log/httpd

***Installer et lancer mysql***
	* wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
	* rpm -ivh mysql-community-release-el7-5.noarch.rpm
	* yum update
	* yum install mysql-server
	* systemctl start mysqld

***Verifier que cela fonctionne***
	* systemctl mysqld status

***Init mysql en auto***
	* service enable mysqld

***Verifier que cela fonctionne***
	* reboot
	* systemctl mysqld status

***Configurer mysql de base***
	* mysql_secure_installation

***Connexion a mysql en root***
	* mysql -u root -p

***Creer une BdD nommee ipme-javaapp***
	* create database ipme_javaapp;

***Creer un user login=ipme_login | passwd=ipme_pwd***
	* CREATE user 'ipme_login'@'localhost' identified by 'ipme_pwd';

***Donner tout les droits a user sur ipme-javaapp***
	* grant all on ipme_javaapp.* to 'ipme_login' identified by 'ipme_pwd';

***Verifier que cela fonctionne***
	* mysql -u ipme_login -p

***Faire en sorte que l'on puisse administrer la BdD de l'exterieur***
	* entrez les infos de connexion dans workbench

***Verifier que cela fonctionne***
	* ba tester la connexion sur workbench
		
***Liste les ports en ecoute***
	* netstat -atn | grep LISTEN

***Afficher page accueil apache***
	* curl localhost

***Mysqldump***
	* mysqldump --host="localhost" --user="ipme_login" --password="ipme_pwd" ipme_javaapp > budb.sql

***Rediriger les erreurs sur /dev/null***
	* macommande > /dev/null 2>&1

***Creer tache cron***
	* crontab -e
	* 00 14 * * *  chemin vers .sh (00 14 = minute heure jour_dans_mois mois jour)
	* service crond restart