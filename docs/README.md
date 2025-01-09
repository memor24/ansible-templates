----------------------------------
## Playbook
```yaml
- 
  name: play
  hosts: all, *, localhost, Group1, Host1  # ==> read from inventory
  tasks:
    - name: run command date
      command: date  # ==> module
    - name: run script
      script: test.sh  # ==> module
```

list of ansible modules :
```bash
ansible-doc -l
```
run ansible playbook :
```bash
ansible-playbook playbook.yml
```
**dynamic inventory**
```bash
ansible-playbook -i inventory.txt playbook.yml
                    inventory.py
```
----------------------------------------------------------
## Inventory
for target system (list of servers) in yaml or ini

* file
* /etc/ansible/hosts

```ini
sever1.company.com
sever1.company.com

[group_1] # Group
sever3.company.com
sever4.company.com

[group_2]
sever5.company.com
sever6.company.com

[main:children]  # Group of Groups with ':children'
group_1
group_2

[main:vars] # apply variables to these groups of groups with ':vars'
halon_system_timeout=30
```

#### Inventory Parameters (these are variables):
* ansible_host (dns, ip)
* ansible_connection (ssh, winrm, localhost) (linux, windows, localhost)
* ansible_port (22, 5986)
* ansible_user (root, administrator) (linux, windows)
* ansible_ssh_pass ( password )

set **Alias** and **Parameters** :

```ini
web  ansible_host=sever1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=1234
db  ansible_host=sever2.company.com ansible_connection=ssh ansible_user=root
mail  ansible_host=sever3.company.com ansible_connection=winrm ansible_user=administrator
localhost  ansible_connection=localhost
```
### ssh config :
```
/etc/ansible/ansible.cfg ==> ansible config file
#host_key_checking=false ==> uncomment it for not check key
```
----------------------------------
## Modules
* System (User, Group, Hostname, Iptables, Service, Systemd, ...)
* Commands (Command, Script, Raw, Expect, ...)
* Files (Archive, File, Copy, Template, ...)
* Database (Mongodb, Mysql, Postgressql)
* Cloud (Amazon, Google, Asure)
* windows
* ...
----------------------------------
## Variable

variable in main plybook file :
```yaml
- 
  name: play
  hosts: localhost
  vars:
    dns_server: 4.2.2.4
  tasks:
```
define var in inventory file for that specific host:
```ini
web dns_server=4.2.2.4 key=value
```
using define variable :
```yaml
  name: play
  hosts: localhost
  vars:
    dns_server: 4.2.2.4
  tasks:
    - lineinfile:
        path: /etc/resolv.conf
        line: 'nameserver {{ dns_server }}' # use var with '{{}}'
```
variable in separate file "var.yml" :
```yaml
var1: value1
var2: value2
```
using var file in playbook in specific dir :
```yaml
- 
  hosts: all
  vars_files:
    - /vars/var.yml
  tasks:
```
----------------------------------
## Conditionals

for using conditionals , use 'when' \
> expressions in 'when' is  **==**, **!=**, **or** , **and** , **is** , **is not** , **>** , **<**

```yaml
- 
  name: play
  hosts: localhost
  tasks:
    - service: name=mysql state=started
      when: ansible_host == sever1.company.com 
      # just run in sever1.company.com server
    
    - service: name=httpd state=started
      when: ansible_host == sever1.company.com or ansible_host == sever2.company.com 
```

store command output in variable use 'register' module :
```yaml
- 
  name: play
  hosts: localhost
  tasks:
    - command: service httpd status
      register: command_output

    - debug: # show during terminal when start playbook
        msg: '{{ command_output }}'
```
----------------------------------
## Loops
> loop is new version of with_* after ansible 2.5
```yaml
- 
  name: start service
  hosts: localhost
  tasks:
    - service: name='{{ item }}' state=started
      with_items: # or it can be 'loop:'
        - mongo
        - mysql
        - httpd
```
----------------------------------
## Roles

