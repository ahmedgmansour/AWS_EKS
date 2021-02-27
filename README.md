# AWS_EKS
What is Amazon EKS?
Amazon EKS (Elastic Container Service for Kubernetes) is a managed Kubernetes service that allows you to run Kubernetes on AWS without the hassle of managing the Kubernetes control plane.
The Kubernetes control plane (API)plays a crucial role in a Kubernetes deployment as it is responsible for how Kubernetes communicates with your cluster — starting and stopping new containers, scheduling containers, performing health checks, and many more management tasks.
EKS will provision, scale and manage the Kubernetes control plane for you to ensure high availability, security and scalability.
You will need to make sure you have the following components installed and set up before you start with Amazon EKS:
AWS CLI – while you can use the AWS Console to create a cluster in EKS, the AWS CLI is easier. 
Kubectl – used for communicating with the cluster API server.
eksctl : is a simple CLI tool for creating clusters on EKS - Amazon's new managed Kubernetes service for EC2. It is written in Go, and uses CloudFormation.
install _kubectl
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
Ekstl – setup and operation of EKS cluster
Setup of eksctl : 
curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp  
#sudo mv /tmp/eksctl /usr/local/bin


Sign in to your AWS account at https://aws.amazon.com with an IAM user role that has the necessary

Step 1: Creating an EKS role
Our first step is to set up a new IAM role with EKS permissions.
Open the IAM console, select Roles on the left and then click the Create Role button at the top of the page.
From the list of AWS services, select EKS and then Next: Permissions at the bottom of the page.
- AmazonEC2FullAccess
  - IAMFullAccess
  - AmazonVPCFullAccess
  - CloudFormation-Admin-policy
  - EKS-Admin-policy
Be sure to note the Role ARN, you will need it when creating the Kubernetes cluster in the steps below.
Create SSH key For IAM User to be able to ssh to my EC2 instances. 
As you may have seen according to the PEM file including the key please ensure that you copy it to proper and safe folder so that no one except you has access to this folder.
Then back to dashboard and search for IAM to user then choose secuirty credetials 
This is the only time that the secret access keys can be viewed or downloaded. You cannot recover them later. However, you can create new access keys at any time. (Use access keys to make programmatic calls to AWS from the AWS CLI, )
Creating the EKS cluster with the easy way
On local machine:
#mkdir eksctl
#vi eksctl/eks-aws.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: EKS-AWS
  region: us-east-1
nodeGroups:
  - name: ng-1
    instanceType: t2.small
    desiredCapacity: 3
    ssh: # use existing EC2 key
      publicKeyName: eks-AWS
-eksctl will automatically creating 2 subnets and two different availability zone
then run the below command:
#eksctl create cluster -f eksctl/eks-aws.yaml
# aws eks update-kubeconfig --name AWS_EKS
#kubectl get nodes
The cluster up and running.


Installing Ansible:
For installing ansible We will create EC2 instance on the same VPC and subnet of cluster to assign IP in the same range.
Connect to the server (Ansible) using SSH:
#ssh -i C:\Users\Devops\Downloads\EKS.pem ec2-user@34.224.27.254
Then install the below packages:
#sudo amazon-linux-extras install ansible2 
#sudo yum install -y python3 && sudo yum install -y python3-pip

we are using Ansible as our deployment tool. There are many other ways to deploy Kubernetes resources, but I thought Ansible is a much easier option. Ansible uses playbooks to organize its instructions.
Ansible already includes the k8s module for handling communication with the Kubernetes API server.


The Jenkins Operator is a Kubernetes native Operator which manages operations for Jenkins on Kubernetes. It was built with immutability and declarative configuration as code in mind. The Jenkins Operator is easy to install with just a few manifest and allows users to 
configure and manage Jenkins on Kubernetes.

Create a manifest e.g. jenkins_instance.yaml with the following data

apiVersion: jenkins.io/v1alpha2
kind: Jenkins
metadata:
  name: example
spec:
  master:
    containers:
    - name: jenkins-master
      image: jenkins/jenkins:lts-jdk11
      imagePullPolicy: Always
      livenessProbe:
        failureThreshold: 12
        httpGet:
          path: /login
          port: http
          scheme: HTTP
        initialDelaySeconds: 80
        periodSeconds: 10
        successThreshold: 1
        timeoutSeconds: 5
      readinessProbe:
        failureThreshold: 3
        httpGet:
          path: /login
          port: http
          scheme: HTTP
        initialDelaySeconds: 30
        periodSeconds: 10
        successThreshold: 1
        timeoutSeconds: 1
      resources:
        limits:
          cpu: 1500m
          memory: 3Gi
        requests:
          cpu: "1"
          memory: 500Mi
  seedJobs:
  - id: jenkins-operator
    targets: "cicd/jobs/*.jenkins"
    description: "Jenkins Operator repository"
    repositoryBranch: master
    repositoryUrl: https://github.com/jenkinsci/kubernetes-operator.git

Ansible uses playbooks to organize its instructions. Our playbook.yml file looks as follows:
- hosts: NodeGroup
  tasks:
  - name: Deploy the service
    k8s:
      state: present
      definition: "{{ lookup('template', 'jenkins_instance.yaml') | from_yaml }}"
      validate_certs: no
      namespace: default

#ansible-playbook -i Playbook.yaml 
Browse to http://localhost:8080 or Public IP (or whichever port you configured for Jenkins when installing it) .

Watch the Jenkins instance being created:
$ kubectl get pods -n Jenkins

Connect to Jenkins (actual Kubernetes cluster)
$ kubectl port-forward Jenkins-AWS_EKS 8080:8080




CI/CD pipeline:
Log in to Jenkins
1-Install plugins of Kubernetes.
2- add Kubernetes credentials. 
3- will add my kubernetes credentials so that jenkins access and authenticate with k8s master to carry out deployment 
To add kubeconfig key I have to login to the Master Node to get conext:
#cat ~/.kube/config 

The copy the entire context to kubconfig in jenkins configuration.
Create Pipeline:
Then choose multibranch pipleline.
Deployiong on kuberentes :

Create new file (AWS_EKS.yml)
After committing it will automatically build thew new changes.
Kubernetes spinning up my application. 
