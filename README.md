# Project

## Setting Up An Azure Cloud Network

* Create Resource Group
* Create Vnet
* Setup Network Security Group
* Create Jump Box VM
* Create VMs Web1 and Web 2
* Create Load Balancer

### Creating A Resource Group

1. On the Azure page search for *Resource groups* 
2. Once you are in the *Resource groups* directory choose **+ Add** on the top left or click the blue *Create resource group* button.
* Choose correct Azure subscriptioin
* Name your resource group
* Choose a region
3. Click *Review & Create*, check that your selections are correct
* Click *Create*

### Creating A Vnet

1. Go back to the Home page on Azure
2. Search for *Virtual Networks*
3. Within *Virtual Networks* choose **+ Add** on the left or click the blue *Create Virtual Network* button
* Choose correct Azure Subscription
* Choose the Resource Group made in previous step
* Choose same region as your resource group
* Leave IP addresses set to *default*
* In the Security tab set all to *disabled*
4. Click *Review & Create*, check that your selections are correct and have the same regions from previous step
* Click *Create*

### Setup A Network Security Group (Firewall)

1. Navigate back to the Azure Portal home screen, search for *Netwrok Security Groups*
2. Click **+ Add** on the top left
 * Choose correct Azure Subscription
 * Choose the *Resource Group* you made in step 1
 * Name your Security group
 * Choose same region
4. Click *Review & Create*, Check your selections and region
 * Click *Create*
  * Click *Go to Resource*
      * On left, click *Inbound Security Rules*
      * Click **+ Add**
      * Set Source to *IP Address* Got to [IP4.me] to get your IP and copy your IP
      * Paste your IP
      * Leave Source Port Range to default
      * Set Destination to *Virtual Network*
      * Set Service to *Custom*
      * Set Destination port ranges to *3389*
      * Leave Protocol and Action set to default
      * Set Priority to *500*
      * Name the rule *Allow RDP*
      * Set description to "Allow RDP from external IP over port 3389"
      * Click *Add*
 5. Navigate back to *Inbound Security Rules*
      * Click *+ Add*
      * Set Source to *Any*
      * Set Source port ranges to "*"
      * Set Destination to *Any*
      * Set Service to *Custom*
      * Set Destination port ranges to "*"
      * Protocol set to *Any*
      * Set Action to *Deny*
      * Set Priority to *4096*
      * Name *Defauly Deny All*
      * Description "Deny all inbound traffic"

## Create Jump Box VM

  1. Navigate to the Azure Portal Home page
  2. Search for *Virtual Machines*
  3. Select **+ Add** and choose Virtual Machine
      * Choose correct subscription
      * Choose the Resource you created
      * Name VM Jump-Box-Provisioner
       * Choose correct Region
       * Leave Availability 
       * Image should be set to Ubuntu 18.04
       * Set size to B1s
  4. Administrator account should be set as follows:
       * SSH Public Key
       * Open Gitbash on your machine
       * Run `cd ~/.ssh ls` If files are present you are set. If not run `ssh_keygen` do NOT set a password
       * Run `cat ~/.ssh/id_rsa.pub` and copy your key
       * Set your Username
       * Choose to Use and existing public key
       * Paste your Public key
       * Public inbound ports should be set to *Allow selected ports*
       *  Select inbound ports set to SSH 22
   * Networking tab
        * Choose the Virtual Network you created
        * Creat a New Public IP and make it static
        * Set NIC to advanced and choose the Security Group you created
        * Click *Review & Create* Check that your selctions are correct
        * Click *Create*

  5. Create 2 more VMs named Web1 and Web 2
      * Set the Resource and security group to what you created
      * Make sure the Region is the same as previous setups
      * Make sure to use the same user/admin name for all machines
      * Set your ssh public key to the same as your Jump Box machine
      * Choose the VM system set B1ms with 1 cpu and 2gb ram
      * Choose the same Availability set for your Web1 and Web2 VMs. Choose Create New on the first VM setup and name appropriately. Choose this set when you create Web2.
      * Do NOT choose a public IP for your Web 1 and Web 2 machines!
      * Leave all other options set to default