for include playbook in our main yml file use '- include < playbook name >' :
```yaml
- include install_dependencies.yml
- include configure_web_server.yml
- include setup_start_app.yml
```

it's sample of each playbook
* install_dependencies.yml
* configure_web_server.yml
* setup_start_app.yml
```yaml
- 
  name: role name
  hosts: localhost
  roles:
```

this is dir structure for using roles:
```
ansible-project/
    inventory.txt
    app.yml
    roles/
        mysql/
            tasks/
            handlers/
            library/
            files/
            templates/
            vars/
            defaults/
            meta/
        nodejs/
            tasks/
            defaults/
            meta/
```

for using role in our main file :
```yaml
---
- hosts: webservers
  roles:
    - mysql
    - nodejs
```

for use tasks and vars in separate file:
```yaml
- 
  name: play
  hosts: localhost
  vars_files:
    - var.yml
  tasks:
    - include: task.yml
```

----------------------------------
## host_vars , group_vars

dir structure for host_vars :
```
playbook.yml
host_vars/
    server1.yml
    server2.yml
```

server1.yml :
```
ansible_host=10.12.90.232
ansible_ssh_pass=qazwsx
```

dir structure for group_vars (same var for all server in this group) :
```
playbook.yml
group_vars/
    db.yml
```
----------------------------------
## Include Tasks

dir structure for separate tasks :

```yaml
- 
  name: separate tasks
  hosts: localhost
  tasks:
  - include: tasks/server.yml
  - include: tasks/db.yml
```

tasks/db.yml :
```yaml
- name: install dep for db
- name: config db
```

tasks/server.yml :

```yaml
- name: install dep for app
- name: config app
```
----------------------------------
## Ansible Galaxy

create new role :
```bash
ansible-galaxy init new-role-name
```

search in public roles :
```bash
ansible-galaxy search role-name
```
----------------------------------
## Error Handling

by default if one of server get error during runtime , just that server stoped but in this case ,even one server get error during runtime all server stoped with set this parameter **any_errors_fatal:** to **true** :
```yaml
- name: error handling
  hosts: all_servers
  any_errors_fatal: true
  tasks:
```

in this task, it's not very important , if fail during runtime , ignore error and continue with set **ignore_errors:** parameter to **yes** :
```yaml
- name: just echo 'hello wolrd'
  command: /opt/hello.sh
  ignore_errors: yes
```

if find **ERROR** word in file, value of file is **command_output.stdout** , this task gonna fail with set **failed_when:** parameter :
```yaml
- name: important script
  command: /var/log/db.log
  register: command_output
  failed_when: "'ERROR' in command_output.stdout"
```
----------------------------------
## Jinja2 Template Engine

### Filters :

#### Strings
```
my name is {{ my_name }} ==> subtitle
my name is {{ my_name | upper }} ==> for upper case of value
my name is {{ my_name | lower }} ==> for lower case of value
my name is {{ my_name | title }} ==> for title
my name is {{ my_name | replace("sepehr", "sina") }} ==> for replca "sepehr" word with "sina"
my name is {{ my_name | default("sepehr") }} {{ family_name }} ==> for default value
```

#### List and Set
```
{{ [1,2,3] | min }}  ==> 1
{{ [1,2,3] | max }}  ==> 3
{{ [1,2,3,3] | unique }} ==> 1,2,3
{{ [1,2,3] | union([4,5]) }} ==> 1,2,3,4,5
{{ [1,2,3,4] | intersect([4,5]) }} ==> 4
{{ 100 | random }} ==> random number from 0 to 100
{{ ["sepehr", "is", "too", "smart" ] | join("") }} ==> sepehr is too smart
```

### Condition
```
{% if x > y %}
  x is bigger
{% elif z == 0 %}
  negetive var value
{% else %}
  y is smaller
{% endif %}
```

### Loop
```yaml
var:
  z: [1,2,3,4,5]
```

```
{% for each in z %}
  {{ each }}
{% endfor %}
```

----------------------------------
## Lookups

