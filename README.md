# Learning Ansible

### Ansible learning module in depth. The host names, or public ips or dns names of aws instances are stored in ansible inventory file. It should be done before running ad-hoc commands or running a ansible playbook.


## Ansible Ping Command Output

```bash
ansible all --private-key=/home/maddy/kube_key.pem -i inventory -m ping -u ubuntu

65.0.29.224 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

In this example, the Ansible ping module successfully executed on the host with IP address 65.0.29.224, and the response indicates that the host is reachable, running Python 3, and returned a "pong" response.
```
___


## Shortened the command 

```bash
ansible all --private-key=/home/maddy/kube_key.pem -i inventory -m ping -u ubuntu 
```
### to  

```bash 

ansible all -m ping  

```

### by creating an ansible.cfg file and putting these details in 

```bash ansible.cfg

[defaults]
inventory = inventory
private_key_file = /home/maddy/kube_key.pem
remote_user = ubuntu
```

___
list hosts defined in inventory

```bash  
ansible all --list-hosts 

```

___

### gathers information about the remote instance/host like cpu cores and things, very cool command

```bash
ansible all -m gather_facts --limit <65.0.29.224> //--limit <65.0.29.224> is optional
```
___



