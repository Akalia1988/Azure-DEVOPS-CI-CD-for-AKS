# Azure-DEVOPS-CI-CD-for-AKS
Kubernetes pipeline

Azure DevOps pipelines considered one of the easiest CI/CD pipelines to be configured thanks to Microsoft.
Microservices considered now one of the trends in web apps and platform architectures, there are number of benefits of having microservices based architecture compared to Monolithic architecture. For now, I will talk about configuring Azure DevOps CI/CD pipeline to release a new build to the AKS services.

Kubernetes cluster considered to be a very important containerization orchestration system in the industry now. The solution Microsoft offered to build Microservices using Kubernetes called Azure Kubernetes Service or AKS.

Advantages of Kubernetes
· Deploy your applications quickly and predictably and easily coordinate deployments of your system.
· Constraint communications between containers
· Continuously monitors and manages your containers
· Improves reliability and availability
· Scales your application to handle changes in load on the fly as needed
· Better use of infrastructure resources

So why use Azure AKS solutions
· Hosts your Kubernetes environment
· Easy integration with Azure services such as Load balancing, Azure Blob Storage, Azure Active Directory, Application Gateway, Azure Traffic Manager etc.
· Quick and easy to deploy
· Hosted control plane
· Easy and secure containerized applications management.
· Continuous Integration by adopting Azure Pipeline concept for Docker images creation for faster deployments and reliability
· Create resources and infrastructure inside the Azure Kubernetes cluster through Deployments and services manifest files
· AKS management service is free of charge in Microsoft Azure

AKS Deployment Strategy
1. Developer pushes the source code to Azure repository
2. Azure pipeline triggers the build by cloning the application code from Azure repo to either Microsoft hosted build agent or self-managed build agents
3. Azure build agents are going to build the job and generates deployable artifacts, which can be pushed to a drop location in artifactory staging directory.
![image](https://user-images.githubusercontent.com/58148717/103787882-d09ce800-5003-11eb-9d4b-39dc754ca659.png)
4. Azure pipeline creates docker image with application code artifacts, tag the image, and push to Azure container registry
5. Azure pipeline tag the application code that was containerized, and pushes the code to Azure repo
6. Azure pipeline executes the kubectl commands to re/deploy pods
Redeployment process starts by pulling the latest image from the Azure container registry
7. Azure Kubernetes services deployment process:
Config Map to store non-sensitive environment variables
Secrets to store sensitive environment variables
Persistent volumes, Blob storage, etc.
Azure key vault to store DevOps secrets, service accounts and manage TLS/SSL certificates
Azure Managed Services
8. Create Kubernetes objects (services, Ingress object, etc.) to communicate with pods
9. Provision a load balancer, forwarding rules, and assigning external public IP address (ingress controller)
10. Customers access the application from the public URLs.

Azure DevOps Build Pipeline for Continuous integration
Build pipeline for Kubernetes usually contains the steps that will fetch the code, build the Docker image, push that image to your Docker repository and then publish the artifacts.
1- Get sources from your favorite code repository, Microsoft supports most of the options with ready interfaces for even authenticating and choosing the right repository and default branch.
2- Define the agent you want to run for building your code (Windows agent, Ubuntu agent etc.).
3- Choose the Docker “Build an image” task, choose the Azure container registry with Azure subscription and the name of the registry in case you want to use Azure one (Or you can configure any outside public registry), then choose the docker file name.
![image](https://user-images.githubusercontent.com/58148717/103788113-18bc0a80-5004-11eb-8165-47bbae6b65de.png)
4- Choose the task for docker image push. Also provide Azure container registry information where we can upload docker images for further AKS deployment process.
![image](https://user-images.githubusercontent.com/58148717/103788236-37ba9c80-5004-11eb-83fe-331f1ccf5895.png)
5- Choose the “Copy files” step to copy the files to an artifact staging directory and we will only need to copy the deployment file that we will talk about next in the deployment section.
![image](https://user-images.githubusercontent.com/58148717/103788307-4ef98a00-5004-11eb-9c00-02207116f423.png)
6- Choose Publish build artifacts step

Azure DevOps Release Pipeline for Continuous Deployment
The release pipeline usually used to put the correct build to its correct running environment.
In our scenario usually, that starts by reading the Artifacts from the build pipeline and fires one of the pipeline stages according to the build code branch. 
One stage will be enough with only two tasks, one of them to copy the building code to the deployment yaml file so we can choose the right docker version for the release, and there is a ready pre-defined step for that purpose which is “Replace Tokens”. All you need to do is to select the deployment file from the Artifacts using “Root Directory” and “Target files” fields.
![image](https://user-images.githubusercontent.com/58148717/103788435-718ba300-5004-11eb-8cf1-ac1008361aac.png)
Next you can use “Deploy to Kubernetes” ready task from the templates, that task actually will only run Kubectl apply command on your deployment file. From the “Configuration file” field you can choose the deployment file you will use in that command.
![image](https://user-images.githubusercontent.com/58148717/103788515-85cfa000-5004-11eb-9c29-1e2636aa0304.png)

we need to write our deployment file

Aks-manifestfile.yaml

So this deployment file actually is creating a service for our code (If it’s not being already created), we choose the name of the service and the port which the service will work on. Secondly, the deployment file will define a deployment in the AKS mesh that will take the last Docker image and deploy it to the mesh (After we inject the build id with the previous step).

Note: we set up our CI/CD pipeline and set up deployment on our K8, but that doesn’t mean that our K8 will be able to access the containers registry. We need to configure ACR integration, we can use the next command to support that:

az aks update -n myAKSCluster -g myResourceGroup — attach-acr <acrName>
If it didn’t work then mostly you need to enable az aks preview features by using the following command then repeat the previous one:
az extension add — name aks-preview












