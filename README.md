# Vault Infrastructure creation
![tools](https://github.com/LauJorge/API-POC/blob/master/lauAPI/vt.png?raw=true)

# Work flow


Openshift consumes a Build Config file, stored on a Bitbucket repository, so as to create a dynamic slave with Terraform. Once the binaries are built and available on Artifectory, Jenkins can consumed them and run the Terraform commands to provide the Vault's HA infrastructure on AWS.

  
  

# Bitbucket

  
Bitbuckets is used to store repositories containing Terraform source code to create resources on AWS, a repository with Packer source code to create AMIs, and repositories with build config files to create the necessary binaries.

  

#### Implementation
The listed necessary repositories are found in Bitbuckets:
  
for AMI creation
-  [https://bitbucket.mfoundry.net/users/maria.jorge/repos/ami_creation](https://bitbucket.mfoundry.net/users/maria.jorge/repos/ami_creation/browse)

  
for Vault Infrastructure creation
-  [https://bitbucket.mfoundry.net/users/maria.jorge/repos/vault-infra](https://bitbucket.mfoundry.net/users/maria.jorge/repos/vault-infra/browse)

  
for Terraform and Packer configuration files 
-  [https://bitbucket.mfoundry.net/projects/D1PIPE/repos/openshift-deployment/browse/cicd/builders](https://bitbucket.mfoundry.net/projects/D1PIPE/repos/openshift-deployment/browse/cicd/builders)

  

# OpenShift


Openshift builds an image of a dynamic Jenkins' slave with Terraform and another with Packer.

  

#### Implementation

  
Steps to build:
  

1. Clone the repository
```
git clone https://username@bitbucket.mfoundry.net/scm/d1pipe/openshift-deployment.git
```
2. Install OCP

3. Login OCP

```
oc login
```
4. Select the CICD repo which is where the slave builds are being run.
```
oc project cicd
```
5. In the same directory as the yml file run the apply option if the build config is new.
```
oc apply -f <yml file> -n cicd
```
From the CLI the start-build operation can be used.

```
oc start-build <name> -n cicd
```

Once the new build config runs successfully it will be uploaded into **Artifactory** and available.
  

-  [https://ocpmaster4.fismobile.net/console/project/cicd/browse/pods/jenkins-agent-maven-35-centos7-dsm-api-2-build](https://ocpmaster4.fismobile.net/console/project/cicd/browse/pods/jenkins-agent-maven-35-centos7-dsm-api-2-build?tab=details)

  

-  [https://ocpmaster4.fismobile.net/console/project/cicd/browse/pods/jenkins-agent-maven-35-centos7-packer-1-build](https://ocpmaster4.fismobile.net/console/project/cicd/browse/pods/jenkins-agent-maven-35-centos7-packer-1-build?tab=details)

  
  
  

# Artifactory

  

Stores the neccessary binaries to build and run a dynamic Jenkins' slave with Terraform and another agent with Packer.

  

  
#### Implementation

  
The following binaries can be found on Artifactory:
-  [https://repository.fismobile.com/artifactory/webapp/#/artifacts/browse/tree/General/mobile-docker-1/jenkinsimages/jenkins-agent-terraform/v1.0.0](https://repository.fismobile.com/artifactory/webapp/#/artifacts/browse/tree/General/mobile-docker-1/jenkinsimages/jenkins-agent-terraform/v1.0.0)

  


-  [https://repository.fismobile.com/artifactory/webapp/#/artifacts/browse/tree/General/mobile-docker-1/jenkinsimages/jenkins-agent-packer/v1.0.0](https://repository.fismobile.com/artifactory/webapp/#/artifacts/browse/tree/General/mobile-docker-1/jenkinsimages/jenkins-agent-packer/v1.0.0)

  

# Jenkins
A pipeline project runs the necessary commands to have the HA Vault infrastructure.

#### Implementation
1. **Create S3 bucket:** provides a backend for the cluster (VPC and certificates too) that will store the Terraform states. Job: [***S3_creation***](https://sage.fismobile.net/view/ITO-DevOps/job/Playpen/job/lau/job/s3_creation)

  

2. **Create a VPC:** *--not actually needed--* provides VPC, subnets and necessary resources where the infrastructure is placed. Job: [***vpc_creation***](https://sage.fismobile.net/view/ITO-DevOps/job/Playpen/job/lau/job/vpc_creation)

  

3. **Create certificates**: provides pem keys and certificates required to create an AMI. Job: [***certificates_creation***](https://sage.fismobile.net/view/ITO-DevOps/job/Playpen/job/lau/job/certificates_destroy)

  

4. **Create the AMI**: this image has Vault and Consul installed and configured. It's created with Packer. It requires a VPC and Subnet, as well as the certificates location. Job: [**AMI_creation**](https://sage.fismobile.net/view/ITO-DevOps/job/Playpen/job/lau/job/AMI_creation)

  

5. **Create a cluster of Vault and Consul virtual machines on AWS**: provisioing of 3 Vault and 3 Consul EC2 and necessary resources for a HA Vault infrastructure. It requires the AMI id, VPC and subnets and S3 . Job: [***vault_infra***](https://sage.fismobile.net/view/ITO-DevOps/job/Playpen/job/lau/job/vault_infra)

  

6. **Clean up**: all jobs have each destroy job, and there is a general destruction job for the entire infrastructure: Job: [***vault_destroy***](https://sage.fismobile.net/view/ITO-DevOps/job/Playpen/job/lau/job/vault-destroy)
  
  

# Terraform

Terraform provisions the HA Vault infrastructure with all its required resources.

#### Implementation

  
When running the Jenkins jobs, Terraform is used to create the infrastructure. If wanted to be tested locally, the following steps are required:  

1. Clone the repository with the Terraform source code
```
git clone https://bitbucket.mfoundry.net/users/maria.jorge/repos/vault-infra.git
```
2. Initialize Terraform
```
terraform init
```
3. Create an execution plan to determine what actions are necessary to achieve the desired state specified in the configuration files
```
terraform plan
```
4. Apply the changes required to reach the desired state of the configuration
```
terraform apply
```

  

# AWS

  

Amazon Web Services provides on-demand cloud computing platforms.

  

#### Usages

  

AWS will host the infrastructure.

  

#### Implementation
An autoscaling group with an IAM instance profile and security group rules are created on AWS to have the HA Vault infrastructure

![tools](https://github.com/LauJorge/API-POC/blob/master/lauAPI/HA.png?raw=true)

When running in HA mode, Vault servers have two additional states: standby and active.
 Within a Vault cluster, only a single instance will be active and handles all requests (reads and writes) 
 and all standby nodes redirect requests to the active node.


# References
- https://github.com/hashicorp/terraform-aws-vault

- https://learn.hashicorp.com/vault/day-one/ops-reference-architecture