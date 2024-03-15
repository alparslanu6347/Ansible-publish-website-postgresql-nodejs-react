### Terraform ile değil de manuel olarak EC2'ları ayağa kaldırsaydım `Python3 ve Ansible` kurulumu yapacağım komutlar ve yapacağım işlemler:
# Biz bu bölümdeki işlemleri terraform ile yaptık, aşağıdan 69. satırdan başlayabilirsin.

```bash
sudo yum update -y
sudo yum install -y python3
sudo yum install -y python3-pip 
pip3 install --user ansible
scp -i <pem-file> <pem-file> ec2-user@<public-ip of ansible_control>:/home/ec2-user # lokaldeki pem file'ı  kopyala-yapıştır -->> /home/ec2-user
pip3 install --user boto3 # install "boto3
ansible --version         # Check Ansible's installation
```

- go to AWS Management Consol and select the IAM roles:
- click the  "create role" then create a role with "AmazonEC2FullAccess"
- go to EC2 instance Dashboard, and select the control-node instance
- select actions -> security -> modify IAM role
- select the role thay you have jsut created for EC2 full access and save it.

-  `ansible.cfg` dosyası oluştur 

```sh (ansible.cfg)  (pwd : /home/ec2-user/ansible)
[defaults]
host_key_checking = False
inventory=inventory_aws_ec2.yml
interpreter_python=auto_silent
private_key_file=/home/ec2-user/arrowlevent.pem   ### write your keyname
remote_user=ec2-user
```

- `inventory_aws_ec2.yml` dosyası oluştur ve içine  -->> Terraform ile ec2 içine çektiğin de olur, aşağıdaki de olur --> yapıştır

```yaml (inventory_aws_ec2.yml)  (pwd : /home/ec2-user/ansible)
plugin: aws_ec2
regions:
  - "us-east-1"
filters:
  tag:stack: ansible_project  # 4 instance'da da var bu tag
keyed_groups:
  - key: tags.Name        # isme göre grupla
  - key: tags.environment # environment ismine göre grupla
compose:
  ansible_host: public_ip_address
```

# EC2'lara tag atama işlemleri

- Tag ansible_control, ansible_postgresql, ansible_nodejs, ansible_react instances as below. """ TÜM NODE'LAR """

```sh
stack=ansible_project
```

- Tag the postgresql, nodejs and react instances as below.  """ TÜM NODE'LAR """

```sh
Name=andible_control
Name=ansible_postgresql
Name=ansible_nodejs
Name=ansible_react
```

- Tag ansible_postgresql, ansible_nodejs, ansible_react instances as below. """ control-node HARİÇ """

```sh
environment=development
```
*** -------------------------------------------------- ******* -------------------------------------------------------------- ***

## Part 1 - Launch the instances

- `207-Ansible-publish-website-postgresql-nodejs-react/terraform-files` dizini içerisindeki terraform dosyaları ile instance'ları ayağa kaldır.
  - `myvars.auto.tfvars` içerisinde değişecek yerleri değiştir, `ansible.cfg` değişecek yerleri değiştir, `main.tf` dosyası 121. satırda belirtilen path ile lokalde  `keyname`inizin bulunduğu adres/path aynı olsun.

- 4 adet  `Red Hat Enterprise Linux 9 (HVM)` makina ayağa kalkacak. 

## Part 2 - Prepare the scene

- ansible_control node'a remote ssh ile bağlan

```bash (pwd : /home/ec2-user/)
export PS1="\[\e[1;34m\]\u\[\e[33m\]@\h# \W:\[\e[34m\]\\$\[\e[m\] "   # .bashrc  içine yapıştır.
ansible --version   # Check Ansible's installation
```  

- `207-Ansible-publish-website-postgresql-nodejs-react` klasörü içerisindeki `student_files` klasörünü kopyala `/home/ec2-user/` dizinine yapıştır

- Çalışmak için yeni bir klasör oluştur ve içine girip orada çalış. 
# BU KLASÖR İÇERİSİNE playbook'ları ALACAĞIM

```bash (pwd : /home/ec2-user/)
mkdir ansible
cd ansible
pwd   # /home/ec2-user/ansible
```

