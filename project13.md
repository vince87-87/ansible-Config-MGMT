# Ansible Dynamic Assignments (Include) and Community Roles

# Introducing Dynamic Assignment Into Our structure

The module that enables dynamic assignments is include.

Hence,

import = Static

include = Dynamic

Create a new folder, name it dynamic-assignments. Then inside this folder, create a new file and name it env-vars.yml.

We will instruct site.yml to include this playbook later.

This will be the file structure
    
Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.

For this reason, we will now create a folder to keep each environment’s variables file. Therefore, create a new folder env-vars, then for each environment, create new YAML files which we will use to set variables.

![image](https://user-images.githubusercontent.com/49937302/122769096-d4cd7100-d2d6-11eb-8b07-4200c348ce79.png)


Paste the below code to env-vars.yml

---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - uat.yml
            - stage.yml
            - prod.yml
            - dev.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always

![image](https://user-images.githubusercontent.com/49937302/122768321-127dca00-d2d6-11eb-9dbe-be527e9b2415.png)

We used include_vars syntax instead of include, this is because Ansible developers decided to separate different features of the module. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:
include_role
include_tasks
include_vars
In the same version, variants of import were also introduces, such as:

import_role
import_tasks
We made use of a special variables {{ playbook_dir }} and {{ inventory_file }}. {{ playbook_dir }} will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. {{ inventory_file }} on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.
We are including the variables using a loop. with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

# Update site.yml with dynamic assignment

Update site.yml file to make use of the dynamic assignment. (At this point, we cannot test it yet. We are just setting the stage for what is yet to come. So hang on to your hats)

site.yml should now look like this.

---

- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
  
 # Download mysql ansible roles
 
 On Jenkins-Ansible server make sure that git is installed with git --version, then go to ‘ansible-config-mgt’ directory and run

git init

git pull https://github.com/vince87-87/ansible-Config-MGMT.git

git remote add origin https://github.com/vince87-87/ansible-Config-MGMT.git

git branch roles-feature

git switch roles-feature

Inside roles directory create your new MySQL role with ansible-galaxy install geerlingguy.mysql and rename the folder to mysql

mv geerlingguy.mysql/ mysql

![image](https://user-images.githubusercontent.com/49937302/122769715-6341f280-d2d7-11eb-93e3-f9ec5cabc175.png)

![image](https://user-images.githubusercontent.com/49937302/122769725-663ce300-d2d7-11eb-8936-3b65f3a661f6.png)

edit roles configuration to use correct credentials for MySQL required for the tooling website under roles/mysql/defaults/main.yml -> to change the variable to the desire credential and configuration

![image](https://user-images.githubusercontent.com/49937302/122770179-c5025c80-d2d7-11eb-9788-0ca79224eac8.png)

![image](https://user-images.githubusercontent.com/49937302/122769988-9c7a6280-d2d7-11eb-9c05-47d4f865fef5.png)

Now it is time to upload the changes into your GitHub:

git add .

git commit -m "add mysql role files into GitHub"

git push --set-upstream origin roles-feature

Now, if you are satisfied with your codes, you can create a Pull Request and merge it to main branch on GitHub.

Verify role can be successfully applied to the uat db server

![image](https://user-images.githubusercontent.com/49937302/122772056-94bbbd80-d2d9-11eb-8253-0c1d1daca9bf.png)

# Loadbalancer roles

We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

Nginx

Apache

Since cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one - this is where you can make use of variables.

Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively.

Set both values to false like this enable_nginx_lb: false and enable_apache_lb: false.

Declare another variable in both roles load_balancer_is_required and set its value to false as well

Update both assignment and site.yml files respectively

loadbalancers.yml file

![image](https://user-images.githubusercontent.com/49937302/122770722-4fe35700-d2d8-11eb-88ae-86fdafd19d35.png)

site.yml

![image](https://user-images.githubusercontent.com/49937302/122770803-64275400-d2d8-11eb-9c0d-4a6d39f6d1a9.png)

Now you can make use of env-vars\uat.yml file to define which loadbalancer to use in UAT environment by setting respective environmental variable to true.

You will activate load balancer, and enable nginx by setting these in the respective environment’s env-vars file.

env-vars/uat.yml

![image](https://user-images.githubusercontent.com/49937302/122770910-81f4b900-d2d8-11eb-8ce5-827a3a4e4cca.png)

On ansible/jenkins servers, run the below command to install galaxy apache & nginx

ansible-galaxy install geerlingguy.nginx

cd /home/ubuntu/ansible-config-artifact/roles/

mv geerlingguy.nginx/ nginx

mv geerlingguy.apache/ apache

cp -r nginx /home/ubuntu/ansible-Config-MGMT/roles/ 

cp -r apache /home/ubuntu/ansible-Config-MGMT/roles/ 

![image](https://user-images.githubusercontent.com/49937302/122771245-d5670700-d2d8-11eb-8350-3841ffb04a1b.png)

![image](https://user-images.githubusercontent.com/49937302/122771264-d861f780-d2d8-11eb-9cd2-74a1aacce143.png)

![image](https://user-images.githubusercontent.com/49937302/122771289-dbf57e80-d2d8-11eb-934f-aef0cf855e17.png)

![image](https://user-images.githubusercontent.com/49937302/122771297-e0219c00-d2d8-11eb-8412-c3d952621609.png)

Push the roles to github

Git add .

Git commit -m "added nginx & apache roles"

Git push

![image](https://user-images.githubusercontent.com/49937302/122771394-f891b680-d2d8-11eb-8dff-0ad341750098.png)

Edit roles/apache/defaults/main.yml & roles/nginx/defaults/main.yml

add the below variables

#load balancer variable

enable_apache_lb: false

load_balancer_is_required: false

![image](https://user-images.githubusercontent.com/49937302/122771736-46a6ba00-d2d9-11eb-8557-008e98d2615d.png)

Verify roles are installed on the loadbalancer servers

ansible-playbook -i Inventory/ Playbooks/site.yml -kK

![image](https://user-images.githubusercontent.com/49937302/122772217-b87f0380-d2d9-11eb-8ca0-6687a6d476b4.png)
