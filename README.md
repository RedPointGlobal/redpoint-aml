![RG](https://user-images.githubusercontent.com/42842390/158004336-60f07c05-7e5d-420e-87a6-22c5ac206fb6.jpg)
## Redpoint Automated Machine Learning (Mercury) - Deployment on Kubernetes
Redpoint Automated Machine Learning (Mercury) revolutionizes the way you apply machine learning to your data. It's designed to simplify and streamline the process, making advanced machine learning accessible even if you don't have extensive data science expertise.

This guide will walk you through a step-by-step deployment of Redpoint Mercury on a Kubernetes environment, using Helm for efficient and manageable deployment. 

Below, is a detailed diagram illustrating the deployment architecture. This visual representation will help you understand how Redpoint Mercury integrates within your Kubernetes infrastructure, ensuring a smooth and successful deployment
![803174cf-6ab4-4c8a-998c-7b959ee6b31c](https://user-images.githubusercontent.com/42842390/197833963-45b725f0-947e-4884-914b-c6942d6d883d.png)

### Table of Contents
- [Prerequisites ](#prerequisites)
- [System Requirements ](#system-requirements)
- [Install Procedure ](#install-procedure)
- [Mercury Endpoints ](#aml-endpoints)
- [Mercury Activation ](#install-license)
- [RPI Integration ](#rpi-integration)
- [Mercury Documentation](#aml-documentation)
- [Support](#support)

### System Requirements

- MongoDB Server for System Databases:

  - Version: 5.0 or later.
  - Memory: At least 16 GB.
  - Disk Space: Minimum of 100 GB free space.
  - Disk Type: SSDs recommended for optimal I/O performance.
  - Supported Environments:
    - MongoDB on Linux.
    - MongoDB Atlas.

- PostgreSQL Server for Keycloak:

  - Version: 12.0 or later.
  - Memory: At least 2 GB.

- Kubernetes Cluster:

  - Use the latest stable version of Kubernetes for compatibility and security.
  - Minimum of 2 nodes for high availability.
  - Each node should have at least 8 vCPUs and 16 GB of memory.
  - A minimum of 100 GB free disk space per node.
  - Managed Kubernetes services like Azure Kubernetes Service (AKS), Amazon Elastic Kubernetes Service (EKS), or Google Kubernetes Engine (GKE).
    
### Prerequisites
Before installing Mercury, ensure that the following requirements are met:

1. Access to a Kubernetes cluster is essential. If you don't have one, consider setting up a Kubernetes solution following the guidelines provided by Kubernetes for [Production](https://kubernetes.io/docs/setup/production-environment/turnkey-solutions/)
2. Install kubectl, a command-line tool for interacting with your Kubernetes cluster. Detailed installation instructions can be found [Here](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
3.  Ensure the availability of a MongoDB Server for hosting Mercury databases. 
4. To access Mercury's container images, request access to Redpoint's container registry. Open a support ticket at support@redpointglobal.com requesting access to the Mercury repository.
5. An activation key is required to use Mercury. Contact Redpoint support to obtain your license key


//////////////
Before you install Mercury, you must:

1. Have a Kubernetes solution available to use. ( https://kubernetes.io/docs/setup/production-environment/turnkey-solutions/ )
2. Install kubectl. ( https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/ )
3. Have a MongoDB server available to use for Mercury system databases
4. Have a PostgreSQL server available to use for Keycloak
5. Docker ID, create one at https://hub.docker.com/ and provide the account ID to Redpoint Support so they can grant you permissions to pull the Mercury container images
5. Have a License key to activate Mercury. Contact Redpoint support for an activation key

| **NOTE:** Before you Begin!           |
|---------------------------------------|
| This guide assumes Microsoft Azure is the underlying platform for your Kubernetes infrastructure. However Mercury can also be deployed on clusters within the Amazon and Google Cloud platforms. Before installing Mercury, set the target Cloud platform in the ```values.yaml``` file as shown below

### Install Procedure
The default installation creates kubernetes deployments for MongoDB and PostgreSQL. This is fine for a DEMO environment. However, for production workloads, you must disable these and instead provide connection strings for your production servers as shown in ```STEP 4```

1. Clone the redpoint Mercury repository
```
git clone https://github.com/RedPointGlobal/redpoint-aml.git
```
2. Connect to your Kubernetes cluster and create a namespace for Mercury
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
Then disable the default MongoDB and PostgreSQL deployments as shown below
```
mongodb:
  enabled: false

keycloak:
  postgresql:
    enabled: false
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
7. Run the following command to install Mercury
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
It takes a few minutes for the all the Mercury services to start. Please wait about 5-10 minutes before proceeding to retrieve the ingress endpoints in the next step.

### Mercury Ingress
8. The default installation creates an nginx ingress controller to expose the Mercury Web UI endpoints. To terminate TLS, provide a TLS certificate for your custom domain by creating the kubernetes secret containing your certificate data
```
kubectl create secret tls ingress-tls --cert=$your_tls_cert --key=$your_tls_key --namespace redpoint-aml
```
If you prefer to use a different Ingress solution, you can disable the default ingress creation in the ```values.yaml``` file as shown below
```
nginx:
  enabled: true # Change this to False to disable Nginx
```

### Mercury Endpoints
Run the command below to retrieve the Mercury endpoints. 
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

At this point, the default installation is complete. You can now access the Mercury User login and Swagger endpoints as described below
```
https://redpoint-aml.example.com              # Web UI and User login
https://redpoint-aml.example.com/auth/        # Keycloak Web UI
https://redpoint-aml.example.com/admin/       # Activation and Admin Setup page
https://redpoint-aml.example.com/docs/        # Swagger Authentication and RPI Services API docs
https://redpoint-aml.example.com/docs-ml/     # Swagger Mercury API docs
```
### Mercury Activation
Once you obtain your activation key from Redpoint Support, access the Mercury admin UI and enter the license.
```
Default username: admin@noemail.com
Default password: .RedPoint123
```
![image](https://user-images.githubusercontent.com/42842390/218563945-94a5b162-dc59-45ae-900e-130a84810f66.png)

### RPI Integration
To configure Mercury integration with Redpoint Interaction (RPI), you need to provide the FQDN of the RPI server. This can be done by updating the section below in the ```values.yaml``` file. Before enabling RPI integration, the integration API setup must be completed in RPI. Ensure the Mercury deployment has network and firewall connectivity with the RPI server on port 443
```
envs:
  RPI_ENABLED: false            # Change this to true
  RPI_CACHES_ENABLED: false     # Change this to true
  RPI_NAME: rpi.example.com     # Replace with the FQDN of your RPI server
```

### Mercury Documentation
For detailed Mercury documentation and release notes, please visit the official Mercury documentation website at the address below

https://mercury-docs.redpointcloud.com

### Support 
Contact support@redpointglobal.com for any application specific issues you may encounter. Note that Kubernetes specific or other network connectivity errors are out of scope.
