# Ansible Automation

# Setup Ansible

apt-get update -y 

apt-get install ansible -y 

ansible --version

![image](https://user-images.githubusercontent.com/49937302/120648028-c9dca900-c4ad-11eb-8d76-dc2515ead0fa.png)

#Create freestyle project on jenkins

Change Repositories to your git ansible-config-mgt repository 

![image](https://user-images.githubusercontent.com/49937302/120649290-28eeed80-c4af-11eb-8c17-544b8cf7bcf2.png)

Configure webhook on ansible-config-mgt repository

![image](https://user-images.githubusercontent.com/49937302/120649878-d06c2000-c4af-11eb-9198-554043b806b1.png)

#Test update README.md and ensure it automatically run

![image](https://user-images.githubusercontent.com/49937302/120648137-e11b9680-c4ad-11eb-8fd4-aa3923a3cb59.png)

#verify files store in the following path on jenkins/ansible servers:

ls -la /var/lib/jenkins/jobs/ansible/builds/1/archive/

![image](https://user-images.githubusercontent.com/49937302/120648210-f395d000-c4ad-11eb-8649-4e1a7294641d.png)


# Setup New Branch & write ansible code

Create new branch "Development" & the following folder & file structure

![image](https://user-images.githubusercontent.com/49937302/120648786-9b130280-c4ae-11eb-967d-12857b718b2e.png)

# Setup Inventory

Update your inventory/dev.yml file with this snippet of code:

nfs-server ansible_host=<NFS IP> ansible_user=ansible
  
web-server1 ansible_host=<Web server1 ip> ansible_user=ansible
  
web-server2 ansible_host=<Web server2 ip> ansible_user=ansible
  
mysql-server ansible_host=<DB server ip> ansible_user=ansible
  
LB-NGINX ansible_host=<LB IP> ansible_user=ansible

[nfs]
  
nfs-server

[webservers]
  
web-server1
  
web-server2

[db]
  
mysql-server

[lb]
  
LB-NGINX

# Create common playbook
  
---
- name: update web, nfs and db servers
  
  hosts: webservers, nfs, db
  
  become: yes
  
  tasks:
  
  - name: ensure wireshark is installed at latest version
  
    yum:
  
      name: wireshark
  
      state: latest

- name: Set time zone & create directory on all servers
  
  hosts: all
  
  become: yes
  
  tasks:
  
    - name: create testfolder directory if it does not exists
  
      file:
  
        path: /home/testfolder
  
        state: directory
  
        mode: '0755'

    - name: Set timezone
  
      timezone:
  
        name: Asia/Singapore
  
        hwclock: local

- name: update LB server
  
  hosts: lb
  
  become: yes
  
  tasks:
  
    - name: ensure wireshark is at the latest version
  
      apt:
  
        name: wireshark
  
        state: latest

# Update Git push with latest code

git add.
  
git commit -m "updated inventory & playbook"
  
git push

Create a Pull request (PR) & merge with master
  
![image](https://user-images.githubusercontent.com/49937302/120651982-e975d080-c4b1-11eb-9ee5-f4d33c65f2de.png)
  
![image](https://user-images.githubusercontent.com/49937302/120652050-f85c8300-c4b1-11eb-88bd-7e9affda1760.png)

![image](https://user-images.githubusercontent.com/49937302/120652067-fdb9cd80-c4b1-11eb-8fb0-53384d7a31cf.png)

![image](https://user-images.githubusercontent.com/49937302/120652085-014d5480-c4b2-11eb-9eff-597b4e20a613.png)
  

![image](https://user-images.githubusercontent.com/49937302/120652112-07433580-c4b2-11eb-9991-04ef44f23ddf.png)

#verify that all the files (build artifacts) to /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/ directory on Jenkins-Ansible server.
  
![image](https://user-images.githubusercontent.com/49937302/120653586-69e90100-c4b3-11eb-91ad-13dd69478e90.png)
  
# Run ansible playbook
  
Verify ansible connection to all server is successful

ansible-playbook -i dev.yml /var/lib/jenkins/jobs/ansible/builds/7/archive/Playbooks/common.yml --check -kK
  
![image](https://user-images.githubusercontent.com/49937302/120652673-951f2080-c4b2-11eb-8bfe-bf048a6480ab.png)
 
ansible-playbook -i dev.yml /var/lib/jenkins/jobs/ansible/builds/8/archive/Playbooks/common.yml -kK

![image](https://user-images.githubusercontent.com/49937302/120652784-ad8f3b00-c4b2-11eb-9832-bad41893962f.png)

Verify wireshark is installed , testfolder created & timezone changed
  
![image](https://user-images.githubusercontent.com/49937302/120652891-c4ce2880-c4b2-11eb-83a7-fb82e28e894e.png)
  
 
