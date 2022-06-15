# Jenkins Job Enchancement

# configure new directory & install Copy Arctifact plugin on Jenkins

sudo mkdir /home/ubuntu/ansible-config-artifact

sudo chmod -R 777 ansible-config-artifact

![image](https://user-images.githubusercontent.com/49937302/120910153-075c5480-c6af-11eb-9e44-91fc0513019f.png)

Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

![image](https://user-images.githubusercontent.com/49937302/120910188-57d3b200-c6af-11eb-86f4-a73f538a1e25.png)

![image](https://user-images.githubusercontent.com/49937302/120910195-5efac000-c6af-11eb-9827-648a3c170619.png)

Create a new Freestyle project: save_artifacts & configure upstream project as ansible

![image](https://user-images.githubusercontent.com/49937302/120910197-63bf7400-c6af-11eb-81a2-fc26bc3a5255.png)

![image](https://user-images.githubusercontent.com/49937302/120910449-b306a400-c6b1-11eb-961a-b489f88269c3.png)

![image](https://user-images.githubusercontent.com/49937302/120910480-e0ebe880-c6b1-11eb-9225-12a84518d120.png)

Test running job from ansible Verify build successfully completed 

![image](https://user-images.githubusercontent.com/49937302/120910203-75088080-c6af-11eb-8fbf-3a693993f466.png)

# Create new branch

Create new branch: “refactor”

# Create static assignment folder

The static-assignments folder is where all other children playbooks will be stored. This is merely for easy 

organization of your work.

Create Static-assignments folder , under static-assignment, create common-del.yml

Move Playbooks/common.yml to Static-assignments/common.yml

# Create site.yml

Within playbooks folder, create a new file and name it site.yml – This file will now be considered as an entry point

into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words,

site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created 

previously. 

Below is the folder/file layout

![image](https://user-images.githubusercontent.com/49937302/120910208-85206000-c6af-11eb-8d9f-dc4877827c3c.png)

Site.yml

---

- hosts: all

- import_playbook: ../static-assignments/common.yml

---

- name: uninstall wireshark on web, nfs and db servers
  
  hosts: webservers, nfs, db
  
  become: yes
  
  tasks:
  
  - name: ensure wireshark is uninstalled
  
    yum:
      
      name: wireshark
      
      state: removed

- name: Remove testfolder
  
  hosts: all
  
  become: yes
  
  tasks:
  
    - name: Recursively Remove directory
      
      file:
        
        path: /home/testfolder
        
        state: absent

    - name: uninstall wireshark
      
      hosts: lb
      
      become: yes
      
      tasks:
      
    - name: ensure wireshark is is uninstalled
      
      apt:
      
        name: wireshark
      
        state: absent
      
        autoremove: yes
      
        purge: yes
      
        autoclean: yes

# Push code to git

git add .

git commit -m “added refactor”

git push --set-upstream origin refactor

git status

Merge refactor to master

![image](https://user-images.githubusercontent.com/49937302/120910280-32937380-c6b0-11eb-99b0-003bccc0b759.png)

# run ansible to the servers

cd /home/ubuntu/ansible-config-artifact/Inventory

ansible-playbook -i dev.yml /home/ubuntu/ansible-config-artifact/Playbooks/site.yml -kK

![image](https://user-images.githubusercontent.com/49937302/120910292-49d26100-c6b0-11eb-8f5a-6722d283c2f1.png)

Verify wireshark & testfolder is removed

![image](https://user-images.githubusercontent.com/49937302/120910296-522a9c00-c6b0-11eb-84f6-9e6b7773f08f.png)

Run adhoc command to verify wireshark & testfolder have been removed

ansible all -i dev.yml -a "ls -la /home" -kK

ansible all -i dev.yml -a "wireshark --version" -kK

![image](https://user-images.githubusercontent.com/49937302/120910301-5e165e00-c6b0-11eb-800f-5cf07da06844.png)

# Configure UAT Webservers with ansible galaxy webserver

Provision 2 web server using rhel ami

![image](https://user-images.githubusercontent.com/49937302/120910303-68385c80-c6b0-11eb-8ac2-fabed1ad7367.png)

#create ansible galaxy web server role

mkdir roles

cd roles

ansible-galaxy init webserver

![image](https://user-images.githubusercontent.com/49937302/120910313-738b8800-c6b0-11eb-9c70-e2e889c85e25.png)

![image](https://user-images.githubusercontent.com/49937302/120910318-7ab29600-c6b0-11eb-9d48-f6a499151dd7.png)

#update tasks/main.yml

---

- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo

  become: true

  ansible.builtin.git:

    repo: https://github.com/vince87-87/tooling.git

    dest: /var/www/html

    force: yes

- name: copy html content to one level up

  become: true

  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started

  become: true

  ansible.builtin.service:

    name: httpd

    state: started


- name: recursively remove /var/www/html/html/ directory

  become: true

  ansible.builtin.file:

    path: /var/www/html/html

    state: absent

# Change roles_path to point it to correct directory

sudo vi /etc/ansible/ansible.cfg

![image](https://user-images.githubusercontent.com/49937302/120910321-843bfe00-c6b0-11eb-870f-47516d80d126.png)

update uat.yml -> inventory for uat web servers , create uat-webservers.yml -> to use of roles & update site.yml to import playbook

![image](https://user-images.githubusercontent.com/49937302/120910583-8d2dcf00-c6b2-11eb-936a-38793c215e68.png)

![image](https://user-images.githubusercontent.com/49937302/120910595-be0e0400-c6b2-11eb-82da-7d5f557a054d.png)

![image](https://user-images.githubusercontent.com/49937302/120910628-f9103780-c6b2-11eb-94a1-2c4b6df588e8.png)

push the changes to github

![image](https://user-images.githubusercontent.com/49937302/120910324-87cf8500-c6b0-11eb-9fcc-9d708dcf1676.png)

create a Pull Request and merge them to master branch

![image](https://user-images.githubusercontent.com/49937302/120910326-8c943900-c6b0-11eb-9fa1-670f678dcc68.png)

ansible-playbook -i /home/ubuntu/ansible-config-artifact/Inventory/uat.yml /home/ubuntu/ansible-config-artifact/Playbooks/site.yml -kK

Verify ansible playbook completed!!

![image](https://user-images.githubusercontent.com/49937302/120910328-94ec7400-c6b0-11eb-8b64-8f3f66c43cd7.png)

Access the webserver via browser:

http://UAT-WEBSERVER-IP/index.php

![image](https://user-images.githubusercontent.com/49937302/120910330-99b12800-c6b0-11eb-9602-f4cd08fab692.png)
  
http://UAT2-WEBSERVER-IP/index.php

![image](https://user-images.githubusercontent.com/49937302/120910336-9f0e7280-c6b0-11eb-849c-cf6c71080577.png)
