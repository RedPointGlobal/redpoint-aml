## Redpoint Automated Machine Learning (AML): Deployment on Kubernetes
AML gives you the ability to use machine learning on your data without requiring data scientist-level expertise.

AML is packaged as Docker Container Images. These container images are archived to Redpoint Global's private docker repository. 

In this guide, we take a Step-by-Step deployment of Redpoint Automated Machine Learning (AML) on Kubernetes. The diagram below shows what a complete Deployment of AML would look like in Kubernetes.
![803174cf-6ab4-4c8a-998c-7b959ee6b31c](https://user-images.githubusercontent.com/42842390/197833963-45b725f0-947e-4884-914b-c6942d6d883d.png)

### Table of Contents
- [Prerequisites ](#prerequisites)
- [System Requirements ](#system-requirements)
- [Install Procedure ](#install-procedure)
- [Retrieve the AML URL Endpoints ](#retrieve-the-aml-url-endpoints)
- [Install AML License](#install-aml-license)
- [Installation Gotchas](#installation-gotchas)
- [Get AML Support](#get-aml-support)

### System Requirements

- MongoDB server sizing
    - 4 GB memory for data < 1 million records total: 
    - 8-16 GB memory for data 1-10 million records total
    - SSD disks for best IO perfomance
    - Size the MongoDB memory to keep working index set in memory for data > 10 million records total

- Kubernetes Cluster sizing
    - nodepool with 2-3 nodes for high availabilty
    - 8 vCPUs and 32 GB memory per node
    - 20 GB or more free disk space per node

### Prerequisites

Before you install MDM, you must:

1. Install kubectl. ( https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/ )
2. Install HELM. ( https://helm.sh/docs/intro/install/ )
3. Have a Kubernetes solution available to use. ( https://kubernetes.io/docs/setup/production-environment/turnkey-solutions/ )
4. Have an Ingress Controller solution deployed in your Kubernetes cluster (https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
5. Have a MongoDB database server available to use for the MDM application Databases. ( https://docs.mongodb.com/v4.4/installation/). This can also be a managed solution like mongo Atlas. (As of this writing, MDM has been tested on MongoDB version 4.4)
6. Clone this repository ( git clone https://github.com/RedPointGlobal/rp-mdm.git ) 
7. If you dont have a docker ID, create one at https://hub.docker.com/ and provide Redpoint Support with your account ID so they can grant you permissions to pull the MDM container images
8. Have a license key to activate a Trial for Production License. Contact Redpoint support for an activation key
