# CP-Ansible Playbooks

My [CP-Ansible](https://docs.confluent.io/ansible/current/overview.html) Playbooks

## Installation

Follow these steps to install CP-Ansible (Ubuntu):

```bash
pip3 install ansible
python3.11 -m pip install --upgrade pip
ansible-galaxy collection install git+https://github.com/confluentinc/cp-ansible.git
ansible-galaxy collection install community.general
```

## What instances should I use for my target machines?

These playbooks have been tested with the latest version of cp-ansible (7.4.0) and Ubuntu 20.04 on AWS:

![Ubuntu 20.04](img/compatible-instance.png)

## Useful reading

- https://docs.confluent.io/platform/current/installation/system-requirements.html#operating-systems