credential.csv file is :
```
target1,password
target2,password
```
for get password from credential.csv file use **lookup** :
```
{{ lookup('csvfile' , 'target1 file=/tmp/credential.csv delimiter=,') }} ==> password
```

> it can use with **ini** , **dns** , **mongodb**, ... files
----------------------------------
## Ansible Vault

```
ansible-vault encrypt inventory.txt ==> encrypt inventory file
ansible-vault decrypt inventory.txt ==> decrypt inventory file
ansible-vault rekey inventory.txt ==> change password
ansible-vault view inventory.txt ==> view value of encrypted file
ansible-vault create inventory.txt ==> create encrypt file
ansible-vault edit inventory.txt ==> edit encrypted file
```

for run playbook with encrypted files :
```
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
ansible-playbook -i inventory.ini playbook.yml --vault-password-file ./pass.txt ==> read pass from text file
ansible-playbook -i inventory.ini playbook.yml --vault-password-file ./pass.py ==> read pass from py script 
```
----------------------------------
## important ansible var
```
ansible_os_family ==> this var for specify os type (RedHat,Ubuntu,Debian)
ansible_system ==> (windows,linux)
inventory_hostname ==> name of server
```

## Block and Rescue and Always

### block
for use same module in many task we create block , in this example use **ignore_errors** for all **command** tasks :

```yaml
- 
  name: block
  hosts: localhost
  tasks:
  - block:
        - command: "ls /home"
          register: home_out

        - command: "ls /root"
          register: root_out

        - command: "ls /temp"
          register: home_out
    ignore_errors: yes
```

### rescue and always

it's like (try,catch) in ansible is (block,rescue) that means if first task under **block** will be fail , the task under **resume** will be run and the task under **always** will always executes:

```yaml
- 
  name: block
  hosts: localhost
  tasks:
  - block:
        - command: "ls /home"
          register: home_out
    rescue:
        - debug:
            msg: "this task is gonna run if 'ls /home' be failed"
    always:
        - debug:
            msg: "this will always executes"
```

## include_* vs import_*

import_ * are static and include_* are dynamic

All import* statements are pre-processed at the time playbooks are parsed. \
All include* statements are processed as they encountered during the execution of the playbook. \
So import is static, include is dynamic.

include: ( include_playbook, include_tasks, include_vars, include_role ) \
import: ( import_playbook, import_tasks, import_vars, import_role )

```yaml
- name: import and include
  hosts: localhost
  tasks:
    - include_task: install_webserver_{{ ansible_os_family }}.yml # it's run without problem
    - import_task: install_webserver_{{ ansible_os_family }}.yml # it's going to error, beacase can't parse var during plybook execution
```

## delegate_to, local_action

**delegate_to** for run just for specific task in different server but **local_action** for localhost + all servers

```yaml
  name: delegate_to
  hosts: server1
  tasks:
    - name: ls /home
      command: "ls /home"
      register: home_out
      delegate_to: localhost # just run in localhost
      run_once: true # just run once in localhost not another server in the group
```

```yaml
  name: local_action
  hosts: server1
  tasks:
    - name: ls /home
      local_action: shell 'ls /home' # run on localhost + all servers
      register: home_out
```
--------------------------------------
## Ansible best practices
Ansible <a href="https://github.com/devopshobbies/ansible-templates/blob/master/part33-playbook-best-practices/README.md" target="_blank" rel="noopener noreferrer">best practices</a>

----------------------
### PS: YAML

Key/Value :
```yaml
name: jane
family: doe
age: 18
```

Array / List :
```yaml
courses:
- math
- physucs
- biology
```

Dictionary / Map:
```yaml
grade:
    math: 70
    physics: 80
    biology: 100
```

List / Map / Key Value :
```yaml
Fruits:
    - Apple:
        Calories: 105
        Fat: 0.4 g
        Carbs: 0.4 g
    - Grape:
        Calories: 62
        Fat: 0.3 g
        Carbs: 17 g
```

Dictionary = Unordered \
List = Ordered \

