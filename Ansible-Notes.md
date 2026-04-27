
# Ansible notes

## Ansible Ad-Hoc Commands

```sh
ansible all -i 54.87.2.32, -e ansible_user=ec2-user -e ansible_password=DevOps321 -m ping
ansible all -i <inventory_file> -m ping
ansible webservers -i inventory.ini -m ping
ansible 192.168.1.1 -i inventory.ini -m shell -a "uptime"
```

* Here -i indicates inventory file or the IP address of the target host
* The comma after the IP is important to indicate it's a single host inventory
* -e indicates extra variables like username and password*
* -m indicates the module to be used
* Here all is the group name defined in the inventory file, you can also specify a specific host or group of hosts to run the command on.


```sh
ansible all -i 54.87.2.32, -e ansible_user=ec2-user -e ansible_password=DevOps321 -b -m dnf -a "name=nginx state=installed"
ansible all -i 54.87.2.32, -e ansible_user=ec2-user -e ansible_password=DevOps321 -b -m service -a "name=nginx state=stopped"
```
* Here -b indicates to run the command with sudo privileges
* -m indicates the module to be used
* -a indicates the arguments for the module

---

## Ansible Playbook notes

* Playbooks are written in YAML format
* A playbook contains one or more plays
* A play contains one or more tasks
* A task uses a module to perform a specific action on the target host
* You can have multiple tasks in a playbook
* Example playbook to install and start nginx on target host
* You can have multiple plays in a playbook
* To run a playbook, use the command:

```sh
ansible-playbook <playbook_name>.yaml
```



---

## Ansible Inventory notes

* Inventory file contains the list of target hosts
* Hosts can be grouped into groups for easier management
* To specify a group, use square brackets [group_name]
* To run ansible commands on a specific group, use the -i option with the inventory file and specify the group name

Example:
```sh
ansible webservers -i inventory.ini -m ping
```

---

## Variables

* Variables can be defined in inventory file, playbook, or separate variable files as key-value pairs.
* variables can be defined in the inventory file under a group or for each target host. Refern documentation for more details.
* Variables can be used to store values like usernames, passwords, package names, etc.
* To use a variable in a playbook, use the syntax {{ variable_name }}

Example:
```yaml
vars:
  nginx_package: nginx
tasks:
  - name: Install nginx
    dnf:
      name: "{{ nginx_package }}"
      state: installed
```

* variables defined in playbook override those in inventory file
* variables defined in variable files override those in playbook and inventory file
* variables can be given at playlevel or at task level
* variables can be taken from user input using vars_prompt module
* variables can be overridden using -e option in ansible command

**variable preference is as:**
1. command line variables extra vars (-e option)
2. task level variables
3. variable files
4. from prompt
5. play level variables
6. inventory file variables




## Data Types in ansible variables\
1. String
Example:
```yaml
name: "Ansible Course"
```
2. Integer
Example:
```yaml
version: 2
``` 
3. Float
Example:
```yaml
price: 9.99
```
4. Boolean
Example:
```yaml
is_published: true
```
5. List
Example:
```yaml
tags:
  - ansible
  - automation
```  
6. Dictionary
Example:
```yaml
    database:
      name: mydb
      user: dbuser
      password: dbpass
```


## Conditionals in Ansible
Conditionals are used to execute tasks based on certain conditions
The 'when' keyword is used to define conditionals in ansible

Example:
```yaml
tasks:
  - name: Install nginx on CentOS
    dnf:
      name: nginx
      state: installed
    when: ansible_facts['os_family'] == "RedHat"
```
You can use comparison operators like ==, !=, >, <, >=, <=
You can use logical operators like and, or, not
You can use filters to manipulate variables in conditionals

Example:
```yaml
  when: version is version('2.9', '>=')
  when: version | int > 2.9
```  
You can use facts to get information about the target host and use them in conditionals

Example:
```yaml
  when: ansible_facts['memory_mb']['total'] > 2048    
```


## Gathering Facts in Ansible
Facts are system information collected from target hosts
Facts are collected using the setup module
By default, facts are gathered at the beginning of a playbook run
To disable fact gathering, use gather_facts: no at the play level
To gather facts manually, use the setup module in a task

Example:
```yaml
tasks:
  - name: Gather facts
    setup: {}
```    
You can use facts in your playbook to make decisions based on the target host's configuration

Example:
```yaml
  when: ansible_facts['os_family'] == "Debian"
  ```
You can filter the facts gathered using the filter parameter in the setup module

