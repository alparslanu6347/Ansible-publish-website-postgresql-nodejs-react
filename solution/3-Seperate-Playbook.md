### postgres

```bash (pwd : /home/ec2-user/ansible)
ansible-vault create secret.yml
```

```secret.yml
password: Pp123456789
```

``` (pwd : /home/ec2-user/ansible)
ansible-playbook --ask-vault-pass docker_postgre.yml
```

```yaml (docker_postgre.yml) (pwd : /home/ec2-user/ansible)

- name: Install docker
  gather_facts: No  # bu instance bilgilerini toplama . In Ansible, I can use gather_facts: yes to collect info about my hosts. As gather_facts collects a lot of information, it takes quite a while. ansible 3.ders 
  any_errors_fatal: true  # BİR YERDE HATA ALDIĞINDA playbook'u KOMPLE DURDUR. ansible 4.ders
  hosts: _ansible_postgresql
  become: true
  vars_files:
    - secret.yml  # Tek playbook'da env: POSTGRES_PASSWORD: "Pp123456789"
  tasks:  # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html#parameter-state , https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html#parameter-name

    - name: upgrade all packages
      ansible.builtin.yum: 
        name: '*'
        state: latest
  
    # we may need to uninstall any existing docker files from the centos repo first.
    - name: Remove docker if installed from CentOS repo
      ansible.builtin.yum:  # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html#parameter-state 
        name: # derste loop görmüştük, fakat günceli paketleri aşağıdaki gibi listelemek
          - docker
          - docker-client
          - docker-client-latest
          - docker-common
          - docker-latest
          - docker-latest-logrotate
          - docker-logrotate
          - docker-engine
        state: removed

  # yum-utils is a collection of tools and programs for managing yum repositories, installing debug packages, source packages, extended information from repositories and administration.
    - name: Install yum utils
      ansible.builtin.yum:
        name: "yum-utils"
        state: latest

  # set up the repository (`yum_repository` modülü de kullanılabilir.) https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_repository_module.html
    - name: Add Docker repo
      ansible.builtin.get_url:  # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/get_url_module.html
        url: https://download.docker.com/linux/centos/docker-ce.repo  # manuel kurulumda bu reponun adresi yazıyor
        dest: /etc/yum.repos.d/docker-ce.repo # manuel kurulumda komutun bu destination'ı kullandığını öğrendik

    - name: Install Docker
      ansible.builtin.package:
        name: docker-ce     # sadece docker-ce kurulumu bize yaterli, cli-containerd ... kurmuyorum
        state: latest

    - name: Add user ec2-user to docker group
      ansible.builtin.user: # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
        name: ec2-user
        groups: docker
        append: yes

    - name: Start Docker service
      ansible.builtin.service:  # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html
        name: docker
        state: started
        enabled: yes

    - name: copy files to the postgresql node
      ansible.builtin.copy:
        src: /home/ec2-user/ansible-project/postgres/
        dest: /home/ec2-user/postgresql # container_path  (Tek playbook vars:)

# Remove the container if it exists
    - name: remove cla_postgre container
      community.docker.docker_container:
        name: cla_postgre # container_name  (Tek playbook vars:)
        state: absent
        force_kill: true

# Remove the image if it exists
    - name: remove clarusway/postgre image
      community.docker.docker_image:
        name: clarusway/postgre # image_name  (Tek playbook vars:)
        state: absent

    - name: build container image
      community.docker.docker_image:
        name: clarusway/postgre   # image_name  (Tek playbook vars:)
        build:
          path: /home/ec2-user/postgresql # image'ı build edeceğin yer # container_path  (Tek playbook vars:)
        source: build   # Use build to build the image from a Dockerfile. build.path must be specified when this value is used.
        state: present
      register: image_info  # yukarıdaki komutun çıktısı bu değişkene (image_info) atanıyor, bu ismi(image_info) ben belirliyorum

    - name: print the image info  # Bu debug Tek playbook'da yok
      ansible.builtin.debug:    # yukarıda register ile kaydettiğim komut sonucunu bu modül ile ekrana yazdırıyorum.
        var: image_info

    - name: Launch postgresql docker container
      community.docker.docker_container:
        name: cla_postgre # container adı # container_name  (Tek playbook vars:)
        image: clarusway/postgre  # image_name  (Tek playbook vars:)
        state: started  # started - Asserts that the container is first present, and then if the container is not running moves it to a running state. Use restart to force a matching container to be stopped and restarted.
        ports: #  published_ports ==>>  aliases: ports
        - "5432:5432"
        env:  # postgres'in olması gereken environment variable'ı
          POSTGRES_PASSWORD: "{{ password }}" # secret.yml'dan alacak => tasks'lardan önce "vars_files: secret.yml" olarak belirlemişim
        volumes:  # https://hub.docker.com/_/postgres
          - /db-data:/var/lib/postgresql/data # "/db-data" isminde bir klasör oluştur ve sonra "/var/lib/postgresql/data" klasörüne bağla 
      register: container_info
    
    - name: print the container info
      ansible.builtin.debug:
        var: container_info
```

