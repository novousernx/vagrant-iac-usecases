# Vagrant IaaC (Infra as a Code) on AWS - Demo
This is a simple repo to demonstrate how to implement IaaC using Vagrant on AWS. We have implemented **IaaC (Infra-As-A-Code)** and CaaC(Configuration-as-code) using Vagrant and Ansible. 

**See More use cases at [vagrant-iaac-usecases](https://github.com/ginigangadharan/vagrant-iaac-usecases)**

Any questions : [Email](mailto:net.gini@gmail.com) | [LinkedIn](http://bit.ly/gineesh) | [www.techbeatly.com](www.techbeatly.com)

This **IaaC** will create a Virtual Machine in **AWS** in which **nginx** webserver will be installed automatically. Also, website content will be copied from [github sample site](https://github.com/ginigangadharan/vagrant-aws-iaas-demo-site.git). We will also enable firewall and root login securities automatically using ansible provisioning. 

  * [How to use this repo - Quick Overview](#how-to-use-this-repo---quick-overview)
  * [Step 1 - Configure Pre-requisites](#step-1---configure-pre-requisites)
    + [1.1 Vagrant Installation](#11-vagrant-installation)
    + [1.2 Plugin Installation - vagrant-aws](#12-plugin-installation---vagrant-aws)
      - [Notes](#notes)
    + [1.3 Setup Provider Environment - AWS](#13-setup-provider-environment---aws)
      - [Box Image](#box-image)
  * [Step 2 - Create our Virtual Machine - AWS Instance](#step-2---create-our-virtual-machine---aws-instance)
    + [2.1 Vagrantfile](#21-vagrantfile)
    + [2.2 Provisioning](#22-provisioning)
      - [When provisioning happens ?](#when-provisioning-happens--)
    + [2.3 Let's create the VM](#23-let-s-create-the-vm)
    + [2.4 Verify our instance](#24-verify-our-instance)
    + [2.5 Stop or Delete VM](#25-stop-or-delete-vm)
    + [Troubleshooting](#troubleshooting)
      - [vagrant up hang at "==> default: Waiting for SSH to become available..."](#vagrant-up-hang-at------default--waiting-for-ssh-to-become-available-)


## How to use this repo - Quick Overview

1. Install Vagrant
2. Configure AWS credentials.
3. Clone this repo to your working directory
```
git clone https://github.com/ginigangadharan/vagrant-aws-iaas-demo.git
```
4. switch to ```vagrant-aws-iaas-demo``` directory and run ```vagrant up```

See below for detailed instructions.

## Step 1 - Configure Pre-requisites
To test this demo, you need to follow below items.

### 1.1 Vagrant Installation
[Download](https://www.vagrantup.com/downloads.html) and install vagrant on your host/workstation. (Your laptop or a control server)

Refer [Vagrant Documentation](https://www.vagrantup.com/docs/installation/) for more details.

### 1.2 Plugin Installation - vagrant-aws
Vagrant is coming with support for VirtualBox, Hyper-V, and Docker. If you want to create your virtual machine on any other environment (like AWS or Azure) Vagrant still has the ability to manage this but only by using  providers plugins. For this case we are using aws and we use **vagrant-aws** plugin.
You may refer [vagrant-aws](https://github.com/mitchellh/vagrant-aws) in github for the same.

```
# vagrant plugin install vagrant-aws
```

*If any issues during installation, then try with installing dependencies. (Depends on the workstation machine you are using, packages and version may change)*
```
yum -y install gcc ruby-devel rubygems compass
```

#### Notes
For other providers, you can find and install respective plugins; eg:
```
vagrant plugin install vagrant-digitalocean 
vagrant plugin install vagrant-omnibus
```

### 1.3 Setup Provider Environment - AWS
3.1 Make sure you have a proper **security group** created in your VPC (under your AWS account) with SSH, HTTP/HTTPS allowed.
3.2 Make sure you have created a **keypair** for this purpose and key file (**.pem** format) has been kept at a secure location on your machine.
3.3 Get your **access credentials** from AWS console. ([Refer my AWI CLI installation article](https://www.techbeatly.com/2018/03/how-to-install-and-configure-aws-command-line-interface-cli.html/#how-to-get-aws-credentials)). Add the same in ```~/.aws/credentials``` file.

```
# aws configure --profile devops
AWS Access Key ID [None]: AKIAJVOHCIQ72EXAMPLE
AWS Secret Access Key [None]: 7l/j/hxXeEA77/7e+7ZvLLBQW9SxdcEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

Verify content in the files
```
# mkdir ~/.aws
# cd ~/.aws/
# cat credentials 
[default]
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

[devops]
aws_access_key_id=AKIAJVOHCIQ72EXAMPLE
aws_secret_access_key=7l/j/hxXeEA77/7e+7ZvLLBQW9SxdcEXAMPLEKEY

# cat config 
[default] 
region=us-west-2 output=json 
[devops] 
region=us-west-2 output=json
```

#### Box Image 
In normal case with VirtualBox or HyberV, we need to give proper box details to load the image (like a template or clone). But in this case we are using AWS AMI (Amazon Machine Images) and config.vm.box is just for a vagrant syntax purpose. 

You can either add a dummy box(``` vagrant box add aws-dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box ```) or just use any available box image, just like what I did in Vagrantfile.

(You can choose any box by searching [here](https://app.vagrantup.com/boxes/search?provider=aws) for working with VirtualBox, Hyper-V or Docker)

## Step 2 - Create our Virtual Machine - AWS Instance
Vagrant is managed inside a project directory (anywhere at your convenience, eg: your home dir) where we save Vagrantfile, other provisioning scripts etc (bash or ansible playbooks).

### 2.1 Vagrantfile
We have **Vagrantfile** where we specify what type of VM we are creating, what are the specifications needed etc. You may refer [Vagrantfile](Vagrantfile) in this project for reference. (Items are explained inside the file)

### 2.2 Provisioning 
Vagrant Provisioners will help to automatically install software, update configurations etc as part of the vagrant up process. You can use any available provisioning method as Vagrant will support most of the build and configuration management softwares. (eg: bash, ansible, puppet, chef etc). 
Refer [Provisioning doc](https://www.vagrantup.com/docs/provisioning/)

We have used **Ansible** as provisioner and created a playbook called [deploy-infra.yaml](deploy-infra.yaml) in which we have mentioned what are the configurations we need on the server (VM) once its created. All tasks in playbook are self explanatory but i am listing down them for reference.
1. Create the directory for storing our website (/webapp/main-site)
2. Install nginx server
3. Start nginx service
4. Copy nginx configuration ([static_site.cfg](static_site.cfg) to /etc/nginx/sites-available/static_site.cfg on VM)
5. Create symlink to activate the site (from /etc/nginx/sites-available to /etc/nginx/sites-enabled/)
6. Clone website from github to /webapp/main-site
7. Restart nginx to load configuratioins
8. Install ufw (firewall)
9. Start Firewall service
10. Setup ufw and enable for reboot
11. Enable ssh and http ports
12. Disallow password authentication
13. Disallow root SSH access
14. Collect Public Hostname/Url to access
15. Verify website access

And we have 2 handlers in playbook
1. Restart ssh
2. Show public url


#### When provisioning happens ?
- When we run first ```vagrant up``` provisioning is run after creating this instance. 
- When ```vagrant provision``` is used on a running environment.
- When ```vagrant reload --provision``` is called. (If you have a change in provision script, just edit the yaml file and run this.)

### 2.3 Let's create the VM
Now, we will switch to the Vagrant project directory (vagrant-web) and create the VM.
```
# cd vagrant-web
# vagrant up
```
Wait for vagrant to create instance and provision software/configurations using ansible.

### 2.4 Verify our instance
If all goes well, you will see success message as well as a **public hostname url** in this case. We can access the url from browser and verify the website. (We have already a check inside the playbook to verify url access)

Also you can access the instance using ssh as below.
```
# vagrant ssh
```

### 2.5 Stop or Delete VM
```
# vagrant destroy
```

### Troubleshooting
#### vagrant up hang at "==> default: Waiting for SSH to become available..."
This is due to wrong ssh configurations; you need to make sure
- Your .pem key has correct permission and ownership
- You have used correct Security Group (with ssh access) in Vagrantfile
- You have used correct keypair details in Vagrantfile
