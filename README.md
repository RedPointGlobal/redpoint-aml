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

- MongoDB Database for AML system databases
    - Version 5 or later
    - 16 GB Memory or more
    - 100 GB or more free disk space.
    - SSD disks for best IO perfomance
    - The MongoDB database can be any of the following
       - Azure Cosmos DB
       - MongoDB on Linux
       - MongoDB Atlas

- PostgreSQL Database for AML Auth service (Keycloak)
    - Version 12 or later
    - 2 GB Memory or more

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
3. Have a MongoDB server available to use for AML system databases
4. Have a PostgreSQL server available to use for AML Auth databases (Keycloak)
5. Docker ID, create one at https://hub.docker.com/ and provide the account ID to Redpoint Support so they can grant you permissions to pull the AML container images
5. Have a license key to activate AML. Contact Redpoint support for an activation key

| **NOTE:** Before you Begin!           |
|---------------------------------------|
| This guide focuses on Microsoft Azure for the underlying Kubernetes infrastructure, and creates Kubernetes deployments for MongoDB and PostgreSQL. This is only intended for use in a ```DEMO``` or ```DEV``` setting. 

To make this production ready, you need to plug into your production MongoDB and PostgreSQL database servers. Be sure to check out the ```Customize for Production``` section down below  |

### Install Procedure

1. Clone this repository
```
git clone https://github.com/RedPointGlobal/redpoint-aml.git
```
2. Make sure you are inside the ```redpoint-aml``` directory 
```
cd redpoint-aml
```
3. Create the kubernetes secret that contains your docker hub credentials 
```
kubectl create secret docker-registry docker-io --docker-server='https://index.docker.io/v1/' \
--docker-username=$your_docker_username --docker-password=$your_docker_password \
--namespace redpoint-aml
```
4. Create the kubernetes secret that contains the certificate files for your custom domain. This is used by the default nginx ingress controller to terminate TLS for your custom domain. If this secret is not created, you will get a ```502 Bad Gateway``` when you try to access the AML Web UI
```
kubectl create secret tls ingress-tls --cert=$your_tls_cert --key=$your_tls_key --namespace redpoint-aml
```
If you prefer to use a different Ingress solution, disable the default ingress as described here [AML Ingress ](#aml-ingress) 

5. Run the following command to install AML
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
It may take a few minutes for the all the AML services to start. Please wait about 5-10 minutes before proceeding to retrieve the ingress endpoints in the next step.

### AML Endpoints
The default installation includes an Nginx ingress controller that exposes the relevant AML endpoints based on the domain specified in the ingress section within the ```values.yaml```domain. 
```
ingress:
  host: redpoint-aml.example.com
```
Run the command below to retrieve the AML endpoints. 
```
NAMESPACE="redpoint-aml"; INGRESS_IP=""; while true; do INGRESS_IP=$(kubectl get ingress --namespace $NAMESPACE -o jsonpath="{.items[0].status.loadBalancer.ingress[0].ip}"); if [ -n "$INGRESS_IP" ]; then echo "IP address found: $INGRESS_IP"; kubectl get ingress --namespace $NAMESPACE; break; else echo "No IP address found, waiting for 10 seconds before checking again..."; sleep 10; fi; done
```
This command will keep checking the ingress IP address every 10 seconds until it finds one. Once an IP address is found, it will display the IP and the corresponding ingress hostname as shown below;

```
redpoint-aml.example.com
```
Next you need to create the corresponding DNS record in your DNS zone 

At this point, the default installation is complete. You can now access the AML User login and Swagger endpoints as described below
```
https://redpoint-aml.example.com              # Web UI and User login
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
To configure AML integration with Redpoint Interaction (RPI), you need to provide the FQDN of the RPI server. This can be done by updating the section below in the ```values.yaml``` file. Ensure the AML deployment has network and firewall connectivity with the RPI server on port 443
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
