## playbook1 : step by step tasks

```
ansible-playbook playbook1.yml
```
## playbook2 : using community role

Download the role from ansible galaxy:
```
ansible-galaxy role install geerlingguy.jenkins
```
Then run the playbook updated with the role:
```
ansible-playbook playbook1.yml
```
