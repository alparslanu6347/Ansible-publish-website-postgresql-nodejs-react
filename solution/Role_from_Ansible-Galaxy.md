## Using role from ansible-galaxy.

# docker için galaxy'den role (geerlingguy.docker) indiriyorum, diğer 3 role benim hazırladığım role.

```bash (pwd : /home/ec2-user/ansible)
ansible-galaxy install geerlingguy.docker
```

- Create a new play-newrole.yml

```yaml
- name: Docker install and configuration
  hosts: _development
  become: true
  roles: 
    - geerlingguy.docker
    
- name: Postgre Database configuration
  hosts: _ansible_postgresql
  become: true
  roles:
    - postgre

- name: Nodejs server configuration
  hosts: _ansible_nodejs
  become: true
  roles:
    - nodejs

- name: React UI Server configuration
  hosts: _ansible_react
  become: true
  roles:
    - react
```

- Execute the follow.

```bash (pwd : /home/ec2-user/ansible)
ansible-playbook play-newrole.yml
```