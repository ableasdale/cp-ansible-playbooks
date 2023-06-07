# cp-ansible-playbooks
My CP-Ansible Playbooks

## Installation

Follow these steps:

```bash
pip3 install ansible
python3.11 -m pip install --upgrade pip
ansible-galaxy collection install git+https://github.com/confluentinc/cp-ansible.git
ansible-galaxy collection install community.general
```

## What instances should I use for my target machines?

![[img/compatible-instance.png]]

### Useful reading

- https://docs.confluent.io/platform/current/installation/system-requirements.html#operating-systems