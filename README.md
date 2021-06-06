Infrastructure Coding Test
==========================

# Goal

Script the creation of a web server, and a script to check the server is up.

# Prerequisites

You will need an AWS or GCP account. Create one if you don't own one already. You can use free-tier resources for this test. You need to provide a credit card in both cases, but you won't be charged. 

# The Task

You are required to set up a new server in AWS/GCP. It must:

* Be publicly accessible.
* Run Nginx or Apache.
* Serve a `/version.txt` file, containing only static text representing a version number, for example:

```
1.0.6
```

# Mandatory Work

Fork this repository.

* Provide instructions on how to create the server.
* Provide a script that can be run periodically (and externally) to check if the server is up and serving the expected version number. Use your scripting language of choice.
* Alter the README to contain the steps required to:
  * Create the server.
  * Run the checker script.
* Provide us IAM credentials to login to the AWS/GCP account. If you have other resources in it make sure we can only access what is related to this test.

Give our account access to your fork, and send us an email when you’re done. Feel free to ask questions if anything is unclear, confusing, or just plain missing.

# Extra Credit

We know time is precious, we won't mark you down for not doing the extra credits, but if you want to give them a go...

* Use a Terraform template to set up the server.
* Use a configuration management tool (such as Puppet, Chef or Ansible) to bootstrap the server.
* Put the server behind a load balancer.
* Run Nginx/Apache2 inside a Docker container.
* Make the checker script SSH into the instance, check if Nginx is running and start it if it isn't.

# Questions

#### What scripting languages can I use?

Anyone you like. You’ll have to justify your decision. We use Bash, Python and JavaScript internally. Please pick something you're familiar with, as you'll need to be able to discuss it.

#### Will I have to pay for the AWS charges?

No. You are expected to use free-tier resources only and not generate any charges. Please remember to delete your resources once the review process is over so you are not charged by AWS.

#### What will you be grading me on?

Scripting skills, ellegance, understanding of the technologies you use, security, documentation.

#### Will I have a chance to explain my choices?

Feel free to comment your code, or put explanations in a pull request within the repo.
If we proceed to a phone interview, we’ll be asking questions about why you made the choices you made.

#### Why doesn't the test include X?

Good question. Feel free to tell us how to make the test better. Or, you know, fork it and improve it!

# Answers

### 1.  Setup the LAB Environment
In this task, we will use:

-GCP to deploy the infrastructure. 

-Terraform as an infra as a code tool to automate the infrastrucure setup.

-ansible to automate configurations.

### 2. Create a new GCP Project and setup IAM credentials 

-From GCP console, create a new project "Emnos Coding Test".

-Enable compute Engine API for this project.

-Generate the IAM credentials to be used by terraform.

==> Go to Emnos Coding Test project => IAM => Service Accounts.

==> Click on "create service Account.

