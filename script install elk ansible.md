---
- name: Update vm
  hosts: all
  become: true

  tasks:

  - name: Updating
    yum:
      name: '*'
      state: latest

- name: Install httpd
  hosts: all
  become: true

  tasks:

  - name: Httpd installing
    yum:
      name: httpd
      state: latest

  - name: Httpd auto-start
    service:
      name=httpd
      enabled=yes

  - name: Httpd start
    service:
      name=httpd
      state=started

- name: Install java
  hosts: all
  become: true

  tasks:

  - name: Download and install open-jdk
    yum:
      name: java
      state: latest

- name: Install ELK (Elastic, Logstash, Kibana)
  hosts: all
  become: true

  tasks:

  - name: Download and install Elasticsearch
    yum:
      name: https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.2.rpm
      state: present

  - name: Auto-start elasticsearch
    service:
      name=elasticsearch
      enabled=yes

 - name: Start elasticsearch
    service:
      name=elasticsearch
      state=started

  - name: Change jvm.options 'Xms1g' file for elasticsearch
    replace:
      path: /etc/elasticsearch/jvm.options
      regexp: '-Xms1g'
      replace: '-Xms512m'

  - name: Change jvm.options 'Xmx1g' file for elasticsearch
    replace:
      path: /etc/elasticsearch/jvm.options
      regexp: '-Xmx1g'
      replace: '-Xmx512m'

  - name: Change elasticsearch.yml 'network.host' for elasticsearch
    replace:
      path: /etc/elasticsearch/elasticsearch.yml
      regexp: '#network.host: 192.168.0.1'
      replace: 'network.host: localhost'

- name: Install kibana
  hosts: all
  become: true

  tasks:

  - name: Download and install kibana
    yum:
      name: https://artifacts.elastic.co/downloads/kibana/kibana-6.2.2-x86_64.rpm
      state: present

  - name: Auto-start kibana
    service:
      name=kibana
      enabled=yes

  - name: Start kibana
    service:
      name=kibana
      state=started

- name: Install Logstash
  hosts: all
  become: true

  tasks:

  - name: Download and install Logstash
    yum:
      name: https://artifacts.elastic.co/downloads/logstash/logstash-6.2.2.rpm
      state: present

  - name: Change jvm.options '-Xmx1g' file for logstash
    replace:
      path: /etc/logstash/jvm.options
      regexp: '-Xmx1g'
      replace: '-Xmx256m'

  - name: Copy file 'kibana.conf' in /etc/httpd/conf.d
    copy:
      src: /etc/ansible/mes_scripts/reverse-proxy/kibana.conf
      dest: /etc/httpd/conf.d

  - name: Copy file 'logstash-apache.conf' in /etc/logstash/conf.d
    copy:
      src: /etc/ansible/mes_scripts/logstash_file/logstash-apache.conf
      dest: /etc/logstash/conf.d

  - name: Change SELinux
    command: /usr/sbin/setsebool -P httpd_can_network_connect 1

- name: Restart VM
  hosts: all
  become: true

  tasks:

  - name: Send debug msg for reboot system
    debug:
      msg: "System reboot, plz wait"

  - name: Execute shutdown -r now
    command: shutdown -r
    async: 1
    poll: 0
    ignore_errors: true

  - name: Wait for system boot up
    local_action: wait_for host="{{ ansible_ssh_host }}" port=22 state=started delay=15 timeout=300
    become: false

  - name: Send debug msg for system is ok
    debug:
      msg: "I will be back !!! Try to exec logstash (wait 1 munite for system init)"

  - name: Wait for execute logstash
    pause:
      minutes: 2

- name: Execute logstash
  hosts: all
  become: true

  tasks:

  - name: Execute /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/logstash-apache.conf
    command: /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/logstash-apache.conf

