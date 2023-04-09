![RG](https://user-images.githubusercontent.com/42842390/158004336-60f07c05-7e5d-420e-87a6-22c5ac206fb6.jpg)
## Redpoint Automated Machine Learning (AML) - Deployment on Kubernetes
AML gives you the ability to use machine learning on your data without requiring data scientist-level expertise. 

In this guide, we take a Step-by-Step deployment of Redpoint Automated Machine Learning (AML) on Kubernetes using HELM. 

The diagram below shows the deployment architecture
![803174cf-6ab4-4c8a-998c-7b959ee6b31c](https://user-images.githubusercontent.com/42842390/197833963-45b725f0-947e-4884-914b-c6942d6d883d.png)

### Table of Contents
- [Prerequisites ](#prerequisites)
- [System Requirements ](#system-requirements)
- [Install Procedure ](#install-procedure)
- [Ingress ](#ingress)
- [AML URL Endpoints ](#aml-url-endpoints)
- [Install License](#install-license)
- [AML Documentation](#aml-documentation)
- [Support](#support)

### System Requirements

- MongoDB Database 
    - Version: 4.4 or later
    - 16 GB Memory or more
    - 100 GB or more free disk space.
    - SSD disks for best IO perfomance
    - The MongoDB database can be any of the following
       - Azure Cosmos DB
       - MongoDB on Linux
       - MongoDB Atlas

- Kubernetes Cluster
    - Latest stable version of Kubernetes
    - Nodepool with 2-3 nodes for high availabilty
    - 8 vCPUs and 16 GB memory per node
    - 100 GB or more free disk space per node
    - The Kubernetes cluster can be any of the following
       - Self hosted Kubernetes
       - Managed Kubernetes (AKS, EKS, GKE)
    
### Prerequisites

Before you install AML, you must:

1. Have a Kubernetes solution available to use. ( https://kubernetes.io/docs/setup/production-environment/turnkey-solutions/ )
2. Install kubectl. ( https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/ )
3. Have a MongoDB server available to use for AML databases
4. Docker ID, create one at https://hub.docker.com/ and provide the account ID to Redpoint Support so they can grant you permissions to pull the AML container images
5. Have a license key to activate AML. Contact Redpoint support for an activation key

### Install Procedure

1. Clone this repository
```sh
git clone https://github.com/RedPointGlobal/redpoint-aml.git
 ```
2. Create a namespace for AML
```sh
kubectl create namespace redpoint-aml

 ```
3. Create the following secrets that aml needs
 - ```mongo-conn-string``` | Secret that contains your mongodb connection string 
```
 kubectl create secret generic mongo-conn-string \
--from-literal=MONGODB_NAME=$your_mongo_connection_string \
--namespace redpoint-aml
```
 - ```docker-io``` | Secret that contains your docker hub credentials 
```
kubectl create secret docker-registry docker-io --docker-server='https://index.docker.io/v1/' \
--docker-username=$your_docker_username --docker-password=$your_docker_password --docker-email=$your_docker_email --namespace redpoint-aml
```
 - ```aml-tls``` | Secret that contains your TLS certificate files (.crt and .key)
```
kubectl create secret tls aml-tls --cert=$your_tls_cert --key=$your_tls_key --namespace redpoint-aml
```
### Install
Make sure you are in the repo directory that you cloned in step 1 and then run the following command to insall aml
```sh
    helm install redpoint-aml redpoint-aml/ --values values.yaml --namespace redpoint-aml
 ```
It may take a few minutes for the all the AML services to start. Please wait about 10 minutes before testing.

### Retrieve the AML URL endpoints
The chart creates an NGINX ingress controller for you and configures all the ingress rules necesssary for accessing AML endpoints externally. It deploys a public Load Balancer by default.

If you prefer an internal load balancer, simply edit the ```nginx-redpoint-aml``` load balancer service in ```redpoint-aml/templates/nginx-deploy.yaml``` file and add the annotations below depending on the cloud provider
```
AZURE (AKS)
service.beta.kubernetes.io/azure-load-balancer-internal: "true"
service.beta.kubernetes.io/azure-load-balancer-internal-subnet: "<add your subnet name>"

AWS (EKS)
service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
service.beta.kubernetes.io/aws-load-balancer-internal: "true"    
service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: 'true'    
service.beta.kubernetes.io/aws-load-balancer-type: nlb    
service.beta.kubernetes.io/aws-load-balancer-connection-idle-timeout: "60"

GCP (GKE)
cloud.google.com/load-balancer-type: "Internal"
```
Once that is done, upgrade the helm deployment like so ```helm upgrade redpoint-aml redpoint-aml/ --values values.yaml```
### AML URL endpoints
You should now be able to extract the AML URL endpoints by running the commands below 
```sh
    kubectl get ingress
 ```
- The command returns the following URL endpoint
```
https://redpoint-aml.example.com

where;
 - https://redpoint-aml.example.com             # AML login UI
 - https://redpoint-aml.example.com/admin/      # AML admin UI
 - https://redpoint-aml.example.com/docs/       # AML swagger docs UI
 ```  

### Install License
Once you obtain your activation key from Redpoint Support, access the AML admin UI and enter the license.
```
Default username: admin@noemail.com
Default password: .RedPoint123
```
![image](https://user-images.githubusercontent.com/42842390/218563945-94a5b162-dc59-45ae-900e-130a84810f66.png)

### AML Documentation
For detailed AML documentation and release notes, please visit the official AML documentation website at the address below

https://mercury-docs.redpointcloud.com

### Support 
Contact support@redpointglobal.com for any application specific issues you may encounter. Note that Kubernetes specific or other network connectivity errors are out of scope.
