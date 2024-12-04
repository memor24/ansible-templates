# Notes
The security best practice is to encrypt the inventory files using Ansible vault.
## Ansible Vault
```
ansible-vault create inventory
```
To use this encrypted inventory file, run this command to ask for the vault password set before:
```
ansible-playbook -i inventory --ask-vault-pass playbook.yml
```
To automate the use of encrypted inventory without having to enter the vault password manually each time:
'''
ansible-playbook playbook.yml -i inventory --vault-password-file vault_pass.txt
'''
## Dynamic Inventory
The idea is to avoid text files to store vault passwords, for security reasons and also to use inventory scripts for automation, and an inventory server for scalbility when working with hundreds or thousands of target machines.  For example
```
ansible-playbook playbook.yml -i s3.py
```
Plugins can be enabled in the config file globally at /etc/ansible/ansible.cfg
```
enable_plugins = ini, script, yaml
```
For debugging inventory variable issues on hosts, use:
```
ansible-inventory -i s3.py
```
