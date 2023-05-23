# Обновление Nextcloud при помощи Ansible
*Все последующие действия выполняются с учётом того, что Вы установили и настроили Nextcloud с использованием Docker и docker compose*
*. Небольшой пример как это можно сделать https://github.com/PashaShredder/nginx-proxy-nextcloud-docker-postgresql-redis*
*. Мы будем использовать данный образец в нашем примере*

# Установка Ansible https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
*После предварительной установки приступаем к настройке*
*Переходим в папке*
```bash
cd /etc/ansible/
```
*Редактируем файл* **hosts** 
```bash
sudo nano hosts # используйте любой удобный для Вас редактор(Я буду в дальнейшем использовать nano)
```
```bash
[nextcloud_server] # your group name
<your_server_name> # Client, Ubuntu, etc.
```
*Переходим в папку и создаём там файл для нашей конфигурации. Имя можно выбрать любое но его же нужно будет указать в файле* **hosts** и **update.yml**
```bash
cd /etc/ansible/group_vars
```
```bash
sudo nano nextcloud_server
```
```bash
ansible_host: <your_server_ip>
ansible_user: <server_user>
ansible_ssh_private_key_file: /path/to/.ssh/<your_ssh_key>
```
*Проверим корректность конфигурации*
```bash
ansible all -m ping
```
```bash
<your_server_name> | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong" # данный ответ говорит нам что всё ОК, можем продолжать. В противном случае проверьте конфигурацию ansible
}
```
*Вернёмся обратно и создадим наш playbook*
```bash
cd /etc/ansible
```
```bash
sudo nano update.yml
```
```bash
- name: Update Nextcloud
  hosts: nextcloud_server
  become: true

  tasks:
    - name: Stoping docker compose
      shell: |
        docker compose down
      args:
        chdir: /app/nextcloud

    - name: Updating Nextcloud version in docker compose
      lineinfile:
        path: /app/nextcloud/docker-compose.yml
        regexp: '^(\s+image:\s*)"nextcloud:.*"'
        line: '\g<1>"nextcloud:25.0-apache"'
        backrefs: yes

    - name: Recreate and restart Nextcloud
      command: docker compose up -d
      args:
        chdir: /app/nextcloud

    - name: Restarting Nginx
      become: true
      service:
        name: nginx
        state: restarted
```
*Далее запустим наш playbook с учётом того что docker compose работает и nextloud сконфигурирован и доступен для использования*

```bash
sudo ansible-playbook update.yml -i hosts
```
*После успешного завершения работы Ansible переходим в браузер(ждём какое-то время до полной перезагрузки) на Ваш nextcloud  и завершаем установку*

