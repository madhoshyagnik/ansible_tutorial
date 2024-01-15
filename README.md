## Learning Ansible

```bash
ansible all -m apt -a "name=vim-nox" --become --ask-become-pass

```
### So cool thing about ansible is vim can be installed on 1000 aws instances in a single command, just put the server public ip-address in inventory file and run the above comman

### Ansible learning module in depth. The host names, or public ips or dns names of aws instances are stored in ansible inventory file. It should be done before running ad-hoc commands or running a ansible playbook.

### ad-hoc commands are a way to execute a module on remote host by just running a command, and not having to create a whole playbook to do so. Facts are gathered everytime a playbook is run.

### Ansible Ping Command Output

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


### Shortened the command 

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
### Result of above command:
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


### Ansible Privilege Escalation Explanation

When Ansible is executing tasks on remote hosts with privilege escalation (`--become`), the `--ask-become-pass` option is used to prompt for the password of the remote user with elevated privileges (usually `root` or another specified user).

In your case, it's asking for your local Ubuntu password because Ansible is trying to escalate privileges on the remote host, and it needs to verify that you have the right to become another user (in this case, `root` or a specified user like `ubuntu`) on the target AWS instance.

Make sure to provide the correct password for the remote user with elevated privileges on the AWS instance when prompted. This is a security measure to ensure that only authorized users can perform privileged actions on the remote hosts.

___

### Installing vim on aws ubuntu instance

```bash
ansible all -m apt -a "name=vim-nox" --become --ask-become-pass

```
### So cool thing about ansible is vim can be installed on 1000 aws instances in a single command, just put the server public ip-address in inventory file and run the above command. 

___

### This is what it looks like when a package is already installed on ec2 and we still try to install it with ansible

```bash
ansible all -m apt -a "name=snapd" --become --ask-become-pass
BECOME password: 

Output: 

65.0.29.224 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1705146861,
    "cache_updated": false,
    "changed": false
}

```
___

### but updating the package does not work (state=latest updates it if necessary), as it's already latest

```bash

ansible all -m apt -a "name=snapd state=latest" --become --ask-become-pass


BECOME password: 
65.0.29.224 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "cache_update_time": 1705146861,
    "cache_updated": false,
    "changed": false
}


```

___

### Upgrade all packages on all hosts

```bash
ansible all -m apt -a "upgrade=yes" --become --ask-become-pass

Output:

BECOME password: 
65.0.29.224 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "msg": "Reading package lists...\nBuilding dependency tree...\nReading state information...\nCalculating upgrade...\n0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.\n",
    "stderr": "",``
    "stderr_lines": [],
    "stdout": "Reading package lists...\nBuilding dependency tree...\nReading state information...\nCalculating upgrade...\n0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.\n",
    "stdout_lines": [
        "Reading package lists...",
        "Building dependency tree...",
        "Reading state information...",
        "Calculating upgrade...",
        "0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded."
    ]
}


```

### Running first playbook

### install_apache.yml

```bash
---
- hosts: all
  become: true
  tasks:
    - name: install apache2 package
      apt:
        name: apache2

```

### Running command and output:

```bash
ansible-playbook --ask-become-pass install_apache.yml

Output:


BECOME password: 

PLAY [all] ******************************************************************************************

TASK [Gathering Facts] ******************************************************************************
ok: [65.0.29.224]

TASK [install apache2 package] **********************************************************************
changed: [65.0.29.224]

PLAY RECAP ******************************************************************************************
65.0.29.224                : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


```

### Running the playbook again doesn't install apache again as it's already installed, we get changed = 0 this time

```bash
ansible-playbook --ask-become-pass install_apache.yml


Output:


BECOME password: 

PLAY [all] ******************************************************************************************

TASK [Gathering Facts] ******************************************************************************
ok: [65.0.29.224]

TASK [install apache2 package] **********************************************************************
ok: [65.0.29.224]

PLAY RECAP ******************************************************************************************
65.0.29.224                : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


```

___

### Final playbook looks like this

```bash
---

- hosts: all
  become: true
  tasks:

  - name: update repository index
    apt:
      update_cache: yes

  - name: install apache2 package
    apt:
      name: apache2
      state: latest

  - name: add php support for apache2
    apt:
      name: libapache2-mod-php
      state: latest


```

___

### This is the syntax when we are not running the playbook for different distributions

```bash

---

- hosts: all
  become: true
  tasks:

  - name: update repository index
    apt:
      update_cache: yes 
    when: ansible_distribution == "Ubuntu" 

  - name: install apache2 package
    apt:
      name: apache2
      state: latest
    when: ansible_distribution == "Ubuntu"

  - name: add php support for apache2
    apt:
      name: libapache2-mod-php
      state: latest
    when: ansible_distribution == "Ubuntu"


```

___

### Doing gather facts for an amazon linux instance, it's ip is 65.0.184.250 and it's in same inventory. need to define -u ec2-user in command as default remote_user is ubuntu in inventory file.

```bash
ansible all -m gather_facts --limit 65.0.184.250 -u ec2-user | grep ansible_distribution


output:

[WARNING]: Platform linux on host 65.0.184.250 is using the discovered Python
interpreter at /usr/bin/python3.9, but future installation of another Python
interpreter could change the meaning of that path. See
https://docs.ansible.com/ansible-
core/2.16/reference_appendices/interpreter_discovery.html for more information.
        "ansible_distribution": "Amazon",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/os-release",
        "ansible_distribution_file_variety": "Amazon",
        "ansible_distribution_major_version": "2023",
        "ansible_distribution_minor_version": "NA",
        "ansible_distribution_release": "NA",
        "ansible_distribution_version": "2023",


```

### Syntax for targeting a specific distributions like say CentOS and version 8.2, the below task will still fail because the task is using apt, and centos uses yum

```bash
- name: update repository index
    apt:
      update_cache: yes 
    when: ansible_distribution == "CentOS" and ansible_distribution_version == "8.2" 

```
___


### Using tags
```bash 
ansible-playbook --tags amazon --ask-become-pass site.yml

```

### Using multiple tags in one command
```bash
ansible-playbook --tags "apache,db" --ask-become-pass site

```
___


### copying files to remote servers 

```bash

   - name: copy default html file for site
     tags: apache,apache2,httpd
     copy:
       src: default_site.html
       dest: /var/www/html/index.html
       owner: root
       group: root
       mode: 0644

```

___


### installing unzip and installing terraform on remote ubuntu server

```bash
 - hosts: workstations
   become: true
   tasks:
   
   - name: install unzip 
     package: 
       name: unzip

   - name: install terraform
     unarchive: 
       src: https://releases.hashicorp.com/terraform/1.6.6/terraform_1.6.6_darwin_amd64.zip
       dest: /usr/local/bin
       remote_src: yes
       mode: 0755
       owner: root
       group: root

```

___

### running an ansible playbook with elevated privileges 

```bash
ansible-playbook  --ask-become-pass site.yml

```

___

#### change email address, save that change to variable httpd, restart httpd task will only run if the variable has changed, if not it won't run.


```bash 
   - name: change admin email
     tags: amazon,httpd,apache
     lineinfile: 
       path: /etc/httpd/conf/httpd.conf
       regexp: '^ServerAdmin'
       line: ServerAdmin newemailaddress.gmail.com
     when: ansible_distribution == "Amazon"
     register: httpd

   - name: restart httpd (amazon)
     tags: apache,amazon,httpd
     service: 
       name: httpd
       state: restarted
     when: httpd.changed

``` 
___