## Setting up Jump Box Administration

  1. Create an Inbound Rule to allow SSH connections from your IP
      * Select **+ Add* 
      * Set Source to IP Address
      * Paste you IP 
      * Set source port range to *Any*
      * Set Destination to *Virtual Network* or the Internal IP of your Jump Box Machine
      * Set Destination port ranges to 22
      * Set Protocol to Any or TCP
      * Set Action to Allow
      * Set Priority lower than 4096
      * Name your rule
      * Description Allow SSH from my IP
  2. Open GitBash on your machine
  3. Make sure you can SSH into your Jump Box using `ssh AdminName@JumpBoxIP`
  4. Make sure you have sudo access

## Docker Container SetUp

  1. After deploying your Jump Box and VMs with the correct configuration and inbound/outbound rules you will need to `SSH` into your Jump Box to deploy your Docker Container.
    
   * `ssh` into your Jump Box using `ssh username@JumpBoxIP`
   * First install Docker io by running `sudo apt update` and then `sudo apt install docker.io`
   * Verify `docker.io` is runnning `sudo systemctl status docker`
        *If not running use `sudo systemctl start docker` to start service*
   * Once docker has installed you will need to pull your container `sudo docker pull cyberxsecurity/ansible:latest`
   * Drop into root and launch the container `docker run -ti cyberxsecurity/ansible:latest bash`
    
   2. Set a rule to allow your Jump Box full access to your Vnet
     * Navigate to Security Group settings and to add rules allowing SSH connections from your IP address
      * Set source to IP Address and add the private IP of your Jump Box
      * Source port ranges set to Any
      * Destination set to VirtualNetwork
      * Destination port ranges set to 22
      * Set Protocol to Any
      * Set Action to allow traffic from your Jump Box
      * Priority should be set to a lower number than your Deny All rule
      * Name your rule 
      * Description - Allow SSH from Jump Box IP
      
      
  ## Provisioner Setup
  
   1. Connect to Ansible container and generate a new ssh key
   2. Run `docker run -it cyberxsecurity/ansible /bin/bash` to start and connect to container
   3. Run `ssh-keygen` to create a new SSH key
   4. Run `ls .ssh/` to view your keys
   5. Run `cat .ssh/id_rsa.pub` to display your key
   6. Copy your key and return to your Aure Portal and locate your VM detail page
   7. Reset your VMs password and use your container's new public key for the SSH user
     * Test your connection using ssh from your Jump Box Ansible container using the internal IP of your VM
   8. Locate the config and host file for Ansible and Add your internal IP to the Host file
     * Open with `nano /etc/ansible/hosts`
      * Uncomment the `[webservers]` header and add the IP under `[webservers]`
         * Add `ansible_python_interpreter=/usr/bin/python3` next to each IP
   9. Change the Ansible config file to use your administrator account for SSH connections
   10. Open file with `nano /etc/ansible/ansible.cfg`
   11. Find and uncomment `remote_user` and replace `root` with your admin username


## Ansible Playbook

  1. Connect to your Jump Box and then to your Ansible container
  2. Run `docker container list -a` to locate the name of your container
  3. Run `docker start [container_name]` to start your container
  4. Run `docker attach [container_name]` to attach to your container
  5. Next we will create a YAML playbook file to use for configuration
  6. Run `nano /etc/ansible/pentest.yml` to create your playbook

  *  Your YAML should look like this 
   
   
   `
    ---
    
    - name: Config Web VM with Docker
       hosts: webservers
       become: true
       tasks:
       - name: docker.io
         apt:
           force_apt_get: yes
           update_cache: yes
           name: docker.io
           state: present

       - name: Install pip3
         apt:
           force_apt_get: yes
           name: python3-pip
           state: present

       - name: Install Docker python module
         pip:
           name: docker
           state: present

       - name: download and launch a docker web container
         docker_container:
           name: dvwa
           image: cyberxsecurity/dvwa
           state: started
           published_ports: 80:80

     - name: Enable docker service
       systemd:
         name: docker
         enabled: yes`
         
   7. Save and the run your playbook with `ansible-playbook /etc/ansible/pentest.yml`
   8. If no errors occur check that the DVWA site is up
    * SSH into your VM from your Ansible container `ssh user@internalIP`
   9. Run `curl localhost/setup.php` to test the connection, you should get HTML from the DVWA container