-  `ansible.cfg` dosyası oluştur ve içine -->> Terraform ile home/ec2-user içine çektiğin `.ansible.cfg` içeriği de olur, aşağıdaki de olur kopyala --> yapıştır

```sh (ansible.cfg)  (pwd : /home/ec2-user/ansible)
[defaults]
host_key_checking = False
inventory=inventory_aws_ec2.yml
interpreter_python=auto_silent
private_key_file=/home/ec2-user/arrowlevent.pem   ### write your keyname
remote_user=ec2-user
```

```bash (pwd : /home/ec2-user/ansible)
$ ansible --version   # önce bulunduğumuz dizindeki "ansible.cfg" dosyasına bakar, burada yoksa home directory'deki .ansible.cfg , o da yoksa etc klasörüne, FAKAT environment variable olarak tanımlarsak hepsinin önüne geçer
                      # config file = /home/ec2-user/ansible/ansible.cfg
```

- `inventory_aws_ec2.yml` dosyası oluştur ve içine  -->> Terraform ile home/ec2-user içine çektiğin de olur, aşağıdaki de olur --> yapıştır

```yaml (inventory_aws_ec2.yml)  (pwd : /home/ec2-user/ansible)
plugin: aws_ec2
regions:
  - "us-east-1"
filters:          # EC2'ları filtreler sadece filtresi olanları ekranda gösterir.
  tag:stack: ansible_project  # 4 instance'da da var bu tag
keyed_groups:     # filtreye göre ekrana getirdiği EC2'ları gruplandırarak listeler
  - key: tags.Name  # isme göre grupla
  - key: tags.environment # environment ismine göre grupla
compose:
  ansible_host: public_ip_address # ssh bağlanmak için public_ip kullan dedik, private IP de kullanabilirdik
```

```bash (pwd : /home/ec2-user/ansible)
ansible-inventory -i inventory_aws_ec2.yml --graph
```

```
@all:
  |--@_ansible_control:
  |  |--ec2-3-80-96-146.compute-1.amazonaws.com
  |--@_ansible_nodejs:
  |  |--ec2-3-239-243-194.compute-1.amazonaws.com
  |--@_ansible_postgresql:
  |  |--ec2-3-236-160-236.compute-1.amazonaws.com
  |--@_ansible_react:
  |  |--ec2-3-236-197-117.compute-1.amazonaws.com
  |--@_development:
  |  |--ec2-3-236-160-236.compute-1.amazonaws.com
  |  |--ec2-3-236-197-117.compute-1.amazonaws.com
  |  |--ec2-3-239-243-194.compute-1.amazonaws.com
  |--@aws_ec2:
  |  |--ec2-3-236-160-236.compute-1.amazonaws.com
  |  |--ec2-3-236-197-117.compute-1.amazonaws.com
  |  |--ec2-3-239-243-194.compute-1.amazonaws.com
  |  |--ec2-3-80-96-146.compute-1.amazonaws.com
  |--@ungrouped:
```

- To make sure that all our hosts are reachable with dynamic inventory, we will run various ad-hoc commands that use the ping module.

```bash (pwd : /home/ec2-user/ansible)
ansible all -m ping
ansible _development -m ping
ansible _ansible_postgresql -m ping
ansible aws_ec2 -m ping 
```

## Part 3 - Prepare the playbook files

- Create `ansible-Project` directory under home directory and change directory to this directory.

```bash (pwd : /home/ec2-user/ansible)
cd
mkdir ansible-project
cd ansible-project
```

- Create `postgres`, `nodejs`, `react` directories.

```bash (pwd : /home/ec2-user/ansible-project)
mkdir postgres nodejs react
```

### postgres

- Copy the content of `~/student_files/todo-app-pern/database` directory to `~/ansible-project/postgres` folder. (init.sql dosyası)

```bash (pwd : /home/ec2-user/ansible-project)
cd postgres
```

- Create a Dockerfile

# https://hub.docker.com/
# https://hub.docker.com/_/postgres : Initialization scripts : If you would like to do additional initialization in an image derived from this one, add one or more *.sql, *.sql.gz, or *.sh scripts under `/docker-entrypoint-initdb.d` (creating the directory if necessary).

