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
Handlers are executed at the end of a play.
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