## Setting up Your Load Balancer

 1. Navigate to *Load Balancer* on Azure portal
 2. Create a new Load Balancer
 3. Select your subscription
 4. Select correct Resource Group
 5. Name your Load Balancer
 6. Select the correct Region (make sure it matches previous Regions)
 7. Create New IP for Load Balancer and name it appropriately
 8. Choose *static* 
 9. Review & Create
 10. Add a health probe to make sure VMs are receiving traffic
 11. Create a back end pool and add Web 1 and Web 2

 * Create a load balancing rule to forward port 80 from the load balancer to your Vnet
  * Name your rule
  * Choose IPv4
  * Leave frontend IP alone
  * Protocol should be set to TCP
  * Port and Backend port should be set to 80
  * Select the Health probe made perviously
  * Sesion persistance should be set to Client IP and protocol
  * Idle timeout and Floating IP should be left at default

* Create new Security Group rule to allow port 800 traffic from internet to internal Vnet
  * Choose IP for source and add your IP
  * Source port ranges Any
  * Destination VirtualNetwork
  * Destination port ranges 80
  * Protocol Any
  * Actions Allow
  * Name security rule

* After adding new rules, go back and delete the Default Deny All rule
* Verify you can reach your DVWA App from the browser over the internet
* Open your browser and enter the Load Balancer public IP address and add `/setup.php`

## Setting up redundancy

1. Start up your VMs in Azure Portal
2. Make sure you can connect to your VMs
3. Stop one of your VMs and check to see if your DVWA is still accessible
4. If you can still connect your redudancy is working

  



## ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![image](https://user-images.githubusercontent.com/77866059/121909504-968eeb00-ccf3-11eb-8502-a79380bf1873.png)


These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the playbook file may be used to install only certain pieces of it, such as Filebeat.

 [Elk YML](https://github.com/Shulkie512/Project/blob/main/ElkInstall.txt)
 [Filebeat](https://github.com/Shulkie512/Project/blob/main/FilebeatPlaybook.txt)
 [Metricbeat](

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting unwanted access to the network.
- Load Balancers protect against DDoS attacks, allows for traffic to flow between more than one server to maintain uptime. The Jump Box allows you to securly access the network via SSH from a specified IP address to make changes to your environment.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the logs and system traffic.
- What does Filebeat watch for? Changes to log files or specified locations to collects log events and forwards them either to Elasticsearch/Logstash to be indexed.
- What does Metricbeat record? Metrics from the system and services running on the server.

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function  | IP Address | Operating System |
|----------|---------- |------------|------------------|
| Jump Box | Gateway   | 10.0.0.4   | Linux            |
| Web 1    | Webserver | 10.0.0.5   | Linux            |
| Web 2    | Webserver | 10.0.0.6   | Linux            |
| ELK      | Monitoring| 10.1.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
Admins public IP address

Machines within the network can only be accessed by the Jump Box.
- Which machine did you allow to access your ELK VM? What was its IP address? The created Jump Box machine via the Docker container 20.185.180.89

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible |  Allowed IP Addresses |
|----------|---------------------| ----------------------|
| Jump Box | Yes                 |  Admin Public IP      |
| Web 1    | No                  | 10.0.0.4  10.1.0.4    |
| Web 2    | No                  | 10.0.0.4  10.1.0.4    |
| ELK      | No                  | 10.0.0.4              |
### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- What is the main advantage of automating configuration with Ansible?
- Ansible allows you to quickly install, update, add web servers to your network by using the same playbooks.

The playbook implements the following tasks:
- _TODO: In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc._
- ...
- ...

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

!![image](https://user-images.githubusercontent.com/77866059/121912740-73196f80-ccf6-11eb-89f0-3c0830889c51.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- List the IP addresses of the machines you are monitoring:
- 10.0.0.5
- 10.0.0.6

We have installed the following Beats on these machines:
- Filebeats
- Metricbeats

These Beats allow us to collect the following information from each machine:
- _TODO: In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the _____ file to _____.
- Update the _____ file to include...
- Run the playbook, and navigate to ____ to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_
- _Which URL do you navigate to in order to check that the ELK server is running?

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
