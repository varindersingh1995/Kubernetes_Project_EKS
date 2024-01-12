# Kubernetes_Project_EKS

### Step 1: Configure the AWS CLI

- Install AWS CLI:
Ensure that you have the AWS CLI installed on your machine. You can download and install it from the official AWS CLI website: AWS CLI Installation.

- Open a Terminal or Command Prompt:
Open your terminal or command prompt on your machine.

- Run aws configure command:
This command will prompt you to enter your AWS Access Key ID, Secret Access Key, default region, and output format.

- Enter AWS Access Key ID:
Enter your AWS Access Key ID. This is a unique identifier associated with your AWS account.

- Enter Secret Access Key:
Enter your Secret Access Key. This is the secret key associated with your AWS Access Key ID.

- Enter Default Region:
Enter your preferred AWS region. This is the default region where your AWS CLI commands will be executed if you do not explicitly specify a region.

- Enter Default Output Format:
Enter your preferred output format. This can be json, text, or table. This controls how the AWS CLI outputs information


### Step 2: Installing the EKS

eksctl create cluster --name varinder-cluster --region us-east-1 --fargate

![Cluster](https://github.com/varindersingh1995/Kubernetes_Project_EKS/assets/48336937/d71ac844-c937-434b-b609-0fb971852d21)

*Note*: Creation of the EKS Cluster is a time consuming process so it might take around 20 minutes to get it launched completely

### Step 3: Creating a Fargate Profile

Why we are creating it? cuz we are attaching a new namespace "game-2048" instead of deloying it in the default namespace

That is the only reason we're doing it


```
eksctl create fargateprofile \
    --cluster varinder-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```

![image](https://github.com/varindersingh1995/Kubernetes_Project_EKS/assets/48336937/0a90b62c-60de-4f47-80e2-23776d896c02)

### Step 4: Deploy the deployment, service and Ingress

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

We have to apply the Kubenetes Manifest from a File. The Yaml file defines the specifies the deployment's configuration, such as the number of replicas, the container image to use, and other settings. There is also a Service section which defines the Kubernetes Service named "service-2048" within the "game-2048" namespace. The service exposes the deployed application and allows other components to access it.
Ingress section:
This section defines an Ingress resource named "ingress-2048" within the "game-2048" namespace. Ingress is used to expose HTTP and HTTPS routes to services within a cluster.

The Ingress rules specify that incoming HTTP requests to the root path ("/") should be directed to the "service-2048" service on port 80.


### Step 5: Commands to configure IAM OIDC provider 


```
export cluster_name=demo-cluster
```

```
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5) 
```

![image](https://github.com/varindersingh1995/Kubernetes_Project_EKS/assets/48336937/befeb771-5380-43a0-bcbb-1ccdac586982)

### Step 6: Adding the ALB Controller

Download IAM policy

```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

Create IAM Policy

```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

Create IAM Role

```
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```
![image](https://github.com/varindersingh1995/Kubernetes_Project_EKS/assets/48336937/52ee77f8-c67d-40ca-b2e2-e7143ef12321)


## Deploy ALB controller

Add helm repo

```
helm repo add eks https://aws.github.io/eks-charts
```

Update the repo

```
helm repo update eks
```

Install

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```
![image](https://github.com/varindersingh1995/Kubernetes_Project_EKS/assets/48336937/b8089dba-65d7-41c1-bde3-de8b17853d7f)


Verify that the deployments are running.

```
kubectl get deployment -n kube-system aws-load-balancer-controller
```
![image](https://github.com/varindersingh1995/Kubernetes_Project_EKS/assets/48336937/c062ffdd-0b2c-4e78-ac3c-01f9f4915263)

### Step 7: Get the address of the Load balancer address that Ingress controller created.


![image](https://github.com/varindersingh1995/Kubernetes_Project_EKS/assets/48336937/018314b9-546f-46a1-9072-5828a26ad87b)

### Step 8: Open the Address in the browser  and Here is the Final Result:

![image](https://github.com/varindersingh1995/Kubernetes_Project_EKS/assets/48336937/ed392290-9e63-47cb-8eb7-5ad9d1e3d634)

