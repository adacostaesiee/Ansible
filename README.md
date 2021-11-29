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

+ Your Ansible Orchestrator.

+ An aditionnal IP Address for the Virtual IP of the Web cluster.

+ An ansible-vault password, 'esiee' for our example. 

## How to set-up the project?

Open **inventory.yaml** with a text editor, and modify as you want host's IP Addresses.
Then, modify the value of 'ansible_user' field by your local sudoer account.
Add your Ansible Orchestrator's public key to ./ and name it 'id_rsa.pub'

Open a terminal on your **Ansible Orchestrator**, and type: 

```bash
cd ansible
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.mysql
ansible-playbook playbook.yml -i inventory.yaml -K --ask-vault-pass -e "ansible_user=YourLocalSudoerAccount" --ask-pass
```

You'll have to give your local sudoer account password, just for this time. 

/!\ If this doesn't work, try using 'ssh-copy-id' from your Ansible Orchestrator to your remotes hosts before.

```bash
ssh-copy-id YourUsername@YourRemoteHost
```

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

You are now ready to play.


If it crashs during packet installation, you may try on every VM:

```bash
sudo apt-get update
sudo apt-get upgrade
```

```bash
cd ./ansible
ansible-playbook playbook_webservers.yml -i inventory.yaml
ansible-playbook playbook_glpi.yaml -i inventory.yaml --ask-vault-pass
```

**If an error occurs at this step, while installing and unpackaging GLPI, please go on your webservers vm and do this: 

```bash
sudo rm -r /var/www/glpi
```

**If an error occurs at this step, while installing and unpackaging GLPI, please go on your webservers vm and do this: 

```bash
sudo rm -r /var/www/glpi
```

It will crash because of mysql default root password

Then, run: 

```bash
ansible-playbook playbook_dbservers.yml -i inventory.yaml --ask-vault-pass
```

It will crash another time.

Run: 

```bash
ansible-playbook playbook_resetting_mariadb_password_dbservers.yml -i inventory.yaml --ask-vault-pass
```

Then: 

```bash
ansible-playbook playbook_dbservers.yml -i inventory.yaml --ask-vault-pass
ansible-playbook playbook_dbservers_replication.yml -i inventory.yaml --ask-vault-pass
ansible-playbook playbook_ntp.yaml -i inventory.yaml --ask-vault-pass
ansible-playbook playbook_syslog.yml -i inventory.yaml --ask-vault-pass
```