![image](https://user-images.githubusercontent.com/85032988/120905367-78692100-c649-11eb-8b3c-816a3ac4c989.png)

==> Choose a name for the service account "terraform".

==> Grant this service account access to the project as owner ==> this account should be able to deploy resources.

==> Add compute admin/ Organization Administrator and Storage Admin permissions to the service account.

![image](https://user-images.githubusercontent.com/85032988/120906445-51aee880-c651-11eb-9f10-4ca87866fd30.png)


![image](https://user-images.githubusercontent.com/85032988/120905450-01805800-c64a-11eb-9789-bf9a2f7f7121.png)

- Generate Keys : Terraform will use this key to authenticate to the project.

==> Select the terraform account, go to Actions => Manage keys.

==>Choose add key ==> Create new key and choose JSON type.

![image](https://user-images.githubusercontent.com/85032988/120905551-bc105a80-c64a-11eb-9fb6-7ef865d7f73d.png)

==> This json key must be saved in the same location as the terraform files.

### 3. Deploy nginx server with terraform

- create a main.tf file: 

```
//First, we need to specify the cloud provider ; here we will use GCP
provider "google" {
    credentials = file ("emnos-test-new.json")  // credentials are generated from GCP Console 
    project = "emnos-coding-test" //Project ID
    region = "us-central1" 
    zone = "us-central1-a"

}

 //Create a static public IP address for our google compute instance to utilize 
 //this IP will not be release even after VM instance reboot
resource "google_compute_address" "static" {
  name = "vm-public-address" 
  project = "emnos-coding-test"
  region = "us-central1"
    
   }

    

//deploy the nginx server 
resource "google_compute_instance" "nginx"{
    provider = google
    name = "nginx-server"
    machine_type = "f1-micro"
    zone = "us-central1-a"
    tags = ["nginx-node"]

boot_disk {
    initialize_params{
      image = "gce-uefi-images/centos-7" //choose the OS image to use 
      }
    }
    //give the startup command to be executed once the instance is created
    //update the server and then install nginx

    metadata_startup_script = "sudo yum -y update; sudo yum -y install epel-release; sudo yum -y install nginx; sudo service nginx start;"
   

    network_interface {
      
      network = "default"
      access_config {
        nat_ip = google_compute_address.static.address
      }
   
    }
    
```
==> This script will be run with powershell because terraform is installed on a windows 10 machine.

==> From powershell run these commands in order:

- terraform.exe init ==> initialize the working directory and used plugins

![image](https://user-images.githubusercontent.com/85032988/120906645-12819700-c653-11eb-9d22-c937364d3925.png)


- terraform.exe plan ==> create the execution plan

![image](https://user-images.githubusercontent.com/85032988/120906683-5bd1e680-c653-11eb-8ca8-2992480b9011.png)

- terraform.exe apply ==> Apply the changes (below the output of this command)
```
...
  Enter a value: yes

google_compute_address.static: Creating...
google_compute_address.static: Still creating... [10s elapsed]
google_compute_address.static: Creation complete after 14s [id=projects/emnos-coding-test/regions/us-central1/addresses/vm-public-address]
google_compute_instance.nginx: Creating...
google_compute_instance.nginx: Still creating... [10s elapsed]
google_compute_instance.nginx: Creation complete after 16s [id=projects/emnos-coding-test/zones/us-central1-a/instances/nginx-server]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

```
-In order to be able to externally access the nginx server, we will add a firewall resource
```
//create a firewall rule and allow ports 80 http, 443 https and 22 ssh in order to access the web server from public network

resource "google_compute_firewall" "default" {
  name    = "nginx-firewall"
  network = "default"

allow {
  protocol = "tcp"
  ports    = ["80","443","22"]
}
}
```
- terraform plan output:




-From GCP console we can check that the deployement is done successfully;

==> VM instance is created:

![image](https://user-images.githubusercontent.com/85032988/120907170-4a8ad900-c657-11eb-8afa-810520081a02.png)

==> Static IP is created:

![image](https://user-images.githubusercontent.com/85032988/120907203-92116500-c657-11eb-9aa5-fd85a120be91.png)

==> SSH into the VM instance and check if nginx server is created and up

![image](https://user-images.githubusercontent.com/85032988/120907237-d13fb600-c657-11eb-8cb8-894bd6bba27d.png)

### 3. Configure nginx to serve /version.txt containing a static text (server version)

- Check nginx version:

![image](https://user-images.githubusercontent.com/85032988/120907385-c33e6500-c658-11eb-84db-101a8d03fe05.png)

- backup config file /etc/nginx/nginx.conf : 

```
sudo cp nginx.conf nginx.conf.bak
```
- Create a new directory `version` under `/usr/share/nginx` and create a new txt file `version.txt`

```
[khammarii_mariem@nginx-server cron.d]$  cat /usr/share/nginx/version/version.txt 
1.16.1
```
-grant nginx user execution permissionson version directory and version.txt file
```
sudo chown nginx:nginx version
sudo chmod  +x version
sudo chown nginx:nginx version.txt 
sudo chmod +x  version.txt

```
- Edit `/etc/nginx/nginx.conf` file and restart nginx service:

```
...
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/version; 

```
- Go to the browser and ckeck `http://34.121.249.155/version.txt`

![image](https://user-images.githubusercontent.com/85032988/120907697-95a6eb00-c65b-11eb-90a5-cde583ced68e.png)


### 4. Checker Script:

## Create the script : 

- we will use a shell script: the script will check the status of nginx server; if it is up it will return 
'nginx server is up' else it will restart the service.

```
#!/bin/bash
 if  /usr/sbin/nginx -t 2>/dev/null; then
       echo 'nginx server is up'
      else
        service nginx restart
      fi

```

## Run the script remotely from terraform

- First, we should generate public key and private key to run the remote exec command ==> Terraform will ssh to the nginx server and run the `check` script.
- The key generation is done by putty key generator

![image](https://user-images.githubusercontent.com/85032988/120908696-b758a000-c664-11eb-8f98-d1792fb3c669.png)

```
 metadata ={
    ssh-keys = "${file("emnos-task.pub")}" //public_key_path
  }
provisioner "remote-exec" {
    connection {
      host        = "nginx-server"
      type        = "ssh"
      user        = "root"
      private_key = "${file("emnos-task.ppk")}" //private_key_path
      agent       = false
    }

    inline = [
      "sudo ./check "
    ]
   
```
## Automate the execution of the checker script with ansible

# Deploy ansible:

- We will use terraform to deploy a new vm instance and install ansnible on it

```
 // create a public IP address for our google compute instance to utilize
 //the new instance is crceated in "emnos-coding-test" project
resource "google_compute_address" "ansible_address" {
  name = "ansible-public-address"
  project = "emnos-coding-test"
  region = "us-central1"
    
   }

//Create ansible control node
resource "google_compute_instance" "ansible"{
    provider = google
    project = "emnos-coding-test"
    name = "ansible-control-node"
    machine_type = "f1-micro"
    zone = "us-central1-a"
    tags = ["ansible-node"]

boot_disk {
    initialize_params{
      image = "gce-uefi-images/centos-7"
      }
    }
    //give the startup command which need to execute after created an instance
//
    metadata_startup_script = " sudo yum -y update; sudo yum -y install epel-release; sudo yum -y install epel-repo ; sudo yum -y install ansible"
   

//configure a static IP adress for the instance
network_interface {
      
   network = "default"
     access_config {
        nat_ip = google_compute_address.ansible_address.address
      }
   
    }
}
```
-From GCP console, we can check the deployement

![image](https://user-images.githubusercontent.com/85032988/120908707-e111c700-c664-11eb-83aa-ff76b3348e81.png)

![image](https://user-images.githubusercontent.com/85032988/120908708-e8d16b80-c664-11eb-8a65-4a7ae25bb426.png)

-SSH to ansible node and verify ansible is installed

![image](https://user-images.githubusercontent.com/85032988/120908710-ee2eb600-c664-11eb-80dd-896947d45e6d.png)


# Create the ckecker playbook 

- Create a user `ansible` to execute the playbook and authenticate to the remote nginx server
- Create the playbook `Checkplaybook.yaml` 

```
---
- name: check nginx is up
  hosts: all
  tasks:
   - name: nginx is up
     command: sudo sh /home/ansible/checkscript.sh

   - name: nginx version
     uri:
      url: http://34.121.249.155/version.txt

   - name:  display nginx version
     uri:
      url: http://34.121.249.155/version.txt
      return_content: yes
      method: GET
     register: _content
   - name: debug
     debug:
      var: _content.content

```

==> This playbook executes two tasks : 
- check ngnix is up else restart it by executing the shell script (`checkscript.sh` is a copy of `check` script in the home directory of ansible user so that it can execute it)
- display nginx version by returning the content of nginx url 


## make the script run periodically

- We will make the ckecker script run every 20min ==> This task will create a file in /etc/cron.d/check-script
- Create a new playbook `cron.yaml`


```
---

- name: check nginx is up with cron
  hosts: all
  tasks:

   - name: run check script
     cron: minute="20"
      name="run check script (every 20min)"
       cron_file="check-script"
       user="ansible"
       job="/home/ansible/checkscript.sh"

```
- Execute playbook:

```
[khammarii_mariem@ansible-control-node emnostask]$ ansible-playbook cron.yaml 

....

PLAYBOOK: cron.yaml ***********************************************************************************************
1 plays in cron.yaml

PLAY [check nginx is up with cron] ********************************************************************************

TASK [Gathering Facts] ********************************************************************************************
task path: /home/khammarii_mariem/emnostask/cron.yaml:3
ok: [nginx]
META: ran handlers

TASK [run check script] *******************************************************************************************
task path: /home/khammarii_mariem/emnostask/cron.yaml:7
changed: [nginx] => {"changed": true, "cron_file": "check-script", "envs": [], "jobs": ["run check script (every 20min)"]}
META: ran handlers
META: ran handlers

PLAY RECAP ********************************************************************************************************
nginx                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
- Go to the nginx server and grant permissions to ansible user on `/etc/cron.d`
``
sudo chown ansible cron.d
``

- To ckeck : connect with ansible user 

![image](https://user-images.githubusercontent.com/85032988/120908676-85dfd480-c664-11eb-95eb-8044d233b567.png)

![image](https://user-images.githubusercontent.com/85032988/120908681-8d9f7900-c664-11eb-9a90-eb09ce69e1ef.png)