```Dockerfile (pwd : /home/ec2-user/ansible-project/postgres)
FROM postgres

COPY ./init.sql /docker-entrypoint-initdb.d/

EXPOSE 5432
```

```bash (pwd : /home/ec2-user/ansible-project/postgres)
cd
cd ~/ansible
```

- Create a yaml file as postgres playbook and name it `docker_postgre.yml`.

# https://docs.docker.com/engine/install/
# https://docs.docker.com/engine/install/centos/  : docker manuel kurmak için neye ihtiyacım var.

```yaml (docker_postgre.yml) (pwd : /home/ec2-user/ansible)
- name: Install docker
  gather_facts: No  # bu instance bilgilerini toplamaz. In Ansible, I can use gather_facts: yes to collect info about my hosts. As gather_facts collects a lot of information, it takes quite a while. ansible 3.ders 
  any_errors_fatal: true  # BİR YERDE HATA ALDIĞINDA playbook'u KOMPLE DURDUR. ansible 4.ders
  hosts: _ansible_postgresql
  become: true
  vars_files:
    - secret.yml  # Tek playbook'da  -->> env: POSTGRES_PASSWORD: "Pp123456789"
  tasks:  
  
  # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html#parameter-state , https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html#parameter-name
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
        src: /home/ec2-user/ansible-project/postgres/   # bu dizinde "Dockerfile"  ile "init.sql" var
        dest: /home/ec2-user/postgresql # "Dockerfile" çalıştıracağım dizin.  # container_path  (Tek playbook vars:)

# Remove the container if it exists       # BİZ ansible İLE image GÜNCELLERİZ Kİ EN SON HALİ ÇALIŞSIN İSTERİZ, ESKİ VERSİYON İLE İLGİLİ container VE image'LERİ SİLERİZ.
    - name: remove cla_postgre container
      community.docker.docker_container:
        name: cla_postgre # container_name  (Tek playbook vars:)
        state: absent
        force_kill: true

# Remove the image if it exists       # BİZ ansible İLE image GÜNCELLERİZ Kİ EN SON HALİ ÇALIŞSIN İSTERİZ, ESKİ VERSİYON İLE İLGİLİ container VE image'LERİ SİLERİZ.
    - name: remove clarusway/postgre image
      community.docker.docker_image:
        name: clarusway/postgre # image_name  (Tek playbook vars:)
        state: absent

    - name: build container image
      community.docker.docker_image:
        name: clarusway/postgre   # image_name  (Tek playbook vars:)
        build:
          path: /home/ec2-user/postgresql # image'ı build edeceğin yer # container_path  (Tek playbook vars:)
        source: build       # Use build to build the image from a Dockerfile. build.path must be specified when this value is used.
        state: present
      register: image_info  # yukarıdaki komutun çıktısı bu değişkene (image_info) atanıyor, bu ismi(image_info) ben belirliyorum

    - name: print the image info  # Bu debug Tek playbook'da yok
      ansible.builtin.debug:      # yukarıda register ile kaydettiğim komut sonucunu bu modül ile ekrana yazdırıyorum.
        var: image_info

    - name: Launch postgresql docker container
      community.docker.docker_container:
        name: cla_postgre         # container adı # container_name  (Tek playbook vars:)
        image: clarusway/postgre  # image_name  (Tek playbook vars:)
        state: started            # started - Asserts that the container is first present, and then if the container is not running moves it to a running state. Use restart to force a matching container to be stopped and restarted.
        ports: #  published_ports ==>>  aliases: ports
        - "5432:5432"
        env:  # postgres'in olması gereken environment variable'ı
          POSTGRES_PASSWORD: "{{ password }}" # secret.yml'dan alacak => tasks'lardan önce en başlarda  "vars_files: secret.yml" olarak belirlemiştim.
        volumes:  # https://hub.docker.com/_/postgres
          - /db-data:/var/lib/postgresql/data # "/db-data" isminde bir klasör oluştur ve sonra "/var/lib/postgresql/data" klasörüne bağla 
      register: container_info
    
    - name: print the container info
      ansible.builtin.debug:
        var: container_info
```

