### To setup Vagrant environment
cd to vagrant directory
```
cd vagrant
```  
Create and provision VMs
```
vagrant up
```   
Confirm VMs are up
```
vagrant status
```
Connect to ansible-server   
```
vagrant ssh ansible-server
```
su to ansible user
```
[vagrant@ansible-server ~]$ sudo su ansible
```  
Run ad-hoc ping module to check that nodes 1 and 2 reachable
```
[ansible@ansible-server vagrant]$ ansible all -i node1,node2, -m ping
node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
node2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```
