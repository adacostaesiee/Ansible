# ESIEE-IT (M2i RS) - Ansible - Advanced Linux Administration final project
This project aims to develop a secure and redundant infrastructure using automating the deployment and configuration of services (rsyslog, apache, heartbeat, mariadb replication), thanks to Ansible, while using Git to manage versions.

Theses playbooks will automatically install and configure GLPI 9.4.5 an duplicate the database to the 2nd dbserver. Thanks to [Supertarto](https://github.com/supertarto/ansible-glpi)

## What does the infrastructure requires?

This infrastructure requires at least 5 machines: 
- webserver1: x.x.x.x/x
- webserver2: x.x.x.x/x
- dbserver1: x.x.x.x/x
- dbserver2: x.x.x.x/x
- syslog: x.x.x.x/x

+ Your Ansible Orchestrator


+ An aditionnal IP Address for the Virtual IP of the Web cluster

+ An ansible-vault password, 'esiee' for our example.

## How to set-up the project?

* Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)


```
https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html
```


* Open **inventory.yaml** with a text editor, and modify as you want host's IP Addresses.
Then, modify the value of 'ansible_user' field by your local sudoer account.
Add your Ansible Orchestrator's public key to ./ and name it 'id_rsa.pub'

* Open a terminal on your **Ansible Orchestrator**, and type: 

```bash
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.mysql
cd ansible
```

You'll have to give your local sudoer account password, just for this time.

Don't forget to modify the Jinga2 templates. 

**./templates/authkeys.j2**
- Modify as you want the authkey for the Heartbeat Cluster

**./templates/ha.cf.j2**
- Modify the 8th line by your default GW
```
ping x.x.x.x
```

**./templates/haresources.cf.j2**
- Modify the unique line by the Virtual IP you want, the subnet mask and the name of your virtual network interface of your webservers.
```
webserver1 IPaddr::x.x.x.x/xx/ethx:0
```

**./files/webapp.j2**
- Modify 'ServerName' as you want, but you'll also be able to access your web cluster using it's IP Address
```
ServerName your.server.name
```

**You are now ready to play.**


/!\ If it crashs during packet installation, you may try on every VM:

```bash
sudo apt-get update
sudo apt-get upgrade
```
_______________________
/!\ If a playbook's crashing during execution, relaunch it.
_______________________
/!\ If an error occurs at this step, while installing and unpackaging GLPI, please go on your webservers vm and do this:
```bash
sudo rm -r /var/www/glpi
```
_______________________
* Let's start with: 
```bash
ansible-playbook playbook.yml -i inventory.yaml -K --ask-vault-pass -e "ansible_user=YourLocalSudoerAccount" --ask-pass
ansible-playbook playbook_resetting_mariadb_password_dbservers.yml -i inventory.yaml --ask-vault-pass
ansible-playbook playbook_glpi.yml -i inventory.yaml --ask-vault-pass
ansible-playbook playbook_webservers.yml -i inventory.yaml --ask-vault-pass
ansible-playbook playbook_dbservers_replication.yml -i inventory.yaml --ask-vault-pass
ansible-playbook playbook_syslog.yml -i inventory.yaml --ask-vault-pass
ansible-playbook playbook_ntp.yml -i inventory.yaml --ask-vault-pass
ansible-playbook playbook_dbservers.yml -i inventory.yaml --ask-vault-pass
```

Try replaying every playbooks if it doesn't work.
It is possible that Internet connectivity problems occur during the download of the packages and thus that this produces errors.