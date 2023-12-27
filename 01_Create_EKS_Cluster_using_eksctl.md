# Create EKS Cluster using eksctl

## Step-01: Create Cluster
```bash
eksctl create cluster --name=eksdemo1 \
    --region=us-east-1 \
    --zones=us-east-1a,us-east-1b \
    --without-nodegroup 
```

### Get list of Clusters
```bash
eksctl get Clusters
```
### Get list of Nodes - no nodes should be displayed
```bash
kubectl get nodes
```

## Step-02: Create & Associate IAM OIDC Provider for our EKS Cluster
#### Replace with region & cluster name
```bash
# Template
eksctl utils associate-iam-oidc-provider \
    --region region-code \
    --cluster <cluter-name> \
    --approve

# Replace with region & cluster name
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster eksdemo1 \
    --approve
```

## Step-03: Create EC2 Keypair
- Create a new EC2 Keypair with name as `kube-demo`
- This keypair we will use it when creating the EKS NodeGroup.
- This will help us to login to the EKS Worker Nodes using Terminal.

## Step-04: Create Node Group with additional Add-Ons in Public Subnets
- These add-ons will create the respective IAM policies for us automatically within our Node Group role.
- t3.medium - 17 pods
 ```
eksctl create nodegroup --cluster=eksdemo1 \
    --region=us-east-1 \
    --name=eksdemo1-ng-public1 \
    --node-type=t3.medium \
    --nodes=2 \
    --nodes-min=2 \
    --nodes-max=4 \
    --node-volume-size=20 \
    --ssh-access \
    --ssh-public-key=kube-demo \
    --managed \
    --asg-access \
    --external-dns-access \
    --full-ecr-access \
    --appmesh-access \
    --alb-ingress-access 
```
### Try to add Security Group to this command
--node-security-groups strings 

### Verify Cluster, NodeGroup in EKS Management Console
- Go to Services -> Elastic Kubernetes Service -> eksdemo1

### List Worker Nodes
```
# List EKS clusters
eksctl get cluster

# List NodeGroups in a cluster
eksctl get nodegroup --cluster=<clusterName>

# List Nodes in current kubernetes cluster
kubectl get nodes -o wide

# Our kubectl context should be automatically changed to new cluster
kubectl config view --minify
```
### Verify Worker Node IAM Role and list of Policies
- Go to Services -> EC2 -> Worker Nodes
- Click on **IAM Role associated to EC2 Worker Nodes**

### Verify Security Group Associated to Worker Nodes
- Go to Services -> EC2 -> Worker Nodes
- Click on **Security Group** associated to EC2 Instance which contains `remote` in the name.

### Verify CloudFormation Stacks
- Verify Control Plane Stack & Events
- Verify NodeGroup Stack & Events

### Login to Worker Node using Keypai kube-demo
- Login to worker node
- ssh -i kube-demo.pem ec2-user@<Public-IP-of-Worker-Node>
- df -h = disk space of node

