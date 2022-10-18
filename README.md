# Ansible - Personal Learning Notes

## Ansible - Resources
- https://docs.ansible.com/
- https://galaxy.ansible.com/
- https://www.linkedin.com/learning/learning-ansible-2020/
- https://www.redhat.com/en/technologies/management/ansible
- https://www.redhat.com/en/technologies/management/ansible/azure

## Ansible is a task execution engine
- **Control Node** executes tasks both on a local system (control node itself) and on remote systems
- Open source - *although it has been bought by Red Hat*
- https://github.com/ansible/ansible

## Ansible - History
- 2012 - Michael DeHaan releases Ansible
- 2015 - Red Hat buys Ansible Inc. - *"core" Ansible is still open source, "wrapper" products are paid*

## Ansible - Architecture
- Clientless (SSH) == no need to install client software on remote systems
- Ansible is written in Python
- Playbooks are written in YAML

## Ansible - Benefits
- Simple (human-readable), easy to use, flexible

## Fleet Management - Red Hat Ansible Tower
- Graphical UX
- Low-cost fleet management
- Not free software

## Fleet Management - AWX
- Graphical UX
- Free software
- Open source

## Ansible - Life before Ansible
- Manual management (by hand)
- Server tools development

## Ansible - Automation VS Orchestration
- **Automation** --> refers to a single task
- **Orchestration** --> refers to management of many automated tasks
  - ... often a complicated ordering with dependencies

## Ansible - Installation
- See docs: https://docs.ansible.com/ansible-core/devel/installation_guide/index.html

## Ansible - First playbook - Localhost
**first.yml**
```yml
---
  - name: "My first play"
    hosts: localhost

    tasks:

      - name: "test reachability"
        ping:

      - name: "install stress"
        apt:
          name: stress
          state: present
```
```bash
ansible-playbook first.yml -v
```

## Ansible - Second playbook - Localhost
**second.yml**
```yml
---
  - name: "Some name"
    hosts: localhost

    tasks:

      - name: "teach reachability"
        ping:

      - name: "find module"
        find:
          paths: ~/
          file_type: directory
```
```bash
ansible-playbook second.yml -v
```
## Ansible - Third playbook - Orchestration

- [How To Set Up Ansible Inventories](https://www.digitalocean.com/community/tutorials/how-to-set-up-ansible-inventories)

Create custom **inventory** file
```
[logicservers]
server1 ansible_host=127.0.0.1 ansible_connection=local deprecation_warnings=false
server2 ansible_host=127.0.0.2 ansible_connection=local INTERPRETER_PYTHON=auto_silent
server3 ansible_host=127.0.0.3 ansible_connection=local
server4 ansible_host=127.0.0.4 ansible_connection=local
```
Make sure that custom **inventory** file is OK
```bash
ansible-inventory -i inventory --list
```
Create **third.yml** file
```yml
---
  - name: "Orchestration Example"
    hosts: logicservers
    serial: 1

    tasks:

      - name: "Shutdown Server"
        debug:
          msg: "Shutdown {{ inventory_hostname }}"

      - name: "Upgrade Firmware"
        debug:
          msg: "Upgrade {{ inventory_hostname }}"

      - name: "Start Server"
        debug:
          msg: "Start {{ inventory_hostname }}"
```
Run **third.yml** playbook with a custom inventory file
```bash
ansible-playbook -i -inventory third.yml -v
```

## Ansible - Fourth playbook - Change Handler

- [How To Set Up Ansible Inventories](https://www.digitalocean.com/community/tutorials/how-to-set-up-ansible-inventories)

Create custom **inventory** file
```
[logicservers]
server1 ansible_host=127.0.0.1 ansible_connection=local deprecation_warnings=false
server2 ansible_host=127.0.0.2 ansible_connection=local INTERPRETER_PYTHON=auto_silent
server3 ansible_host=127.0.0.3 ansible_connection=local
server4 ansible_host=127.0.0.4 ansible_connection=local
```
Make sure that custom **inventory** file is OK
```bash
ansible-inventory -i inventory --list
```
Create **fourth.yml** file
```yml
---
  - name: "React with Change Example - with a Change Handler"
    hosts: logicservers
    serial: 1

    tasks:

      - name: "Install nginx"
        debug:
          msg: "Install nginx on {{ inventory_hostname }}"

      - name: "Upgrade nginx"
        debug:
          msg: "Upgrade nginx on {{ inventory_hostname }}"

      - name: "Configure nginx"
        debug:
          msg: "Configure nginx on {{ inventory_hostname }}"
        notify: restart nginx
        changed_when: True

      - name: "Verify nginx"
        debug:
          msg: "Verify nginx on {{ inventory_hostname }}"

    handlers:
    - name: restart nginx
      debug:
        msg: "CALLED HANDLER FOR RESTART"
```
Run **fourth.yml** playbook with a custom inventory file
```bash
ansible-playbook -i -inventory fourth.yml -v
```

## Ansible - Fifth playbook - Free Strategy and Forks

- [How To Set Up Ansible Inventories](https://www.digitalocean.com/community/tutorials/how-to-set-up-ansible-inventories)
- [Controlling playbook execution: strategies and more](https://docs.ansible.com/ansible/latest/user_guide/playbooks_strategies.html)

Create custom **inventory** file
```
[logicservers]
server1 ansible_host=127.0.0.1 ansible_connection=local deprecation_warnings=false
server2 ansible_host=127.0.0.2 ansible_connection=local INTERPRETER_PYTHON=auto_silent
server3 ansible_host=127.0.0.3 ansible_connection=local
server4 ansible_host=127.0.0.4 ansible_connection=local
```
Make sure that custom **inventory** file is OK
```bash
ansible-inventory -i inventory --list
```
Create **fifth.yml** file
```yml
---
  - name: "Free Strategy and Forks"
    hosts: logicservers
    strategy: free

    tasks:

      - name: "Install nginx"
        debug:
          msg: "Install nginx on {{ inventory_hostname }}"

      - name: "Upgrade nginx"
        debug:
          msg: "Upgrade nginx on {{ inventory_hostname }}"

      - name: "Configure nginx"
        debug:
          msg: "Configure nginx on {{ inventory_hostname }}"
        notify: restart nginx
        changed_when: True

      - name: "Verify nginx"
        debug:
          msg: "Verify nginx on {{ inventory_hostname }}"

    handlers:
    - name: restart nginx
      debug:
        msg: "CALLED HANDLER FOR RESTART"
```
Run **fifth.yml** playbook with a custom inventory file
```bash
ansible-playbook -i -inventory fifth.yml -v -f 4
```
