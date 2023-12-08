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
3.  Ensure the availability of a MongoDB and a PostgreSQL server for hosting Mercury databases. 
4. To access Mercury's container images, request access to Redpoint's container registry. Open a support ticket at support@redpointglobal.com requesting access to the Mercury repository.
5. An activation key is required to use Mercury. Contact Redpoint support to obtain your license key

| **NOTE:** Before you Begin!           |
|---------------------------------------|
| This guide is primarily tailored for deployments on Microsoft Azure. However, Mercury is also compatible with Amazon Web Services (AWS) and Google Cloud Platform (GCP). Ensure you select the appropriate cloud provider in the values.yaml file before proceeding with the installation. This setting can be found in the global section of values.yaml.|

### Install Procedure
Before installing Mercury, follow these preparatory steps to ensure a smooth setup:

- Configure Database Settings:

Ensure you have correctly configured the MongoDB and PostgreSQL Server details in the databases section of the ```values.yaml``` file. This includes setting the correct server address, username, password, database names, and other relevant settings.
```
  databases:
    mongodb:
    keycloak:
```

- Select Cloud Provider:

In the ```values.yaml``` file, under the global application settings, specify the cloud provider where your infrastructure is hosted. Supported providers include Azure, AWS, and GCP. This setting ensures that Mercury aligns with your cloud infrastructure.
```
  cloudProvider: azure
  deploymentType: client
```

- Create Kubernetes Namespace:

Run the following command to create a Kubernetes namespace where the RPI services will be deployed:
```
kubectl create namespace redpoint-mercury
```

- Create Docker Registry Secret:

Create a Kubernetes secret containing the image pull credentials for the Redpoint container registry. These credentials are provided by Redpoint Support. Replace <your_username> and <your_password> with your actual credentials:
```
kubectl create secret docker-registry docker-io \
--namespace redpoint-mercury \
--docker-server=rg1acrpub.azurecr.io \
--docker-username=<your_username> \
--docker-password=<your_password>
```
- Create TLS Certificate Secret:

If you are using SSL for RPI access endpoints, create a Kubernetes secret containing your TLS certificate's private and public keys. Replace path/to/tls.cert and path/to/tls.key with the actual paths to your certificate files:
```
kubectl create secret tls tls-secret \
--namespace redpoint-mercury \
--cert=path/to/tls.cert \
--key=path/to/tls.key
```
After completing the above steps, proceed with the installation:

- Clone the RPI repository to your local machine:
```
git clone https://github.com/RedPointGlobal/redpoint-aml.git
```
- Change into the cloned repository's directory:
```
cd redpoint-mercury
```
- Execute the following Helm command to install RPI on your Kubernetes cluster, using the configurations set in your ```values.yaml``` file:
```
helm install redpoint-mercury redpoint-mercury/ --values ```values.yaml```

```
If everything goes well, You should see the output below.
```
Release "redpoint-mercury" has been installled. Happy Helming!
NAME: redpoint-mercury
LAST DEPLOYED: Fri Apr 21 17:36:41 2023
NAMESPACE: redpoint-mercury
STATUS: deployed
REVISION: 2
TEST SUITE: None
```
It takes a few minutes for the all the Mercury services to start. Please wait about 5-10 minutes before proceeding to retrieve the ingress endpoints in the next step.

### Mercury Ingress
8. The default installation creates an nginx ingress controller to expose the Mercury Web UI endpoints. To terminate TLS, provide a TLS certificate for your custom domain by creating the kubernetes secret containing your certificate data
```
kubectl create secret tls ingress-tls --cert=$your_tls_cert --key=$your_tls_key --namespace redpoint-mercury
```
If you prefer to use a different Ingress solution, you can disable the default ingress creation in the ```values.yaml``` file as shown below
```
nginx:
  enabled: true # Change this to False to disable Nginx
```

### Mercury Endpoints
Run the command below to retrieve the Mercury endpoints. 
```
kubectl get ingress --namespace redpoint-mercury
```
If you dont see an IP address, it because it takes a couple minutes for the ingress load balancer to be created. Wait a few minutes and run the command once more and you should eventually see an IP address returned as shown in the example below

```
dcc-admin-api-ingress     nginx-redpoint-mercury   redpoint-mercury.example.com   < Load Balancer IP>   80, 443   32d
dcc-api-ml-docs-ingress   nginx-redpoint-mercury   redpoint-mercury.example.com   < Load Balancer IP>   80, 443   32d
dcc-apis-ingress          nginx-redpoint-mercury   redpoint-mercury.example.com   < Load Balancer IP>   80, 443   32d
dcc-docs-ingress          nginx-redpoint-mercury   redpoint-mercury.example.com   < Load Balancer IP>   80, 443   32d
dcc-ui-ingress            nginx-redpoint-mercury   redpoint-mercury.example.com   < Load Balancer IP>   80, 443   32d
```
Next you need to create the corresponding DNS record in your DNS zone 

At this point, the default installation is complete. You can now access the Mercury User login and Swagger endpoints as described below
```
https://redpoint-mercury.example.com              # Web UI and User login
https://redpoint-mercury.example.com/auth/        # Keycloak Web UI
https://redpoint-mercury.example.com/admin/       # Activation and Admin Setup page
https://redpoint-mercury.example.com/docs/        # Swagger Authentication and RPI Services API docs
https://redpoint-mercury.example.com/docs-ml/     # Swagger Mercury API docs
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
