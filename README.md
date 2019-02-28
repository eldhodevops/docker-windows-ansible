# docker-windows-ansible
Install docker on windows-server using Ansible script

Docker installation on windows-server-2019

## Prerequisites
Ansible only:

. Ansible 2.5.0+ installed on Ubuntu for Windows (pip installation recommended).
. Additional python packages installed for WinRM (follow [Ansible Windows Setup Guide](http://docs.ansible.com/ansible/2.5/user_guide/windows_setup.html)).
. Windows Server
. WinRM properly configured on Windows node.


##  Ansible usage

#### Prepare Ansible Host system
- Setup for ansible host  on Ubuntu system ( It can be use your Laptop , do not use Servers)
```
apt-get update
apt-get -y install software-properties-common
apt-add-repository -y ppa:ansible/ansible
apt-get update
apt-get install -y ansible python-pip
pip install pywinrm
```


#### Prepare Windows Node to access Ansible Remotely (On windows server)

 - To use this script to enable https port 5986, run the following in PowerShell:

```

$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)

powershell.exe -ExecutionPolicy ByPass -File $file
winrm enumerate winrm/config/Listener
```
- Install chocolatey package manager to avoid error while running ansible-playbook

```
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

```

### Prepare inventory file
A sample inventory file has been provided in [Ansible playbook directory](ansible/inventory/docker.ini).

your inventory should be defined as:

```
[node-windows]
#xxserverIPxx windows_node_hostname=xxxxx

```
### Prepare authentication to Node

 All the sample files has been provided in [Ansible playbook group_var directory](ansible/group_vars).
you need to edit with example below

- Edit windows username and password inside the file group_vars/node-windows.yml

```
ansible_user: xxusernamexx
# Ensure you don't have ansible_ssh_pass var set in your own group_vars/all.yml, it will overwrite this.
ansible_password: xxpasswordxx
ansible_port: 5986
ansible_connection: winrm
ansible_winrm_transport: ntlm
ansible_winrm_operation_timeout_sec: 120
ansible_winrm_read_timeout_sec: 140
# The following is necessary for Python 2.7.9+ (or any older Python that has backported SSLContext, eg, Python 2.7.5 on RHEL7) when using default WinRM self-signed certificates:
ansible_winrm_server_cert_validation: ignore
ansible_become_user: System
ansible_become_method: runas
```
### Run playbook

```
ansible-playbook -i inventory/docker.ini install-docker.yml
```
### Run iis contianer
Edit the docker.ini and inventory/group_vars/node-windows.yml with respective login details and IP address
```
ansible-playbook -i inventory/docker.ini run-docker-windows.yml
```
You will be able to view contents of index.htm as provided in contents folder, try  server IP:8011 to view via browser!!

That's  it:)
===============================================================================================================================
