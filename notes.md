# Ansible - ex407 notes
My notes from the Linux Academy ex407 course

## Understanding Core Components of Ansible

### Inventories
Can be provided with ini or yaml files  
Can contain hostnames and also host or group level variable  
Default inventory file is in /etc/ansible/hosts  

### Modules
Used for running tasks  
Take parameters  
Return json  

### Variables
Numbers, letters, underscores  
Should always start with a letter  
Can be used in group, host and in playbook  

### Facts
Provide information about a specified host (IP address, kernel version, hardware, etc)  
Facts are discovered automatically when a playbook runs. They have an impact on performace and can be cached or disabled

### Plays and Playbooks
Goal of play is to map roles to a group of hosts  
A play can use one or more modules  
A playbook is a series of plays  

### Config files
ANSIBLE_CONFIG (env variable)  
ansible.cfg (in current directory)  
ansible.cfg (in home directory)  
/etc/ansible/ansible.cfg  

### Quick ping check
```
ansible -i /home/ansible/inventory host_name -m ping
```
## Run ad-hoc Ansible Commands
Common modules:  
```ping``` verifies host is up, no parameters required  
```setup``` gathers facts, no parameters required  
```yum``` package manager, requires name of package and state  
```service``` controls daemons, requires name and state of service  
```user``` manipulate users, reqires user name  
```copy``` copies files, requires source and destination  
```file``` working with files, requires path  
```git``` interacts with repos, requires repo and destination

Example:  
```ansible HOSTNAME -i INVENTORY -m MODULE -a ARGUMENTS```  
```ansible host1 -i inv.yaml -b -m yum -a "name=vim state=latest"```  

Flags:  
```-b``` become  
```-m``` module  
```-a``` arguments  

## Inventory Management
Can be ini (default) or yaml format  

ini example:
```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```
yaml example:
```
all:
  hosts:
    mail.example.com:
  children:
    webservers:
      hosts:
        foo.example.com:
        bar.example.com:
    dbservers:
      hosts:
        one.example.com:
        two.example.com:
        three.example.com:
```
Variables can be stored in inventory files but it isn't recommended. Use host_vars and group_vars instead.  
Variables example:
Edit group_vars file to look like this:
```
log : /var/log/messages
```
Then:
```
ansible -b -i hosts webservers -a "tail {{logs}}"
```
This will will use the variable set in group_vars and tail the messages log for all servers in the webservers group.  

```ansible-inventory``` can be used to display the inventory.
```
ansible-inventory -i /home/ansible/inv --list
```
##  Create Ansible Plays and Playbooks
### Example Playbook
```
---
- hosts: webservers
  remote_user: yourname
  tasks:
    - name: ensure httpd is installed
      yum:
        name: httpd
        state: latest
    - name: ensure httpd is started
      service:
        name: nginx
        state: started
```
### Use Variables to Retrieve the Results of Running Commands
```
---
- hosts: local
  tasks:
    - name: create file
      file:
        path: /tmp/newfile
        state: touch
      register: output
    - debug: msg="output is: {{output}}"
```
### Use Conditionals to Control Play Execution
#### Handlers
Executed at the end of a play
```
---
- hosts: all
  become: yes
  handlers:
    - name: restart httpd
      service:
        name: httpd
        state: restarted
        listen: "restart web"
  tasks:
    - name: update config
      copy:
        src: ../files/index.html
        dest: /var/www/index.html
      notify:
        restart web
```
#### when
Conditional   
```
---
- hosts: all
  tasks:
  - name: echo message
    shell: echo "This is Red Hat"
    when: ansible_facts['os_family'] == "RedHat"  
```
#### with_items
Loop
```
---
- hosts: all
  tasks:
  - name: add users
    user:
      name: {{item}}
    with_items:
      - andy
      - barry
      - dave
```
### Error Handling
Ignore, define failure conditions, defining "changed", blocks
#### Ignore
```
---
- hosts: all
  tasks:
  - name: this will not be counted as a failure
    command: /bin/false
    ignore_errors: yes
```
#### Blocks
Similar to try-catch
```
- hosts: all
  tasks:
  - name: Handle the error
    block:
      - debug:
          msg: 'I execute normally'
      - name: i force a failure
        command: /bin/false
      - debug:
          msg: 'I never execute, due to the above task failing, :-('
    rescue:
      - debug:
          msg: 'I caught an error, can do stuff here to fix it, :-)'
```
### Tags
```
- hosts: all
  tasks:
  - yum:
      name: "{{ item }}"
      state: present
    loop:
    - httpd
    - memcached
    tags:
    - packages

  - template:
      src: templates/src.j2
      dest: /etc/foo.conf
    tags:
    - configuration
```
Run playbook with: ```ansible-playbook example.yml --tags "packages"``` to only run the yum task. ```--skip-tags``` can be used to skip specified tags.

## Create and Use Templates to Create Customised Configuration Files
Example play:
```
- hosts:all
  tasks:
  - name: Template a file to /home/ansible/info.txt
    template:
      src: /templates/info.j2
      dest: /home/ansible/info.txt
```
info.j2 template:
```
hostname is: {{ ansible_hostname }}
os family is: {{ ansible_os_family }}
```
## Work with Ansible Variables and Facts
### Variables
Can be defined in playbook:
```
hosts: localhost
 vars:
   http_port: 80
 tasks:
 - name: print variable
   debug:
     msg: http port is {{ http_port }}
```

Example of accessing data:
```
{{ ansible_facts["eth0"]["ipv4"]["address"] }}
```

Magic variables:
``` hostvars, groups, group_names, and inventory_hostname```  
Example:
```  
{{ hostvars['test.example.com']['ansible_facts']['distribution'] }}
```
Can use ```vars_files``` to load variables from a file. Example:  
playbook.yml
```
- hosts: localhost
  become: true
  vars_files:
    - /home/ansible/vars/users.yml
  tasks:
    - name: adding users
      user:
        name: "{{ item.name }}"
        state: present
      with_items: "{{ users }}"
```
/home/ansible/vars/users.yml:
```
users:
  - name: testuser1
  - name: testuser2
```
### Facts
Get all facts for localhost:
```
ansible localhost -m setup
```
Filter for ipv4 info:
```
ansible localhost -m setup -a "filter=*ipv4"
```
facts.d allows you to set custom facts. For example, create ```/etc/ansible/facts.d/prefs.fact``` and add add some custom facts:
```
[location]
type=physical
datacentre=london1
```
Then run the setup module and filter for ansible_local. 
```
ansible localhost -m setup -a "filter=ansible_local"
```
