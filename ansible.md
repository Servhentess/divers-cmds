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


#Fichier de playbook pour ansible#
	
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


#Fichier de vars pour ansible#

Les variables doivent etre placées dans un repertoire host_vars situé au même endroit que l'inventaire et le playbook. Chaque fichier doit porter le nom du groupe defini dans l'inventaire.

>	ansible_host: 13.88.27.40
>	ansible_user: servhentess