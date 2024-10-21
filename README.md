# OpenStack-Installation-and-Configuration
<h2>Description</h2>
<br/> In this project we will install and configure OpenStack
<br />
<br/>  <br/>
<img src="https://github.com/user-attachments/assets/0923c643-abbe-4d83-9494-5130a1aefdb4"/>


<h2>Languages and Utilities Used</h2>

Bash

<h2>Environments Used </h2>

<b>CentOS Stream release 9 release 5.14.0-503.el9.x86_64, OpenStack </b>

<h2>Project walk-through:</h2>
<br/>
<p align="center">

### **Prerequisites**  
- Temporarily disable SELinux and Firewalld (Do not do in production).   
- Configure the network settings to ensure a static IP
- Remove epel-release to avoid package conflicts
- At least 6GB RAM, 20GB Disk Space, 3-4 CPU cores (recommended)

## Step 1: Install OpenStack

```Bash
# View available openstack versions
yum search centos-release-openstack
# Install the Antelope openstack version
yum install -y centos-release-openstack-antelope.noarch
# Update the system
yum update -y
```

<img src="https://github.com/user-attachments/assets/cea1a9d7-0645-4720-947a-267e242798a5"/>
<img src="https://github.com/user-attachments/assets/7f366068-82c3-42dc-8f7e-2208aaf86774"/>
<br/> Note: for this demo I am using the Antelope OpenStack version because it is the most recent version listed with the latest features, security updates, and performance improvements. <br/>
<img src="https://github.com/user-attachments/assets/677a0afb-d90d-478b-95ef-be0cfb2fb809"/>
<br/> Handle any dependency conflicts if they occur such as this one
(This issue was caused between the installed version of ansible-core and the OpenStack ansible-core package from the centos-openstack-antelope repository. The openstack-ansible-core package is obsoleting the installed version of ansible-core, leading to a package conflict during the update. <br/>
<br/> To solve I ran: <br/> 

```Bash
sudo yum remove ansible ansible-core
sudo yum update -y
```

<br/> Result: <br/>
<img src="https://github.com/user-attachments/assets/9af41849-c4f9-40b8-b1ff-316833c066dc"/>

## Step 2: Install PackStack

```Bash
# Install packstack
yum install -y openstack-packstack
# Verify packstack installation
rpm -qa | grep packstack
```

<img src="https://github.com/user-attachments/assets/7521d3d4-9ca7-4d4c-a2fa-459a3a612bfe"/>
<img src="https://github.com/user-attachments/assets/71dde2a1-de61-4137-842e-4ea7e8e90fcc"/>

## Step 3: Run the PackStack installer

```Bash
# Use PackStack to create OpenStack
packstack --allinone
```

<img src="https://github.com/user-attachments/assets/8a454f0c-fb62-48e7-a18c-a37b3c3f5413"/>
<br/> The build has been completed but there are errors and the UI is inaccessable <br/>

<img src="https://github.com/user-attachments/assets/3794771b-6e19-4da9-9001-67f7d6e0b14e"/>
<img src="https://github.com/user-attachments/assets/8aa25662-5119-4de6-a313-088c45a1ed84"/>

<br/> To solve this I will run the following: <br/>

```Bash
# Disable the NetworkManager service
systemctl disable NetworkManager
# Enable the crb repo
dnf config-manager --set-enabled crb
# Rebuild the DNF cache and update repos
dnf clean all && dnf makecache
# Install pyxattr
dnf install -y python3-pyxattr
# Install openstack glance
dnf install -y openstack-glance
# Reinitiate the build (will be unique)
packstack --answer-file /root/packstack-answers-20241020-180438.txt

```


<img src="https://github.com/user-attachments/assets/2ecabda8-1b91-4969-8d55-72bda3fea271"/>
<img src="https://github.com/user-attachments/assets/381d811a-1ac9-480d-887e-5f504ed5fc8b"/>
<img src="https://github.com/user-attachments/assets/5bb52f9f-d346-4b26-b5d1-fad4f72afd1a"/>





## Step 4:

```Bash

```

<img src=""/>