# playbook çalıştırmadan önce ``secret.yml`` oluşturalım.

```bash (pwd : /home/ec2-user/ansible)
ansible-vault create secret.yml   # 2 defa şifre soracak    12345
```

```yml (secret.yml)
password: Pp123456789
```

```bash (pwd : /home/ec2-user/ansible)
ansible-vault view secret.yaml    # bu komutla içerisini görebiliyorum.
        Vault password: xxxx      # şifre ister     12345
        
        password: Pp123456789

ansible-playbook --ask-vault-pass docker_postgre.yml  # şifre soracak, yukarıda 2 defa sorduğunda girdiğin şifre
        Vault password: xxxx  # şifre ister     12345
# [WARNING]: The input password appears not to have been hashed. The 'password' argument must be encrypted for this module to work properly.
# Bu uyarıya göre password şifrelenmedi.

ansible _ansible_postgresql -b -m shell -a "docker ps"  # -b:sudo , -m:module , -a:argument

ansible-inventory --graph # eğer yukarıdaki komuta yazacağın host ismini hatırlamadıysan bu komutu gir --->> _ansible_postgresql
```

### nodejs

- Copy the content of `~/student_files/todo-app-pern/server` directory to `~/ansible-project/nodejs` folder.

- Change directory to `~/ansible-project/nodejs` directory.

```bash
cd ~/ansible-project/nodejs
```

- Create a Dockerfile.

```Dockerfile   (pwd : /home/ec2-user/ansible-project/nodejs)
FROM node:14

WORKDIR /usr/src/app    # Create app directory

COPY package*.json ./

RUN npm install   # If you are building your code for production  # RUN npm ci --only=production

COPY . .  # copy all files into the image

EXPOSE 5000

CMD ["node","app.js"]
```

- Change the `~/ansible-project/nodejs/.env` file as below.
  # `DB_NAME=clarustodo` nereden geliyor --->> `/home/ec2-user/postgresql/init.sql` # student_files/todo-app-pern/database/init.sql ---manuel copy_paste--->> ansible-project/postgres/init.sql  ---ansible ile kopyalamıştık--->>> /home/ec2-user/postgresql/init.sql

  # Diğer bilgiler ``/home/ec2-user/nodejs/db.js``
  
```sh (.env)  (pwd : ~/ansible-project/nodejs/)
SERVER_PORT=5000
DB_USER=postgres
DB_PASSWORD=Pp123456789
DB_NAME=clarustodo    ### `DB_NAME=clarustodo` nereden geliyor --->> `/home/ec2-user/postgresql/init.sql`
DB_HOST=172.31.26.183 ### (private ip of postgresql instance) AWS Console'dan kopyala-yapıştır.
DB_PORT=5432
```

- change directory `~/ansible` directory.

```bash (pwd : ~/ansible-project/nodejs)
cd ~/ansible
```

- Create a yaml file as nodejs playbook and name it `docker_nodejs.yml`.

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
        name: cla_nodejs          # container_name (Tek playbook vars:)
        image: clarusway/nodejs   # image_name (Tek playbook vars:)
        state: started
        ports: 
        - "5000:5000"
      register: container_info
    
    - name: print the container info
      ansible.builtin.debug:
        var: container_info
```

- Execute it.

```bash (pwd : /home/ec2-user/ansible)
ansible-playbook docker_nodejs.yml

ansible _ansible_nodejs -b -m shell -a "docker ps"

ansible-inventory --graph # eğer yukarıdaki komuta yazacağın host ismini hatırlamadıysan bu komutu gir --->> _ansible_nodejs

http://nodejs-EC2-Public-IP:5000/todos    # uygulama göründü.
```

### react

- Copy the content of `~/student_files/todo-app-pern/client` directory to `~/ansible-project/react` folder.

- Change directory to `~/ansible-Project/react` directory.

```bash
cd ~/ansible-project/react
```

- Create a Dockerfile.

```Dockerfile   (pwd : /home/ec2-user/ansible-project/react)
FROM node:14

WORKDIR /app    # Create app directory

COPY package*.json ./

RUN yarn install

COPY . .    # copy all files into the image

EXPOSE 3000

