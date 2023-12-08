![rg1_SMALL](https://github.com/RedPointGlobal/redpoint-aml/assets/42842390/12f47916-ad62-406f-8da3-d34e7e3a3a4b)
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
--namespace ingress-tls \
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
helm install redpoint-mercury redpoint-mercury/ --values values.yaml

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
It may take some time for all the Mercury services to fully initialize. We recommend waiting approximately 5-10 minutes to ensure that the services are completely up and running. This patience is crucial for the successful retrieval of ingress endpoints in the subsequent step.

### Mercury Endpoints
To view the Mercury endpoints, use the following kubectl command. This command lists all the ingress resources in the redpoint-mercury namespace, showing you the configured endpoints.
```
kubectl get ingress --namespace redpoint-mercury
```
Initially, you might not see an IP address for your endpoints. This delay is normal and occurs because it takes some time for the ingress load balancer to be provisioned. If no IP address is displayed, wait a few minutes and then re-run the command. Once the load balancer is ready, you should see output similar to the following, where <Load Balancer IP> will be replaced with the actual IP address:
```
dcc-admin-api-ingress     redpointmercury.example.com   <Load Balancer IP>   80, 443   32d
dcc-api-ml-docs-ingress   redpointmercury.example.com   <Load Balancer IP>   80, 443   32d
dcc-apis-ingress          redpointmercury.example.com   <Load Balancer IP>   80, 443   32d
dcc-docs-ingress          redpointmercury.example.com   <Load Balancer IP>   80, 443   32d
dcc-ui-ingress            redpointmercury.example.com   <Load Balancer IP>   80, 443   32d
```
After completing the default installation, the next crucial step involves setting up your DNS:

Add a DNS record in your DNS zone. This record should point to the IP address of the load balancer provided by your Kubernetes ingress. This setup ensures that the domain names you use (like redpointmercury.example.com) correctly route to your AML instance.

With the DNS configuration in place, you're ready to access the Mercury interfaces:
```
https://redpointmercury.example.com              # Web UI and User login
https://redpointmercury.example.com/auth/        # Keycloak Web UI
https://redpointmercury.example.com/admin/       # Activation and Admin Setup page
https://redpointmercury.example.com/docs/        # Swagger Authentication and RPI Services API docs
https://redpointmercury.example.com/docs-ml/     # Swagger AML API docs
````
### Mercury Activation
After receiving your activation key from Redpoint Support, you can activate your Mercury instance. Follow these steps to access the Mercury admin UI and enter your license key:

- Navigate to the Mercury admin UI using your web browser. This interface is where you will enter the provided activation key.

- Use the default login credentials that you set in the ```values.yaml``` to access the admin panel:
```
envs:
  DEFAULT_DCC_ADMIN_USER: <your admin username>
  DEFAULT_DCC_ADMIN_PASS: <your admin password>
```
- Once logged in, locate the section for license activation and enter the activation key you received from Redpoint Support.

![image](https://user-images.githubusercontent.com/42842390/218563945-94a5b162-dc59-45ae-900e-130a84810f66.png)

### RPI Integration
Enabling RPI integration will allow Mercury to effectively communicate and synchronize with the RPI server, leveraging its capabilities to enhance overall functionality.

Integrating Mercury with Redpoint Interaction (RPI) requires a few key configurations. Follow these steps to enable and configure RPI integration:

- Ensure that the RPI server is fully set up and operational. This includes having the integration API configured within RPI.

- Verify that there is proper network and firewall connectivity between the Mercury deployment and the RPI server. Specifically, ensure connectivity over port 443, which is used for secure communication

- Modify the envs section of the ```values.yaml```
```
envs:
  RPI_ENABLED: true            # Enable RPI integration
  RPI_CACHES_ENABLED: true     # Enable RPI caching feature
  RPI_NAME: rpi.example.com    # Replace with your RPI server's FQDN
```

### Mercury Documentation
To explore in-depth documentation and stay updated with the latest release notes for Mercury, visit our official documentation portal. This resource offers a wealth of information to help you maximize the potential of Mercury in your environment

 [Mercury Documentation](https://mercury-docs.redpointcloud.com)

### Support 
If you encounter any challenges specific to the Mercury application, our dedicated support team is here to assist you. Please reach out to us with details of the issue for prompt and expert help.

[support@redpointglobal.com](support@redpointglobal.com)

```Note on Scope of Support```
While we are fully equipped to address issues directly related to the Mercury application, please be aware that challenges pertaining to Kubernetes configurations, network connectivity, or other external system issues fall outside our support scope. For these, we recommend consulting with your IT infrastructure team or seeking assistance from relevant technical forums.
