# argocd-setup

## Introduction
This documentation provides a comprehensive guide for setting up and deploying a microservice-based application on a Kubernetes cluster. It covers various aspects such as cluster setup, Docker image creation, Kubernetes deployment, autoscaling, and CI/CD integration.

## Prerequsites:  
- AWS account
- EKS Cluster
- AWS access credentials

## Kubernetes Setup
Kubernetes Cluster was setup using AWS EKS. The configuration file (personal-eks.yaml) can be found in the root folder of the repository. EKS cluster nodes were placed in the private subnets for security purpose with NAT gateway configured and a Network LoadBalancer placed in public subnet. EFS volume was also setup for persistent storage for the database that will be used by the microservice applications.

- `eksctl create cluster -f personal-eks.yaml`
- `eksctl delete cluster EksCluster`

## Dockerization
The microsoervice application was dockerized and built using a dockerfile which can be found in the microservice-based-app repository.

- `docker build -t frontend .`
- `docker build -t backend .`


## Kubernetes Deployment
Kubernetes manifest was created for each microservice which can be found inside the my-app folder in the root repository. The manifest consist of a Frontend and Backend. Altough we have a database created for the microservice application, this was deployed using helm chart which can be found in db folder.
For each of the workload specification of desired replicas was defined in the manifest file. Resource requests and limits were appropriately defined by monitoring the metrics usage of the microservice based application in the manifest file.

- `kubectl apply -f my-app/frontend/deployment.yaml`
- `kubectl apply -f my-app/frontend/service.yaml`
- `kubectl apply -f my-app/backend/deployment.yaml`
- `kubectl apply -f my-app/backend/service.yaml`
- `helm upgrade -i postgres-db oci://registry-1.docker.io/bitnamicharts/postgresql -f db/values.yaml -n simple-inventory-app`

## Autoscaling
Horizontal Pod autoscaler was setup and configured for this project based on CPU utilization. In other words when the CPU usage is high, it triggers the Horizontal pod autoscaler and more pods are added to handle the workload when the CPU usage decreases Horizontal pod autoscaler automatically scales down the number of pods. The YAML file hpa.yaml can be found in the root folder of the repository.

- `kubectl apply -f hpa.yaml`

## CI/CD integration 
For this Project, Automatic deployments has been setup using Github Actions and ArgoCD. Basically we have 2 repositories which are:

- `microservice-based-app`
- `argocd-setup`

### microservice-based-app repository
microservice-based-app repo contains the code, dockerfile and workflow files.

- `frontend.yml`
- `backend.yml`

The workflow files are used to define automated workflows that triggers Github Actions in response to a push to the repository. It consists of a Job that builds the code using dockefile, tags the image and push the image to AWS ECR. So Once the image has been pushed to the REPO ECR, the workflow updates the manifest file in argocd-setup repository with the newest image. 

### argocd-setup repository 
This repository contains the setup for the Continous deployment. ArgoCD was setup in EKS cluster which also a yaml file called application.yaml in the root directory of the repo. ArgoCD continuously monitors the git repository for changes to Kubernetes manifests and automically applies the changes to the EKS cluster.   


## Conclusion
This documentation provides a comprehensive guide for deploying a microservice-based application on Kubernetes. By following the outlined steps, teams can efficiently manage their application deployments, ensure scalability, and streamline the development workflow.