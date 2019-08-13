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
Facts are discovered automatically when a playbook runs. They have an impact on performace and can be cached or disabled. 

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
