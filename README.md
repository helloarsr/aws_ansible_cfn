
Pre-requisite before start the Project: 
--------------------------------------------------

1)First Create AWS free account to comlete whole project 

2)create a keypair on AWS console and convert *.pem file using putygen to private key then save it,which needs to be used to access EC2 Instances

3)Launch ansible control server and manage all required roles,playbook from this

4)Installation of git

5)Create a Github account to maintain this project documentaion and placing the project repository

# aws_ansible_templates:
----------------------------------------------------
aws and ansible automated application(Web,App and DB) installation projects

The project is about opencms application installation on AWS cloud IaaS as ansible and cloudformation templates with installing Web,App and Database architecture. So, its mainly focusing on cloudformation and asible to create the stack templates automataion work. The complete automation project is mantained through cloudformation templates and ansible configuration roles to install and configure the Opencms application deployment on tomcat server.

Setup Opencms on your AWS with Ansible and Cloudformation, you can install it from your laptop by making itself a Workstation
If you are fluent with Ansible and Cloudformation, you know where to look and what to do. If you are new to Ansible and cloudformation, just follow along with step-by-step instructions below.


Index:
-----------------------------------------------------
Create a cloudformation templates:

1-VPC network infrastrucure 

2-WEB01 EC2 instance 

3-APP01 EC2 instance

4-DB01 EC2 instance

5-Ansible Control instance
  
6)Ansible Roles 

  a)Opencms -Istall and configure  Opencms app on App instance deploying on tomcat server
  b)webserver -Install and configure MySQL on DB instance.
  c)tomcat -Install and configure Tomcat on App instance
  d)mysqldb - install and configure MySQL on DB instance.

7)Route 53 :
  a)create a recode set for www with EIP of WEB01 (this should be configured as per DNS redirecting URL on freenom.com )
  b)accessing the app through domain name www.opencmsapp.tk

