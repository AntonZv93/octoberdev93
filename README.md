Устанавливаем Ansible, если версия ниже 2.2 обновляем:

добавляем в /etc/apt/sources.list репозиторий 
deb http://ppa.launchpad.net/ansible/ansible/ubuntu xenial main
Добавляем ключ:
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
и устанавливаем: 
sudo apt-get update
sudo apt-get install python-yaml python-jinja2 python-paramiko python-crypto 
sudo apt-get install ansible
Вносим изменения в конфигурационный файл /etc/ansible/ansible.cfg:
Раскомментируем  строку host_key_checking = False . Это необходимо для того, 
что бы при удалении и повторном создании виртуальной машины игнорировать ошибку изменения ключа.
также активируем строку nocows = 1, этот параметр необязателен, если вам нравится изображение веселой коровы, то игнорируем его.

Переходим к созданию проекта. 
Для чего переходим по адресу - https://console.cloud.google.com/projectselector/compute/instances.

Я назвал проект octobercms, т.к. в будущем именно octobercms будет развернут в виртуальной машине.
После создания, нам понадобится идентификатор проекта

Переходим по ссылке htps://console.cloud.google.com/iam-admin/serviceaccounts/  
выбираем аккаунт, либо создаем новый для нашего проекта и сохраняем ключ (тип JSON) 

Последнее что нам необходимо - это сгенерировать SSH ключ:
ssh-keygen -t rsa -f ~/.ssh/[KEY_FILE_NAME] -C [USERNAME] , где для [KEY_FILE_NAME] и [USERNAME] указываем свои значения.
После чего загружаем наш новый публичный ключ по адресу https://console.cloud.google.com/compute/metadata/sshKeys

На этом этапе завершаем работу с Google Cloud и переходим к Ansible

Устанавливаем необходимые зависимости:
sudo apt-get install -y build-essential git python-dev python-pip
Устанавливаем libcloud:
sudo pip install apache-libcloud==0.20.1
Описание модуля можно посмотреть здесь - http://docs.ansible.com/ansible/list_of_cloud_modules.html#google
Переходим к настройке ansible playbook: 
Я создал директорию /etc/ansible/gce куда поместил JSON ключ проекта
Все последующие файлы будут находится в этой директории. Создаем файл /etc/ansible/gce/var
 в котором прописываем необходимые переменные  :
---
service_account_email: *-compute@developer.gserviceaccount.com
credentials_file: /etc/ansible/gce/WordPress-1396309b57b.json
project_id: wordpress-16508
machine_type: f1-micro
image: debian-8

Cоздаем файл playbook /etc/ansible/gce/create.yml

---
- name: Create vm
 hosts: localhost
 connection: local
 gather_facts: no

 vars_files:
   - var

 tasks:
   - name: Launch instances
     gce:
#заменяем точки на тире (точки не используются в имени машины
         instance_names: '{{ domain | regex_replace("(\.)","-") }}'
         machine_type: "{{ machine_type }}"
         image: "{{ image }}"
         service_account_email: "{{ service_account_email }}"
         credentials_file: "{{ credentials_file }}"
         project_id: "{{ project_id }}"
         tags: webserver
     register: gce

   - name: Wait for SSH to come up
     wait_for: host={{ item.public_ip }} port=22 delay=1 timeout=30 state=started
     with_items: "{{ gce.instance_data }}"

   - name: Add host to groupname
     add_host: hostname={{ item.public_ip }} groupname=new_instances
     with_items: "{{ gce.instance_data }}"

   - name: Allow HTTP traffic
     gce_net:
       fwname: pass-http
       name: default
       allowed: tcp:80
       project_id: "{{ project_id }}"
       credentials_file: "{{ credentials_file }}"
       service_account_email: "{{ service_account_email }}"

   - name: Allow HTTPS traffic
     gce_net:
       fwname: pass-https
       name: default
       allowed: tcp:443
       project_id: "{{ project_id }}"
       credentials_file: "{{ credentials_file }}"
       service_account_email: "{{ service_account_email }}"

   - name: Create zone
     gcdns_zone:
       project_id: "{{ project_id }}"
       credentials_file: "{{ credentials_file }}"
       service_account_email: "{{ service_account_email }}"
       zone: "{{ domain }}"

   - name: Create record
     gcdns_record:
       project_id: "{{ project_id }}"
       credentials_file: "{{ credentials_file }}"
       service_account_email: "{{ service_account_email }}"
       record: "{{ domain }}"
       zone: "{{ domain }}"
       type: 'A'
       value: "{{ item.public_ip }}"
     with_items: "{{ gce.instance_data }}"
 

Для запуска это сценария выполняем команду:
ansible-playbook create.yml --extra-vars "domain=your_dns" --user=your_name --private-key=~/.ssh/your_name

С помощью этого playbook создаем новую виртуальную машину, правила разрешающие доступ к портам 80,443 и Cloud DNS с A-записью, в переменной domain передаем нашему сценарию домен который будет создан.

Ниже привожу сценарий для удаления созданной нами машины
/etc/ansible/gce/destroy.yml
---
- name: Destroy
 hosts: localhost
 connection: local
 gather_facts: no

 vars_files:
   - var

 tasks:
   - name: gce
     gce:
         instance_names: '{{ domain | regex_replace("(\.)","-") }}'
         machine_type: "{{ machine_type }}"
         image: "{{ image }}"
         service_account_email: "{{ service_account_email }}"
         credentials_file: "{{ credentials_file }}"
         project_id: "{{ project_id }}"
         tags: webserver
     register: gce

   - name: remove record
     gcdns_record:
       project_id: "{{ project_id }}"
       credentials_file: "{{ credentials_file }}"
       service_account_email: "{{ service_account_email }}"
       record: "{{ domain }}"
       zone: "{{ domain }}"
       state: 'absent'
       type: 'A'
       value: "{{ item.public_ip }}"
     with_items: "{{ gce.instance_data }}"

   - name: remove zone
     gcdns_zone:
       project_id: "{{ project_id }}"
       credentials_file: "{{ credentials_file }}"
       service_account_email: "{{ service_account_email }}"
       zone: "{{ domain }}"
       state: 'absent'

   - name: Destroy
     gce:
         instance_names: '{{ domain | regex_replace("(\.)","-") }}'
         machine_type: "{{ machine_type }}"
         image: "{{ image }}"
         service_account_email: "{{ service_account_email }}"
         credentials_file: "{{ credentials_file }}"
         project_id: "{{ project_id }}"
         tags: webserver
         state: absent
		 
		 
		 
		 В файле thisistrueoctoberplaybookforcentos.yml содержится набор команд для автоматического обновления системы и установки необходимых комманд 
		 для того чтобы можно было создать octobercms на всех instances в вашей облачной платформе
		 запуск ad-hoc комманды :ansible-playbook thisistrueoctoberplaybookforcentos.yml
		 переход по вашему ip instances с добавлением /install.php и указываем порт через двоеточие и следуем инструкциям по установке
		 