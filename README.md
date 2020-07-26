# Notes of GPTE Ansible Advanced Homework  
  
## PoC requirements:  

An Ansible playbook project on Ansible Tower to deploy the three tier application to QA and PROD environments.  
QA environment is based on Openstack which is behind the firewall, so all operations need to run on a workstation.  
The instances for deploying application will create on demand.
PROD environment is on AWS, it is on demand also.  
Three tier application includes load balancer, web container and database. Deploying three tier application to QA and PROD environment will use one module.  
Deployment sequence are following:  
1. Preparing QA & PROD environment concurrently.  
2. Deploying 3 tier application on QA environment, and testing. If test failed then delete QA environment instances.  
3. On test success and PROD environment ready, deploying 3 tier application on PROD environment, and doing the final test.  


### Deployment Architecture
![Diagram of PoC deployment architecture](/images/deployment-arch.png)  


### Guidance
Design and implements guidance are following:  
* Use a clear and consistent style  
* Make sure that plays and tasks are self-documenting, with clear and meaningful names  
* Make sure that the repositories are fully self-contained  
* MitziCom is adopting an Infrastructure as Code policy going forward  
* Use roles extensively to achieve a high degree of reuse and modularity  
* Make sure that optimal modules are used throughout  
* Use templates throughout, making sure they are clearly marked as "Ansible generated"  
* Use handlers where appropriate  
* Use Ansible Vault to protect sensitive information  


## Ansible workflow job template
Following the requirements, the Ansible workflow job is seperate to several jobs.  
![Diagram of workflow job template](/images/workflow-job-template.png)  

Job template | Description
------------|------------
00_Homework Assignment | Update project from GITHUB
01_Provision Prod Env | Preparing PROD environment on AWS
02_Provision QA Env | Preparing QA environment on Openstack, running on workstation
03_06_Prod Three tier inventory source | Getting PROD environment hosts & groups from AWS
04_3tier app deployment on QA Env | Deploying 3 tier application on QA environment
05_Smoke test QA Env | Testing 3 tier application on QA environment
07_Prod Check the status of AWS instances | Checking the status of PROD instances on AWS
08_Prod SSH keys three tier app | Getting user access key from bastion host of PROD environment, for deploying
09_3 tier app on Prod | Deploying 3 tier application on PROD environment
10_Smoke test Prod env | Testing 3 tier application on PROD environment
Nuke QA Env | Cleanup instances of QA env 

## Ansible playbooks & roles
Please review the codes from https://github.com/lees07/nextgen_ansible_advanced_homework.git

Playbook or role | Description
------------|------------
roles/app-tier | Install application server role
roles/db-tier  | Install postgressql server for database role
roles/lb-tier  | Install HA proxy role
roles/base-config | Setup yum repo and base packages role for all tier
roles/setup-workstation | Setup workstation, create network, ssh keypair, security group etc. role, for QA only 
roles/osp-servers | Provision OSP Instances role, for QA only
roles/osp-instance-delete | Delete OSP Instances role, for QA only
roles/osp-facts | Genrate in-memory inventory for OSP instances role, for QA only
roles/config-tower | Role to configure ansible tower job templates and workflow
roles/config-tower/vars/main.yml | All the variable values for ansible tower configuration role
roles/config-tower/tasks/ec2_dynamic.yml | For creating Dynamic inventory in Ansible tower. Use `AWS Access Key` for credential
roles/config-tower/tasks/job_template.yml | For creating job templates
roles/config-tower/tasks/pre-config-tower.yml | Any pre config tasks needed
roles/config-tower/tasks/workflow_template.yml | genrate workflow from `workflow.yml` file
roles/config-tower/tasks/post-config-tower.yml | any post config jobs
aws_creds.yml | Fetch GUIDkey.pem from bastion of Three tier application env and create machine credential to connect to AWS instances
aws_provision.yml | Use `order_svc.sh` script to provision env
aws_status_check.yml | Check aws instances are up or not
site-3tier-app.yml | Playbook to deploy three tier app
site-install-isolated-node.yml | Playbook to install isolated node
site-config-tower.yml | Playbook to call role `config-tower`
site-osp-delete.yml | Playbook to call role
site-osp-instances.yml | Playbook to call role
site-setup-prereqs.yml | Playbook to call role
site-smoke-osp.yml | Playbook to test three tier app on OSP
site-smoketest-aws.yml | Playbook to test three tier app on AWS
grading-script.yml | Self grading script

![Result of workflow job template](/images/result.png)  

## Troubleshooting issues
1. Variable "{{ opentlc_login }}" does not set correctly in config-tower role. Fixed.  
2. Single Quotation Marks need to use when special character exist in username or password in aws_provision.yml, like:  
---
    - name: create credentials
      copy:
        content: |
          export username='{{ param_user }}'
          export password='{{ param_password }}'
          export uri=https://labs.opentlc.com
        dest: /root/credential.rc
        mode: 0755
---



## Some document errors   
Sector 4.3.5, the internal repo with 192.168.0.5 can't be accessed.  
---
5. Configure your instance to use the organization’s local yum repository, 
http://www.opentlc.com/download/ansible_bootcamp/repo/internal.repo  
---

Sector 4.4.2, the item name in command will be "Ansible Advanced - Three Tier App"  
---
source credential.rc ; ./order_svc.sh -y -c 'OPENTLC Automation' -i 'Ansible Advanced' -t 1 -d 'dialog_expiration=7;region=na;nodes=1;dialog_runtime=8'  
---
