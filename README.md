# Learning Ansible

### Ansible learning module in depth. The host names, or public ips or dns names of aws instances are stored in ansible inventory file. It should be done before running ad-hoc commands or running a ansible playbook.

### ad-hoc commands are a way to execute a module on remote host by just running a command, and not having to create a whole playbook to do so.

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


### Elevated ad - hoc commands. 

```bash
ansible all -m apt -a update_cache=true --become --ask-become-pass

-a -> supply arguments

-m -> include module, apt in this case

update_cache=true -> same as apt-get update on local
```
#### Result of above command:
```bash
BECOME password: 
65.0.29.224 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1705145705,
    "cache_updated": true,
    "changed": true
}

```


## Ansible Privilege Escalation Explanation

When Ansible is executing tasks on remote hosts with privilege escalation (`--become`), the `--ask-become-pass` option is used to prompt for the password of the remote user with elevated privileges (usually `root` or another specified user).

In your case, it's asking for your local Ubuntu password because Ansible is trying to escalate privileges on the remote host, and it needs to verify that you have the right to become another user (in this case, `root` or a specified user like `ubuntu`) on the target AWS instance.

Make sure to provide the correct password for the remote user with elevated privileges on the AWS instance when prompted. This is a security measure to ensure that only authorized users can perform privileged actions on the remote hosts.

___