Example:
```yaml
  - name: Gather only network facts
    setup:
      filter: "ansible_*_interfaces"
```
You can view the gathered facts using the debug module

Example:
```yaml
  - name: Display gathered facts
    debug: var=ansible_facts
```
You can cache facts to improve performance using fact caching
Fact caching can be enabled in the ansible.cfg file

Example:
```yaml
[defaults]
fact_caching = jsonfile
fact_caching_connection = /path/to/fact/cache #directory where facts will be stored
fact_caching_timeout = 86400  # 24 hours
```
You can use custom facts by placing scripts in the /etc/ansible/facts.d/ directory on the target host

Example:
```sh
  /etc/ansible/facts.d/custom_fact.sh
  ```
The script should output JSON data
Example output:
```json
  {"custom_fact": "value" }
  ```



## Loops in Ansible
Loops are used to iterate over a list of items and perform tasks for each item

The 'with_items' keyword is used to define loops in ansible

Example:  

tasks:
```yaml
  - name: Install multiple packages
    dnf:
      name: "{{ item }}"
      state: installed
    with_items:  
      - nginx
      - git
      - curl
  # or you can use a variable that contains a list
  - name: Install packages from variable
    dnf:
      name: "{{ item }}"
      state: installed
    with_items: "{{ packages_list }}"

  # or 
  - name: Install packages from variable
    dnf:
      name: "{{ item }}"
      state: installed
    with_items: "{{ lookup('file', 'packages.txt').splitlines() }}"
    
  # or with loop keyword
  - name: Install packages from variable
    dnf:
      name: "{{ item }}"
      state: installed
    loop: 
      - nginx
      - git
      - curl  

```

You can use other loop keywords like with_dict, with_list, with_fileglob, with_nested, etc.

Example with_dict:
```yaml
  - name: Create multiple users
    user:
      name: "{{ item.value.name }}" # item.key gives the key of the dictionary user1 or user2, item.value gives the value of the dictionary 
      state: "{{ item.value.state }}"
    with_dict:
      user1: 
        name: name1
        state: present
      user2: 
        name: name2
        state: present
```



## Functionals in Ansible
Functionals are used to manipulate data in ansible

Filters are used to modify variables and data structures

Example:
```yaml
  - name: Display uppercase name
    debug:
      msg: "{{ name | upper }}"
```      
You can chain multiple filters together

Example:
```yaml
  - name: Display trimmed and uppercase name
    debug:
      msg: "{{ name | trim | upper }}"
```
Commonly used filters include:
  - upper: Converts a string to uppercase
  - lower: Converts a string to lowercase
  - trim: Removes leading and trailing whitespace from a string
  - replace: Replaces occurrences of a substring with another substring 
      
      -  Example: "{{ name | replace(' ', '_') }}" replaces spaces with underscores in the name variable
  - split: Splits a string into a list based on a delimiter 
  
      - Example: "{{ name | split(' ') }}" splits the name variable into a list of words based on spaces
  - sort: Sorts a list in ascending order
  - reverse: Reverses a list or string
  - unique: Removes duplicate items from a list
  - default: Provides a default value if the variable is undefined or empty 
  
      - Example: "{{ name | default('Unknown') }}" returns 'Unknown' if the name variable is undefined or empty
  - join: Joins a list into a string with a specified delimiter 
  
      -  Example: "{{ my_list | join(', ') }}" joins the items in my_list into a string separated by commas and spaces
  - length: Returns the length of a string or list 
  
      -  Example: "{{ name | length }}" returns the number of characters in the name variable

You can use filters in conditionals, loops, and other parts of your playbook

Example in conditional:
```yaml
  when: name | length > 5
```
Example in loop:
```yaml
  with_items: "{{ my_list | sort }}"  
```
You can create custom filters using Python


## Error Handling in Ansible
Error handling is used to manage errors and failures in ansible playbooks

The 'ignore_errors' keyword is used to ignore errors in a task

Example:
```yaml
  - name: Install nginx
    dnf:
      name: nginx
      state: installed
    ignore_errors: true
```
You can use the 'failed_when' keyword to define custom failure conditions for a task

Example:
```yaml
  - name: Check if nginx is installed
    command: rpm -q nginx
    failed_when: "'not installed' in result.stdout" 
 ```

## Ansible configuration file (ansible.cfg)
The ansible.cfg file is used to configure ansible settings

The ansible.cfg file can be located in the following locations (in order of precedence):

1. ANSIBLE_CONFIG environment variable
2. ansible.cfg file in the current directory
3. .ansible.cfg file in the home directory
4. /etc/ansible/ansible.cfg

