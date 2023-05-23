## Playbook with roles solution.

```bash (pwd : /home/ec2-user/ansible)
mkdir roles && cd roles
ansible-galaxy init docker
ansible-galaxy init postgre
ansible-galaxy init nodejs
ansible-galaxy init react
```

  ### "docker" kurulumu , 3 makinaya da kurulacak ###
```yaml (docker için role)  (pwd : /home/ec2-user/ansible/roles/docker/tasks/main.yml)
---
    - name: upgrade all packages
      ansible.builtin.yum: 
        name: '*'
        state: latest

    - name: Remove docker if installed from CentOS repo
      ansible.builtin.yum:
        name:
          - docker
          - docker-client
          - docker-client-latest
          - docker-common
          - docker-latest
          - docker-latest-logrotate
          - docker-logrotate
          - docker-engine
        state: removed

    - name: Install yum utils
      ansible.builtin.yum:
        name: "yum-utils"
        state: latest

    - name: Add Docker repo
      ansible.builtin.get_url:
        url: https://download.docker.com/linux/centos/docker-ce.repo
        dest: /etc/yum.repos.d/docker-ce.repo

    - name: Install Docker
      ansible.builtin.package:
        name: docker-ce
        state: latest

    - name: Add user ec2-user to docker group
      ansible.builtin.user:
        name: ec2-user
        groups: docker
        append: yes

    - name: Start Docker service
      ansible.builtin.service:
        name: docker
        state: started
        enabled: yes
```


  ### postgre ###

```yaml (postgre için role)  (pwd : /home/ec2-user/ansible/roles/postgre/tasks/main.yml)
    - name: copy files to the postgresql node
      ansible.builtin.copy:
        src: postgres/ # write only file name
        dest: "{{ container_path }}"

    - name: remove {{ container_name }} container
      community.docker.docker_container:
        name: "{{ container_name }}"
        state: absent
        force_kill: true

    - name: remove "{{ image_name }}" image
      community.docker.docker_image:
        name: "{{ image_name }}"
        state: absent

    - name: build container image
      community.docker.docker_image:
        name: "{{ image_name }}"
        build:
          path: "{{ container_path }}"
        source: build
        state: present

    - name: Launch postgresql docker container
      community.docker.docker_container:
        name: "{{ container_name }}"
        image: "{{ image_name }}"
        state: started
        ports: 
        - "5432:5432"
        env:
          POSTGRES_PASSWORD: "Pp123456789"
        volumes:
          - /db-data:/var/lib/postgresql/data
      register: container_info
    
    - name: print the container info
      debug:
        var: container_info
```

- Copy /home/ec2-user/ansible-project/postgres folder to /home/ec2-user/ansible/roles/postgre/files.

- Copy these variables to /home/ec2-user/ansible/roles/postgre/vars/main.yml.

```
container_path: /home/ec2-user/postgresql
container_name: cla_postgre
image_name: clarusway/postgre
```

  ### nodejs ###

```yaml (nodejs için role)  (pwd : /home/ec2-user/ansible/roles/nodejs/tasks/main.yml)
    - name: copy files to the nodejs node
      copy:
        src: nodejs/
        dest: "{{ container_path }}"

    - name: remove {{ container_name }} container
      community.docker.docker_container:
        name: "{{ container_name }}"
        state: absent
        force_kill: true

    - name: remove "{{ image_name }}" image
      community.docker.docker_image:
        name: "{{ image_name }}"
        state: absent
  
    - name: build container image
      community.docker.docker_image:
        name: "{{ image_name }}"
        build:
          path: "{{ container_path }}"
        source: build
        state: present

    - name: Launch nodejs docker container
      community.docker.docker_container:
        name: "{{ container_name }}"
        image: "{{ image_name }}"
        state: started
        ports: 
        - "5000:5000"
      register: container_info
    
    - name: print the container info
      debug:
        var: container_info
```

- Copy /home/ec2-user/ansible-project/nodejs folder to /home/ec2-user/ansible/roles/nodejs/files.


- Copy these variables to /home/ec2-user/ansible/roles/nodejs/vars/main.yml.

```
container_path: /home/ec2-user/nodejs
container_name: cla_nodejs
image_name: clarusway/nodejs
```

  ### react ###

```yaml (react için role)  (pwd : /home/ec2-user/ansible/roles/react/tasks/main.yml)
    - name: copy files to the react node
      copy:
        src: react/
        dest: "{{ container_path }}"

    - name: remove {{ container_name }} container
      community.docker.docker_container:
        name: "{{ container_name }}"
        state: absent
        force_kill: true

    - name: remove "{{ image_name }}" image
      community.docker.docker_image:
        name: "{{ image_name }}"
        state: absent
  
    - name: build container image
      community.docker.docker_image:
        name: "{{ image_name }}"
        build:
          path: "{{ container_path }}"
        source: build
        state: present

    - name: Launch react docker container
      community.docker.docker_container:
        name: "{{ container_name }}"
        image: "{{ image_name }}"
        state: started
        ports: 
        - "3000:3000"
      register: container_info
    
    - name: print the container info
      debug:
        var: container_info
```

- Copy /home/ec2-user/ansible-project/react folder to /home/ec2-user/ansible/roles/react/files.

- Copy these variables to /home/ec2-user/ansible/roles/react/vars/main.yml.

```
container_path: /home/ec2-user/react
container_name: cla_react
image_name: clarusway/react
```

- Update the ansible.cfg file in /home/ec2-user/ansible/. Add roles_path as below.

```cfg
roles_path= /home/ec2-user/ansible/roles
```

  ###  4 role'ü (docker, postgre, nodejs, react) çalıştıracak playbook ###

```yaml  (play-role.yml) (pwd: /home/ec2-user/ansible)

- name: Docker install and configuration
  hosts: _development
  become: true
  roles: 
    - docker
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
ansible-playbook play-role.yml
```