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
>	\- name: start update
>	  hosts: all
>	  become: true
>
>	  tasks:
>
>	  - name: update
>	    yum: name='*' state=latest