### nodejs

```yaml (docker_nodejs.yml) (pwd : /home/ec2-user/ansible)
- name: Install docker
  gather_facts: No
  any_errors_fatal: true
  hosts: _ansible_nodejs
  become: true
  tasks:

    - name: upgrade all packages
      ansible.builtin.yum: 
        name: '*'
        state: latest

    # we may need to uninstall any existing docker files from the centos repo first. 
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

  # yum-utils is a collection of tools and programs for managing yum repositories, installing debug packages, source packages, extended information from repositories and administration.
    - name: Install yum utils
      ansible.builtin.yum:
        name: "yum-utils"
        state: latest

 # set up the repository (`yum_repository` modülü de kullanılabilir.)
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

    - name: copy files to the nodejs node
      ansible.builtin.copy:
        src: /home/ec2-user/ansible-project/nodejs/
        dest: /home/ec2-user/nodejs # container_path (Tek playbook vars:)

  # Remove the container if it exists
    - name: remove cla_nodejs container
      community.docker.docker_container:
        name: cla_nodejs  # container_name (Tek playbook vars:)
        state: absent
        force_kill: true

  # Remove the image if it exists
    - name: remove clarusway/nodejs image
      community.docker.docker_image:
        name: clarusway/nodejs  # image_name (Tek playbook vars:)
        state: absent

    - name: build container image
      community.docker.docker_image:
        name: clarusway/nodejs   # image_name (Tek playbook vars:)
        build:
          path: /home/ec2-user/nodejs # container_path (Tek playbook vars:)
        source: build
        state: present
      register: image_info

    - name: print the image info  # Bu debug Tek playbook'ta yok
      ansible.builtin.debug:
        var: image_info

    - name: Launch nodejs docker container
      community.docker.docker_container:
        name: cla_nodejs    # container_name (Tek playbook vars:)
        image: clarusway/nodejs # image_name (Tek playbook vars:)
        state: started
        ports: 
        - "5000:5000"
      register: container_info
    
    - name: print the container info
      ansible.builtin.debug:
        var: container_info
```

### react

```yaml (docker_react.yml) (pwd : /home/ec2-user/ansible)
- name: Install docker
  gather_facts: No
  any_errors_fatal: true
  hosts: _ansible_react
  become: true
  tasks:

    - name: upgrade all packages
      yum: 
        name: '*'
        state: latest

    # we may need to uninstall any existing docker files from the centos repo first.
    - name: Remove docker if installed from CentOS repo
      yum:
        name: "{{ item }}"
        state: removed
      with_items:
        - docker
        - docker-client
        - docker-client-latest
        - docker-common
        - docker-latest
        - docker-latest-logrotate
        - docker-logrotate
        - docker-engine

    - name: Install yum utils
      yum:
        name: "{{ item }}"
        state: latest
      with_items:
        - yum-utils

    - name: Add Docker repo
      get_url:
        url: https://download.docker.com/linux/centos/docker-ce.repo
        dest: /etc/yum.repos.d/docker-ce.repo

    - name: Install Docker
      package:
        name: docker-ce
        state: latest

    - name: Add user ec2-user to docker group
      user:
        name: ec2-user
        groups: docker
        append: yes

    - name: Start Docker service
      service:
        name: docker
        state: started
        enabled: yes

    - name: copy files to the react node
      copy:
        src: /home/ec2-user/ansible-project/react/
        dest: /home/ec2-user/react  # container_path (Tek playbook vars:)

# remove image ve remove container işlemleri bu kez "shell" modülü kullanıldı
    - name: remove cla_react container and clacw/react image if exists
      shell: "docker ps -q --filter 'name=cla_react' && docker stop cla_react && docker rm -fv cla_react && docker image rm -f clacw/react || echo 'Not Found'" 

    - name: build container image
      docker_image:
        name: clacw/react # image_name (Tek playbook vars:)
        build:
          path: /home/ec2-user/react  # container_path (Tek playbook vars:)
        source: build
        state: present

    - name: Launch react docker container
      docker_container:
        name: cla_react   # container_name (Tek playbook vars:) 
        image: clacw/react  # image_name (Tek playbook vars:)
        state: started
        ports:
        - "3000:3000"
      register: container_info

    - name: Print the container_info
      debug:
        msg: "{{ container_info }}"
```