Commonly used settings in ansible.cfg include:
[defaults]
inventory = /path/to/inventory/file, in this case no need to specif the -i option in ansible commands
remote_user = ec2-user
host_key_checking = False
retry_files_enabled = False
log_path = /var/log/ansible.log 
etc etc



## Ansible tags
Tags are used to categorize tasks in ansible playbooks
Tags allow you to run specific tasks or groups of tasks in a playbook
The 'tags' keyword is used to define tags for tasks
Example:
```yaml
tasks:
  - name: Install nginx
    dnf:
      name: nginx
      state: installed
    tags:
      - install
  - name: Start nginx
    service:
      name: nginx
      state: started
    tags:
      - start
```
 To run tasks with specific tags, use the --tags option in the ansible-playbook command

 Example:
```sh 
ansible-playbook playbook.yaml --tags "install"
```
To skip tasks with specific tags, use the --skip-tags option in the ansible-playbook command

 Example:
```sh
ansible-playbook playbook.yaml --skip-tags "start"
```

## include and import in Ansible
include and import are used to include or import other playbooks or task files into a play

The 'include' keyword is used to include a playbook or task file at runtime

Example:
```yaml
  - name: Include tasks
    include: tasks.yaml
```
The 'import' keyword is used to import a playbook or task file at parse time

Example:
```yaml
  - name: Import tasks
    import_tasks: tasks.yaml
```
The main difference between include and import is that include is dynamic and can be used conditionally
while import is static and is processed at the time of parsing the playbook
Use include when you want to include tasks based on certain conditions
Use import when you want to include tasks unconditionally

## Handlers in Ansible
Handlers are special tasks that are triggered by other tasks when they report a change

Handlers are defined using the 'handlers' section in a playbook
Example:
```yaml
handlers:
  - name: restart nginx   
    service:
      name: nginx
      state: restarted
# To notify a handler from a task, use the 'notify' keyword

  - name: Install nginx
    dnf:
      name: nginx
      state: installed
    notify:
      - restart nginx
```
Handlers are executed at the end of a play, after all tasks have been completed

Handlers are only executed once, even if they are notified multiple times

Handlers can also have conditionals using the 'when' keyword

Example:
```yaml
handlers:
  - name: restart nginx   
    service:
      name: nginx
      state: restarted
    when: ansible_facts['os_family'] == "RedHat"  

```    
You can use handlers to manage services, restart applications, or perform any other actions that need to be triggered by changes in tasks

## Roles in Ansible
Roles are a way to organize and reuse ansible code

Roles are defined in a specific directory structure

A role typically contains the following directories:
  - tasks: Contains the main tasks for the role
  - handlers: Contains the handlers for the role
  - templates: Contains the Jinja2 templates for the role
  - files: Contains the static files for the role
  - vars: Contains the variables for the role 
  - defaults: Contains the default variables for the role
  - meta: Contains the metadata for the role
To use a role in a playbook, use the 'roles' keyword

Example:
```yaml
  - hosts: webservers
    roles:
      - nginx_role
```      
You can pass variables to roles using the 'vars' keyword

Example:
```yaml
  - hosts: webservers
    roles:
      - role: nginx_role
        vars:
          nginx_port: 8080
```          
Roles can be shared and reused across multiple playbooks and projects

You can create your own roles or use roles from Ansible Galaxy, a repository of community-contributed roles

To install a role from Ansible Galaxy, use the ansible-galaxy command


## Dynamic Inventory in Ansible
Dynamic inventory allows you to generate inventory dynamically using scripts or plugins

Dynamic inventory scripts can be written in any programming language that can output JSON or YAML

Ansible provides several built-in dynamic inventory plugins for popular cloud providers like AWS, GCP, Azure, etc.

To use a dynamic inventory script, specify the script path in the -i option of the ansible command
  
Example:
```sh
ansible all -i /path/to/dynamic_inventory_script.py -m ping
```
To use a dynamic inventory plugin, specify the plugin name in the ansible.cfg file or in the -i option

Example in ansible.cfg:
```
[defaults]
inventory = aws_ec2
```

Example in ansible command:
```sh
ansible all -i aws_ec2 -m ping
```
 Dynamic inventory allows you to manage large and dynamic environments more efficiently

 You can also combine static and dynamic inventory by specifying multiple inventory sources in the ansible.cfg file

 Example:
```
 [defaults]

 inventory = /path/to/static_inventory.ini, aws_ec2
 ```