CMD ["yarn", "run", "start"]
```

- Change the `~/ansible-project/react/.env` file as below.

```sh (.env)  (pwd : ~/ansible-project/react/)
REACT_APP_BASE_URL=http://3.83.160.118:5000/     ### public ip of nodejs : 3.83.160.118
```

- change directory `~/ansible` directory.

```bash
cd ~/ansible
```

- Create a yaml file as react playbook and name it `docker_react.yml`.

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

- Execute it.

```bash
ansible-playbook docker_react.yml

ansible _ansible_react -b -m shell -a "docker ps"

ansible-inventory --graph # eğer yukarıdaki komuta yazacağın host ismini hatırlamadıysan bu komutu gir --->> _ansible_react

http://react-EC2-Public-IP:3000    # uygulama göründü.
```

## Part 4 - Prepare ``one playbook`` file for all instances.

- Create a `docker_project.yml` file under `the ~/ansible` folder.

```yaml (docker_project.yml)  (pwd : /home/ec2-user/ansible)
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


- name: Postgre Database configuration
  hosts: _ansible_postgresql
  become: true
  gather_facts: No
  any_errors_fatal: true
  vars_files:
    - secret.yml
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
        env:
          POSTGRES_PASSWORD: "{{ password }}"
          # POSTGRES_PASSWORD: "Pp123456789"  # vars_file olarak secret.yml kullanmasaydık direkt olarak buraya yazardık.
        volumes:
          - /db-data:/var/lib/postgresql/data
      register: container_info
    
    - name: print the container info
      debug:
        var: container_info


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

# playbook çalıştırmadan önce `~/ansible-project/nodejs/.env` ve `~/ansible-project/react/.env` dosyalarının içeriğini değiştirelim . (Başta yapmadık farzedelim)

```sh (.env)  (pwd : ~/ansible-project/nodejs/)
SERVER_PORT=5000
DB_USER=postgres
DB_PASSWORD=Pp123456789
DB_NAME=clarustodo    ### `DB_NAME=clarustodo` nereden geliyor --->> `/home/ec2-user/postgresql/init.sql`
DB_HOST=172.31.26.183 ### (private ip of postgresql instance) AWS Console'dan kopyala-yapıştır.
DB_PORT=5432
```

```sh (.env)  (pwd : ~/ansible-project/react/)
REACT_APP_BASE_URL=http://3.83.160.118:5000/     ### public ip of nodejs : 3.83.160.118
```

# playbook çalıştırmadan önce ``secret.yml`` oluşturalım. (Başta yapmadık farzedelim)

```bash (pwd : /home/ec2-user/ansible)
ansible-vault create secret.yml   # 2 defa şifre soracak    12345
```

```yml (secret.yml)
password: Pp123456789
```

```bash (pwd : /home/ec2-user/ansible)
ansible-vault view secret.yaml    # bu komutla içerisini görebiliyorum.
        Vault password: xxxx      # şifre ister     12345
        
        password: Pp123456789
```

- Execute it.

```bash (pwd : /home/ec2-user/ansible)
ansible-playbook --ask-vault-pass docker_postgre.yml
# ansible-playbook docker_project.yml   # vars_file - secret.yml kullanmasaydım direkt env yazsaydım bu komut

http://nodejs-EC2-Public-IP:5000/todos    # uygulama göründü.
http://react-EC2-Public-IP:3000           # uygulama göründü.
```

------------------------------------------------------------------------------------------------------------------------------------------------------

## Part 5 - Prepare ``playbook with roles`` solution.

- Cretae a role folder in /home/ec2-user/ansible. Then create role folders.

```bash (pwd : /home/ec2-user/ansible)
mkdir roles && cd roles
ansible-galaxy init docker
ansible-galaxy init postgre
ansible-galaxy init nodejs
ansible-galaxy init react
```

- roles klasörünün adresini/path `/home/ec2-user/ansible` içerisindeki `ansible.cfg`e ekleyelim
    "   roles_path=/home/ec2-user/ansible/roles   "

- Hazırlamış olduğum `/home/ec2-user/ansible` içerisindeki tek parça playbook olan `docker-project.yaml`ı  roles klasörü içerisinde ilgili yerlere yerleştiriyorum.

