## 1 Playbook file for all instances.

```yaml (docker_project.yml)

- name: Docker install and configuration
  gather_facts: No
  any_errors_fatal: true
  hosts: _development 
  become: true
  tasks:

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

#####   Postgresql   #####

- name: Postgre Database configuration
  hosts: _ansible_postgresql
  become: true
  gather_facts: No
  any_errors_fatal: true
  # vars_files:
  #   - secret.yml
  vars:
    container_path: /home/ec2-user/postgresql
    container_name: cla_postgre
    image_name: clarusway/postgre

  tasks:
    - name: copy files to the postgresql node
      ansible.builtin.copy:
        src: /home/ec2-user/ansible-project/postgres/
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
        # env:
        #   POSTGRES_PASSWORD: "{{ password }}"
        env:
          POSTGRES_PASSWORD: "Pp123456789"
        volumes:
          - /db-data:/var/lib/postgresql/data
      register: container_info
    
    - name: print the container info
      debug:
        var: container_info

#####   Nodejs   #####

- name: Nodejs Server configuration
  hosts: _ansible_nodejs
  become: true
  gather_facts: No
  any_errors_fatal: true
  vars: 
    container_path: /home/ec2-user/nodejs
    container_name: cla_nodejs
    image_name: clarusway/nodejs

  tasks:

    - name: copy files to the nodejs node
      copy:
        src: /home/ec2-user/ansible-project/nodejs/
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

    - name: Launch Nodejs docker container
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

#####   React   #####

- name: React UI Server configuration
  hosts: _ansible_react
  become: true
  gather_facts: No
  any_errors_fatal: true
  vars:
    container_path: /home/ec2-user/react
    container_name: cla_react
    image_name: clarusway/react

  tasks:

    - name: copy files to the react node
      copy:
        src: /home/ec2-user/ansible-project/react/
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

```bash
ansible-playbook docker_project.yml
```


#### EĞER "Postgresql" İÇERİSİNDEKİ YORUMLARI KALDIRIRSAN ==>> "secret.yml" KULLANIYORSUN DEMEKTİR ==>> KOMUTLAR AŞAĞIDAKİ GİBİ OLUR

```secret.yml
password: Pp123456789
```
```bash
ansible-vault create secret.yml
ansible-playbook --ask-vault-pass docker_project.yml
```