## Plugins in Ansible
Plugins are used to extend the functionality of ansible

Ansible provides several types of plugins, including connection plugins, callback plugins, lookup plugins, filter plugins, etc.

Connection plugins are used to connect to target hosts using different protocols like SSH, WinRM, etc.

Callback plugins are used to customize the output of ansible commands and playbooks

Lookup plugins are used to retrieve data from external sources like files, databases, etc.

Filter plugins are used to manipulate data in ansible using custom filters

To use a plugin, specify the plugin name in the appropriate section of the ansible.cfg file or in the playbook

Example in ansible.cfg:
```
[defaults]
stdout_callback = yaml
```
Example in playbook:
```yaml
  - name: Display file content
    debug:
      msg: "{{ lookup('file', '/path/to/file.txt') }}"
```
You can also create your own custom plugins using Python

Custom plugins should be placed in the appropriate plugin directory and follow the ansible plugin structure

## Ansible Vault

Ansible Vault is used to encrypt sensitive data in ansible playbooks and inventory files

To create an encrypted file using ansible vault, use the command: ansible-vault create <file_name>

To encrypt an existing file, use the command: ansible-vault encrypt <file_name>

To decrypt an encrypted file, use the command: ansible-vault decrypt <file_name>

To edit an encrypted file, use the command: ansible-vault edit <file_name>

To run a playbook with encrypted files, use the --ask-vault-pass option in the ansible-playbook command

Example:
```sh
ansible-playbook playbook.yaml --ask-vault-pass
```
You can also use the --vault-password-file option to specify a file containing the vault password

Example:
```sh
ansible-playbook playbook.yaml --vault-password-file /path/to/vault_password_file
```
You can encrypt specific variables in a playbook using the 'vars' section with the 'vault' keyword

Example:
```yaml
vars:
  secret_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          6162636465666768696a6b6c6d6e6f70
          7172737475767778797a7b7c7d7e7f80
```          
password comes from group vars and vault.yaml, it can be placed anywhere, its like a var file

use --ask-vault-pass to prompt for vault password

or use --vault-password-file to provide a file that contains the vault password, file can be created anywhere, can be specifiesn in ansible.cfg or in command line or in environment variable etc etc

You can use ansible vault to securely manage sensitive data like passwords, API keys, etc. in your ansible projects

## Secrets manager
Ansible can integrate with various secrets management tools like HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, etc.

To use a secrets manager, you need to install the appropriate ansible collection or plugin for the secrets manager

You can then use the lookup plugins provided by the collection or plugin to retrieve secrets from the secrets manager

Example using HashiCorp Vault:
```yaml
  - name: Retrieve secret from HashiCorp Vault
    debug:
      msg: "{{ lookup('hashi_vault', 'secret/data/mysecret:password ') }}"
```      
Example using AWS Secrets Manager:
```yaml
  - name: Retrieve secret from AWS Secrets Manager
    debug:
      msg: "{{ lookup('aws_secretsmanager', 'mysecret:SecretString') }}"  
```      
Using a secrets manager allows you to securely manage and retrieve sensitive data in your ansible projects without hardcoding them in playbooks or inventory files 

## SSM Parameter Store
Ansible can integrate with AWS Systems Manager (SSM) Parameter Store to retrieve parameters and secrets

To use SSM Parameter Store, you need to install the amazon.aws collection

You can then use the aws_ssm lookup plugin to retrieve parameters from SSM Parameter Store

Example:
```yaml
  - name: Retrieve parameter from SSM Parameter Store
    debug:
      msg: "{{ lookup('amazon.aws.aws_ssm', 'my_parameter_name', region='us-east-1' ) }}"  
```      
Using SSM Parameter Store allows you to securely manage and retrieve parameters and secrets in your ansible projects without hardcoding them in playbooks or inventory files

## Ansible forks
Forks are used to define the number of parallel processes ansible can use to execute tasks on target hosts

By default, ansible uses 5 forks

You can change the number of forks by specifying the 'forks' setting in the ansible.cfg file

Example:
```
[defaults]
forks = 10
```
You can also specify the number of forks using the -f or --forks option in the ansible command

Example:
```sh
ansible all -i inventory.ini -m ping -f 10
```
Increasing the number of forks can improve performance when managing a large number of target hosts



## Dynamic Inventory with AWS EC2 Instances 
To use dynamic inventory with AWS EC2 instances, you need to install the amazon.aws collection

You can then use the aws_ec2 inventory plugin to dynamically generate inventory from your AWS EC2















