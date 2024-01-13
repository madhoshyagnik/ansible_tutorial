##Learning Ansible

# Ansible learning module in depth. The host names, or public ips or dns names of aws instances are stored in ansible inventory file.
It should be done before running ad-hoc commands or running a ansible playbook.


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

___

