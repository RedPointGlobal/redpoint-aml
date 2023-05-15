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
- [AML Endpoints ](#aml-endpoints)
- [AML Activation ](#install-license)
- [RPI Integration ](#rpi-integration)
- [AML Documentation](#aml-documentation)
- [Support](#support)

### System Requirements

- MongoDB server for system databases
    - Version 5 or later
    - 16 GB Memory or more
    - 100 GB or more free disk space.
    - SSD disks for best IO perfomance
    - The MongoDB database can be any of the following
       - Azure Cosmos DB
       - MongoDB on Linux
       - MongoDB Atlas

- PostgreSQL server for Keycloak 
    - Version 12 or later
    - 2 GB Memory or more

- Kubernetes Cluster
    - Latest stable version of Kubernetes
    - Nodepool with 2 or more nodes for high availabilty
    - 8 vCPUs and 16 GB memory per node
    - 100 GB or more free disk space per node
    - The Kubernetes cluster can be any of the following
       - Self hosted Kubernetes
       - Managed Kubernetes (AKS, EKS, GKE)
    
### Prerequisites

Before you install AML, you must:

1. Have a Kubernetes solution available to use. ( https://kubernetes.io/docs/setup/production-environment/turnkey-solutions/ )
2. Install kubectl. ( https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/ )
3. Have a MongoDB server available to use for AML system databases
4. Have a PostgreSQL server available to use for Keycloak
5. Docker ID, create one at https://hub.docker.com/ and provide the account ID to Redpoint Support so they can grant you permissions to pull the AML container images
5. Have a License key to activate AML. Contact Redpoint support for an activation key

| **NOTE:** Before you Begin!           |
|---------------------------------------|
| This guide assumes Microsoft Azure is the underlying platform for your Kubernetes infrastructure. However AML can also be deployed on clusters within the Amazon and Google Cloud platforms. Before installing AML, set the target Cloud platform in the ```values.yaml``` file as shown below

### Install Procedure
1. Clone the redpoint AML repository
```
git clone https://github.com/RedPointGlobal/redpoint-aml.git
```
2. Connect to your Kubernetes cluster and create a namespace for AML
```
kubectl create namespace redpoint-aml
```
3. Create a kubernetes secret that contains your docker hub credentials.
```
kubectl create secret docker-registry docker-io --docker-server='https://index.docker.io/v1/' \
--docker-username=$your_docker_username --docker-password=$your_docker_password \
--namespace redpoint-aml
```
4. Edit the ```values.yaml``` file and provide connection strings for your MongoDB and PostgreSQL servers as shown below
```
mongodb:
  connection_string: # <your mongodb connection string>

keycloak:
  postgresql:
    db_address: postgresql          # Your Postgresql server address
    db_user: keycloak               # Your Postgresql server admin user
    db_password: 7rU8w9o8ocTa8Zp1   # Your Postgresql server admin password
```
5. Edit the ```values.yaml``` file and set your target cloud platform as shown below
```
global:
  cloudProvider: azure # or google or amazon   
```
6. Make sure you are inside the ```redpoint-aml``` directory 
```
cd redpoint-aml
```
7. Run the following command to install AML
```
helm install redpoint-aml redpoint-aml/ --values values.yaml --create-namespace
 ```
If everything goes well, You should see the output below.
```
Release "redpoint-aml" has been installled. Happy Helming!
NAME: redpoint-aml
LAST DEPLOYED: Fri Apr 21 17:36:41 2023
NAMESPACE: redpoint-aml
STATUS: deployed
REVISION: 2
TEST SUITE: None
```
It takes a few minutes for the all the AML services to start. Please wait about 5-10 minutes before proceeding to retrieve the ingress endpoints in the next step.

### AML Ingress
8. The default installation creates an nginx ingress controller to expose the AML Web UI endpoints. To terminate TLS, provide a TLS certificate for your custom domain by creating the kubernetes secret containing your certificate data
```
kubectl create secret tls ingress-tls --cert=$your_tls_cert --key=$your_tls_key --namespace redpoint-aml
```
If you prefer to use a different Ingress solution, you can disable the default ingress creation in the ```values.yaml``` file as shown below
```
nginx:
  enabled: true # Change this to False to disable Nginx
```

### AML Endpoints
Run the command below to retrieve the AML endpoints. 
```
kubectl get ingress --namespace redpoint-aml
```
If you dont see an IP address, it because it takes a couple minutes for the ingress load balancer to be created. Wait a few minutes and run the command once more and you should eventually see an IP address returned as shown in the example below

```
dcc-admin-api-ingress     nginx-redpoint-aml   redpoint-aml.example.com   < Load Balancer IP>   80, 443   32d
dcc-api-ml-docs-ingress   nginx-redpoint-aml   redpoint-aml.example.com   < Load Balancer IP>   80, 443   32d
dcc-apis-ingress          nginx-redpoint-aml   redpoint-aml.example.com   < Load Balancer IP>   80, 443   32d
dcc-docs-ingress          nginx-redpoint-aml   redpoint-aml.example.com   < Load Balancer IP>   80, 443   32d
dcc-ui-ingress            nginx-redpoint-aml   redpoint-aml.example.com   < Load Balancer IP>   80, 443   32d
```
Next you need to create the corresponding DNS record in your DNS zone 

At this point, the default installation is complete. You can now access the AML User login and Swagger endpoints as described below
```
https://redpoint-aml.example.com              # Web UI and User login
https://redpoint-aml.example.com/auth/        # Keycloak Web UI
https://redpoint-aml.example.com/admin/       # Activation and Admin Setup page
https://redpoint-aml.example.com/docs/        # Swagger Authentication and RPI Services API docs
https://redpoint-aml.example.com/docs-ml/     # Swagger AML API docs
```
### AML Activation
Once you obtain your activation key from Redpoint Support, access the AML admin UI and enter the license.
```
Default username: admin@noemail.com
Default password: .RedPoint123
```
![image](https://user-images.githubusercontent.com/42842390/218563945-94a5b162-dc59-45ae-900e-130a84810f66.png)

### RPI Integration
To configure AML integration with Redpoint Interaction (RPI), you need to provide the FQDN of the RPI server. This can be done by updating the section below in the ```values.yaml``` file. Before enabling RPI integration, the integration API setup must be completed in RPI. Ensure the AML deployment has network and firewall connectivity with the RPI server on port 443
```
envs:
  RPI_ENABLED: false            # Change this to true
  RPI_CACHES_ENABLED: false     # Change this to true
  RPI_NAME: rpi.example.com     # Replace with the FQDN of your RPI server
```

### AML Documentation
For detailed AML documentation and release notes, please visit the official AML documentation website at the address below

https://mercury-docs.redpointcloud.com

### Support 
Contact support@redpointglobal.com for any application specific issues you may encounter. Note that Kubernetes specific or other network connectivity errors are out of scope.