- Go to the `/home/ec2-user/ansible/roles/docker/tasks/main.yml` and copy the following. `docker` role için

```yaml (docker için role)  (pwd : /home/ec2-user/ansible/roles/docker/tasks)
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
```


- Go to the `/home/ec2-user/ansible/roles/postgre/tasks/main.yml` and copy the followings. `postgre` role için

```yaml (pwd : /home/ec2-user/ansible/roles/postgre/tasks/)
    - name: copy files to the postgresql node 
      ansible.builtin.copy:
        src: postgres/ # write only file name : ``/home/ec2-user/ansible/roles/postgre/files`` içindeki `postgres` klasörü
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


- Copy `/home/ec2-user/ansible-project/postgres` folder to ``/home/ec2-user/ansible/roles/postgre/files``.

- Copy these variables to ``/home/ec2-user/ansible/roles/postgre/vars/main.yml``.

```yml (pwd : /home/ec2-user/ansible/roles/postgre/vars/)
container_path: /home/ec2-user/postgresql
container_name: cla_postgre
image_name: clarusway/postgre
```

- Go to the `/home/ec2-user/ansible/roles/nodejs/tasks/main.yml` and copy the followings. `nodejs` role için

```yaml (pwd : /home/ec2-user/ansible/roles/nodejs/tasks/)
    - name: copy files to the nodejs node
      copy:
        src: nodejs/  # ``/home/ec2-user/ansible/roles/nodejs/files`` içerisindeki `nodejs` klasörü
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
        - "5000:5000"
      register: container_info
    
    - name: print the container info
      debug:
        var: container_info
```

- Copy `/home/ec2-user/ansible-project/nodejs` folder to ``/home/ec2-user/ansible/roles/nodejs/files``.


- Copy these variables to ``/home/ec2-user/ansible/roles/nodejs/vars/main.yml``.

```yml (pwd : /home/ec2-user/ansible/roles/nodejs/vars/)
container_path: /home/ec2-user/nodejs
container_name: cla_nodejs
image_name: clarusway/nodejs
```

- Go to the `/home/ec2-user/ansible/roles/react/tasks/main.yml` and copy the followings.  `react` role için

```yaml (pwd : /home/ec2-user/ansible/roles/react/tasks/)
    - name: copy files to the react node
      copy:
        src: react/ # ``/home/ec2-user/ansible/roles/react/files`` içerisindeki `react` klasörü
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
        - "3000:3000"
      register: container_info
    
    - name: print the container info
      debug:
        var: container_info
```

- Copy `/home/ec2-user/ansible-project/react folder` to ``/home/ec2-user/ansible/roles/react/files``.

- Copy these variables to ``/home/ec2-user/ansible/roles/react/vars/main.yml``.
`
```yaml (pwd : /home/ec2-user/ansible/roles/react/vars/)
container_path: /home/ec2-user/react
container_name: cla_react
image_name: clarusway/react
```

- Update the ansible.cfg file in /home/ec2-user/ansible/. Add roles_path as below.

```sh
# Bu işlemi zaten tek playbook hazırlarken yaptık.
roles_path= /home/ec2-user/ansible/roles
```

- Go to the /home/ec2-user/ansible/ and create a playbook.

```bash
cd /home/ec2-user/ansible/
nano play-role.yml
```

  #  4 role'ü (docker, postgre, nodejs, react) çalıştıracak playbook 

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

---------------------------------------------------------------------------------------------------------------------------------------------------------

## Part 6 - Using role from ``ansible-galaxy``.

- Search for a docker role.

```bash
ansible-galaxy search docker --platform EL | grep geerl
```

- Change directory to ansible folder.

```bash
cd /home/ec2-user/ansible
```

- install the role.

```bash (pwd : /home/ec2-user/ansible)
ansible-galaxy role install geerlingguy.docker    # roles klasörü altında geerlingguy.docker role klasörü oluştu
```

- Create a new play-newrole.yml
# docker için galaxy'den role (geerlingguy.docker) indiriyorum, diğer 3 role benim hazırladığım role.

```yaml (play-newrole.yml)
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