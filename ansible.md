#Fichier d'inventaire pour ansible#

>	front-1 ansible_host=13.64.66.207
>	front-2 ansible_host=13.64.68.56
>	back-1 ansible_host=13.88.27.40
>
>	[front_end]
>	front-1
>	front-2
>
>	[back_end]
>	back-1
>
>	[server:children]
>	front_end
>	back_end
>	[server:vars]
>	ansible_user=servhentess


#Fichier de playbook pour ansible (install)#
	
>	\-\-\-
>	- name: Update n install Lynx
>	  hosts: all
>	  become: true
>
>	  tasks:
>
>	  - name: update
>	    yum: name='*' state=latest
>
>	  - name: lynx
>	    yum: name=lynx state=latest
>
>	\- name: Install httpd
>	  hosts: front_end
>	  become: true
>
>	  tasks:
>
>	  - name: httpd
>	    yum: name=httpd state=latest
>
>	  - name: httpd autostart
>	    service: name=httpd enabled=yes
>
>	  - name: httpd started
>	    service: name=httpd state=started


#Fichier de vars pour l'inventaire d'ansible#

Les variables doivent etre placées dans un repertoire host_vars situé au même endroit que l'inventaire et le playbook. Chaque fichier doit porter le nom du groupe defini dans l'inventaire.

>	ansible_host: 13.88.27.40
>	ansible_user: servhentess


#Fichier de playbook pour ansible (uninstall)#

>	\-\-\-
>	- name: Delete Lynx
>	  hosts: all
>	  become: true
>
>	  tasks:
>
>	  - name: lynx
>	    yum: name=lynx state=removed
>
>	\- name: Delete httpd
>	  hosts: front_end
>	  become: true
>
>	  tasks:
>
>	  - name: httpd
>	    yum: name=httpd state=removed


#Fichier de playbook pour ansible avec variable#

>	\-\-\-
>	- name: variable
>	  hosts: all
>	  vars:
>	    variable_in: interne
>	  vars_files:
>	    - variablestest.yml
>
>	  tasks:
>
>	  - name: afficher variable
>	    debug: var=variable_in
>
>	  - name: afficher variable ext
>	    debug: var=deuxieme_var

* vars : On declare les variables interne à ce niveau
* vars_files : on declare ici le/les fichier(s) contenant les variables

#Fichier de var pour playbook d'ancible#

>	deuxieme_var: externe


#Fichier pour la recuperation de donnes dans des variables#

>	\-\-\-
>	- name: variable
>	  hosts: back_end
>
>	  tasks:
>
>	  - name: recup info user
>	    command: whoami
>	    register: login
>
>	  - name: afficher resultat login
>	    debug: var=login
>
>	  - name: afficher le nom du user depuis login
>	    debug: var=login.stdout_lines

* command : Permet d'executer une commande linux pur
* register : Commande qui demande l'enregistrement du resultat de la commande entré dans "command"
* Variante pour afficher un message avec le resultat :

>	debug: msg="logger en tant que {{login.stdout_lines}}"

* Ansible possede des variables et fonction native comme :

>	debug: msg={{ ansible_eth0["ipv4"]["address"] }}

#Diverse commandes#

* Afficher l'IP publique

>	\- name: get public IP
>	  ipify_facts:
>	  register: public_ip
>
>	\- name: output
>	  debug: msg="{{ public_ip }}"

* Recuperer des infos sur la machine cliente

>	ansible all -i /etc/ansible/catalogue.ini -m setup -a 'filter=ansible_distribution*'

* "ansible_distribution" donne quel OS est installé, "ansible_processor" donnerait le type de processeur

* Il est possible d'obtenir des infos via les facts des machines clientes :
  * Creer un dossier /etc/ansible/fact.d
  * Creer un ou des fichiers .fact au format yml ou ini
  * Exemple de fichier :
  
>	[fact]
>	ma_variable= back_1

  * Ces fichiers s'appelent avec une commande de type :

>	ansible all -i /etc/ansible/catalogue.ini -m setup -a 'filter=ansible_local'