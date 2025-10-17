## Creating VMs in Proxmox VE using Ansible with Playbooks 

For information about each major playbook look into each playbook at the top description

Also, this README is WIP

install these on the proxmox ve server
```bash
apt install ansible-core sudo python3 python3-pip
pip install ansible-dev-tools --break-system-packages
```
after cloning the project
```bash
ansible-galaxy collection install -r collections/requirements.yml -p ./collections
```

install on controller node
```bash
python3 -m pip install --user proxmoxer requests --break-system-packages
```
create ansible user on proxmox inside base os, not webgui
```bash
adduser ansibleuser --home /home/ansibleuser --shell /bin/bash
visudo
```

give required priviliges, one method below
```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL
ansibleuser ALL=(ALL:ALL) ALL
```

on remote, not on proxmox
```bash
ssh-keygen -t rsa -b 4096 -C "ansibleuser@192.168.0.222"
ssh-copy-id ansibleuser@192.168.0.222
```

become password management using ansible vault
```bash
mkdir -p group_vars/all
```

check file `group_vars/all/main.yml`

generating ansible vault
```bash
ansible-vault encrypt_string --name 'vault_become_password' --vault-id default@prompt 'sudo_root_password'
```

may be needed
```bash
ansible-vault encrypt_string --name 'pm_api_token_secret' --vault-id default@prompt 'YOUR_VAULTED_SECRET'
```

passwordless ansible playbook execution
```bash
echo 'ansibleVaultPassFromPrevoiusStep' > ~/.vault_pass.txt
chmod 600 ~/.vault_pass.txt
echo 'export ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass.txt' >> ~/.bashrc
source ~/.bashrc
```

> make sure that you API token has the right permissions in the permissions section in the datacentre tab

### General information

to modify/add VMs, make sure to edit just `group_vars/all/vms.yml`

### Playbook usage examples
need to fetch/download isos?
```
ansible-playbook -i inventory/hosts.ini fetch-iso.yml --tags "checkIso"
```

create a bare minimum VM

to create a bare minimum VM make sure these parameters are set inside `group_vars/all/vms.yml`
```yml
vms:
  - name: "ansible-minimal-vm"
    vmid: 600
    ostype: "l26"
    iso_storage: "local"
    iso_path: "iso/"
    iso_file: "debian-13.1.0-amd64-netinst.iso"
```

creating VMs
```bash
ansible-playbook -i inventory/hosts.ini create-vm-from-iso-proxmox.yml --tags "createVMs,createDisks,mountIso,bootOrder"
```

if you want to resize disks only, you have to update vms in `group_vars/all/vms.yml`
```bash
ansible-playbook -i inventory/hosts.ini create-vm-from-iso-proxmox.yml --tags "diskResize"
```

if you want to remove etire vms here is an example of all vms removed defined by `group_vars/all/vms.yml`
```bash
ansible-playbook -i inventory/hosts.ini create-vm-from-iso-proxmox.yml --tags "removeVMs"
```

to update VMs based on changes in vms.yml
```bash
ansible-playbook -i inventory/hosts.ini create-vm-from-iso-proxmox.yml --tags "updateVMs"
```

and to both update VM configs and disk resize
```bash
ansible-playbook -i inventory/hosts.ini create-vm-from-iso-proxmox.yml --tags "updateVMs,diskResize"
```

mount isos etither existing or new and set boot order
```bash
ansible-playbook -i inventory/hosts.ini create-vm-from-iso-proxmox.yml --tags "mountIso,bootOrder"
```

---
Thomas Mozdren, 2025