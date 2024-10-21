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
<br/> But now we must configure the password: <br/>
<img src="https://github.com/user-attachments/assets/2ce9197d-0cc8-4725-ac3f-127ff8c0e1eb"/>

```Bash
# Stop the httpd service
systemctl stop httpd
# Reset the password (ip address is unique)
keystone-manage bootstrap --bootstrap-password 'admin' --bootstrap-admin-url http://192.168.1.186:5000/v3/ --bootstrap-internal-url http://192.168.1.186:5000/v3/ --bootstrap-public-url http://192.168.1.186:5000/v3/ --bootstrap-region-id RegionOne
# Update and Source keystonerc_admin
vi ~/keystonerc_admin
# Start the http service
systemctl start httpd
```

<img src="https://github.com/user-attachments/assets/7cfabaf7-0c1a-4afc-a41b-f2e38bb6ec40"/>
<img src="https://github.com/user-attachments/assets/cb05833f-58e4-4fab-92f9-558f2a4c54fa"/>
<img src="https://github.com/user-attachments/assets/a76fc557-7ed2-40ed-9676-6564e1fab068"/>
<br/> The UI is now accessable with the password admin!<br/>

## Step 4: Create a project



```Bash
# To avoid an auth-url error source the keystonerc_admin dir
source ~/keystonerc_admin
# Verify necessary variables
env | grep OS_
# Create the project
openstack project create --domain default --description "Demo Project" demo_project
```


<img src="https://github.com/user-attachments/assets/681a9fc0-c21c-4ec5-87f7-e6275fadc84e"/>
<br/> Verify in the UI: <br/>
<img src="https://github.com/user-attachments/assets/b0c3117d-7a97-4c14-b56b-326290054a50"/>

## Step 5: Create a user and assign a role

```Bash
# Create a user
openstack user create --domain default --password 'password123' demo_user
# Assign a role to the demo user
openstack role add --project demo_project --user demo_user member
```

<img src="https://github.com/user-attachments/assets/fb850884-7781-4509-801f-a5dd4449090c"/>

<br/> Verify: <br/>
<img src="https://github.com/user-attachments/assets/02cdef3c-ac4a-4531-8e7d-dc770e09652d"/>
<img src="https://github.com/user-attachments/assets/604ee1a6-7397-47e3-b6b1-fc9c730b4653"/>

## Step 6: Glance Service Registration and Endpoint Verification 

```Bash
# Create the "services" project
openstack project create --domain default --description "Services Project" services
# Register the Glance Service
openstack service create --name glance --description "OpenStack Image Service" image
# Create Endpoints for Glance
openstack endpoint create --region RegionOne image public http://192.168.1.186:9292/v3
openstack endpoint create --region RegionOne image internal http://192.168.1.186:9292/v3
openstack endpoint create --region RegionOne image admin http://192.168.1.186:9292/v3
# Verify Endpoints
openstack endpoint list --service image
openstack endpoint list
```

<img src="https://github.com/user-attachments/assets/e56904d4-3c85-434c-a7b1-cdd827217609"/>
<img src="https://github.com/user-attachments/assets/8eec8e27-d957-47f2-a5e1-eecbf97733ba"/>
<img src="https://github.com/user-attachments/assets/5a7f4007-963d-462a-9baa-c7cb61a99ec6"/>









