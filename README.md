# Sample implementation of a Flask App deployed over AWS EKS.

Had started with the Kubernetes tutorial over here https://kubernetes.io/blog/2019/07/23/get-started-with-kubernetes-using-python/.
The example talks about deploying a simple Flask app into Docker Desktop. I felt AWS EKS is superior.

## Step 1 - Installations.
I did all my development by spinning up a temporary EC2 linux instance. I needed to install 
* AWS CLI
* kubectl - can find by googling 'AWS kubectl'
* eksctl - I found the link here: https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html

## Step 2 - configure AWS CLI
You need to download the AWS Key and Secret. Then use aws configure to set it up.

Use these two commands to verify that you have the correct configuration.
aws sts get-caller-identity
aws sts get-session-token

## Step 3 - Create an EKS Cluster with nodes.
This is easy with eksctl. This is the command I used;
eksctl create cluster --name my-cluster3  --version 1.21 --region us-east-1 --nodegroup-name my-nodes --node-type t2.micro --nodes 2
The above creates a cluster with 2 nodes. I plan to have a pod on each node. (refer the deployment.yaml)

It looks like the EKS cluster needs to sit in it's own VPC. eksctl creates the new VPC and takes care of subnet creation, security groups, roles tec. It also creates a CloudFormation template which is useful.

Use following command to verify that you can connect to the server.
kubectl version --short

can also use the below to verify the configuration
kubectl config view

## Step 4 - Create the Docker Image locally.
I grabbed the code from here. This is a simple Hello World application https://kubernetes.io/blog/2019/07/23/get-started-with-kubernetes-using-python/
git clone https://github.com/vvr-rao/test-Flask-Kubernetes.git
cd test-Flask-Kubernetes
docker build -f Dockerfile -t hello-python:latest .

use the below to verify:
docker images

use the below to test:
docker run -p 5001:5000 hello-python
curl http://localhost:5001/

## Step 5 - Push the image to a repository. There are a couple of options - DockerHub and AWS ECR.

### For Dockerhub, commands are as follows:
docker tag hello-python:latest <YOUR_DOCKERHUB_USER_NAME>/hello-python:latest
docker push <YOUR_DOCKERHUB_USER_NAME>/hello-python:latest

### For ECR, 
You can create a public repository in the console and them use the 'View Push Commands" to ge the correct commands.

Whatever you choose, make sure to set it in the image: entry
I set imagePullPolicy: IfNotPresent

## Step 6 - Deploy the App to EKS
kubectl apply -f deployment.yaml

use the below to check status of deployment
kubectl get nodes

If you see an error, use this to delete the service and troubleshoot. This should delete the Pods.:
kubectl delete -f deployment.yaml

Note that I set replicas: 2 - so there will be 2 pods, one in each node

## Step 7 - View.
Use the below to get the service.
kubectl get svc
That should get you the endpoint.

You can access the app by using
curl http://<endpoint>:6000
  
## Step 8 - Cleanup
To delete the cluster including the nodes and CloudFormation template use this
eksctl delete cluster --region us-east-1 --name my-cluster2
  
## Other useful commands.
To stop a node for troublesooting:
  kubectl drain  <NODE_NAME> --ignore-daemonsets  --delete-emptydir-data 

To Make the node available again:
  kubectl uncordon <NODE_NAME> 
