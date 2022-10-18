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
- Install Ansible --> see docs: https://docs.ansible.com/ansible-core/devel/installation_guide/index.html
- Install Python3 --> required for running Ansible
- Check Ansible installation --> `ansible --version`
- Test Ansible --> `ansible localhost -m ping`

## Ansible - Inventory file
- List the hosts (machines) that you will be managing with Ansible
- https://docs.ansible.com/ansible/2.3/intro_inventory.html

## Ansible - Variables
- Variables can be used in many places
  - e.g. in Inventory file --> each host can define its own set of the same variables (each with different values)
  
```
[targets]

localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_user=myuser
other2.example.com     ansible_connection=ssh        ansible_user=myotheruser
```

## Ansible - Modules
- [Ansible Docs - Module Index](https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html)
- [Ansible Galaxy Modules](https://galaxy.ansible.com/)

## Ansible - Files.find module
```
ansible localhost -m find -a "paths=Downloads file_type=file"
```

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
