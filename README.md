# Updating Nextcloud using Ansible

*Please note that the following steps assume you have already installed and configured Nextcloud using Docker and docker-compose.*
*. Here's a small example of how you can set it up https://github.com/PashaShredder/nginx-proxy-nextcloud-docker-postgresql-redis*
*. We will use this sample setup in our example*

# Install Ansible
 https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
*After pre-installation, proceed to setup*
*Go to folder*
```bash
cd /etc/ansible/
```
*Edit file* **hosts** 
```bash
sudo nano hosts # use any editor convenient for you (I will use nano in the future)
```
```bash
[nextcloud_server] # your group name
<your_server_name> # Client, Ubuntu, etc.
```
*Go to the folder and create a file there for our configuration. You can choose any name, but it will also need to be specified in the file * **hosts** and **update.yml**
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
*Check if the configuration is correct*
```bash
ansible all -m ping
```
```bash
<your_server_name> | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong" # This answer tells us that everything is OK, we can continue. Otherwise check your ansible config
}
```
*Let's go back and create our playbook*
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
*Next, let's run our playbook, taking into account that docker compose is running and nextloud is configured and available for use*

```bash
sudo ansible-playbook update.yml -i hosts
```
*After the successful completion of Ansible, go to the browser (wait for some time until a full reboot) to your nextcloud and complete the installation*

