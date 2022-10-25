## Redpoint Automated Machine Learning (AML): Deployment on Kubernetes
AML gives you the ability to use machine learning on your data without requiring data scientist-level expertise.

AML is packaged as Docker Container Images. These container images are archived to Redpoint Global's private docker repository. 

In this guide, we take a Step-by-Step deployment of Redpoint Automated Machine Learning (AML) on Kubernetes. The diagram below shows what a complete Deployment of AML would look like in Kubernetes.
![803174cf-6ab4-4c8a-998c-7b959ee6b31c](https://user-images.githubusercontent.com/42842390/197833963-45b725f0-947e-4884-914b-c6942d6d883d.png)

### Table of Contents
- [Prerequisites ](#prerequisites)
- [Minimum System Requirements ](#system-requirements)
- [Install Procedure ](#install-procedure)
- [Retrieve the AML URL Endpoints ](#retrieve-the-aml-url-endpoints)
- [Install AML License](#install-aml-license)
- [Installation Gotchas](#installation-gotchas)
- [Get AML Support](#get-aml-support)

### Minimum System Requirements

- MongoDB server
    - Ubuntu or Redhat Linux  
    - Version: 4.4   
    - 16 GB Memory or more
    - 100 GB or more free disk space.
    - SSD disks for best IO perfomance
     
- Azure CosmosDB
    - Server Version: 3.6
    - Throughput: 4000 RUs

- Kubernetes Cluster
    - Nodepool with 2-3 nodes for high availabilty
    - 8 vCPUs and 16 GB memory per node
    - 100 GB or more free disk space per node
    
### Prerequisites

Before you install AML, you must:

1. Install kubectl. ( https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/ )
3. Have a Kubernetes solution available to use. ( https://kubernetes.io/docs/setup/production-environment/turnkey-solutions/ )
4. Have an Ingress Controller solution deployed in your Kubernetes cluster (https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
5. Have a MongoDB database server available to use for the AML application Databases.
6. Clone this repository ( git clone https://github.com/RedPointGlobal/rp-mdm.git ) 
7. If you dont have a docker ID, create one at https://hub.docker.com/ and provide Redpoint Support with your account ID so they can grant you permissions to pull the AML container images
8. Have a license key to activate AML. Contact Redpoint support for an activation key
