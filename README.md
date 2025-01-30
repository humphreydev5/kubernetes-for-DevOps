# Step-by-Step Guide: Installing Kubernetes on AWS EC2 Using KOPS
> Perfect for Beginners – Includes Cost-Saving Tips!

## Prerequisites
1. AWS Account: With permissions to create EC2, S3, IAM, and VPC resources.

2. EC2 Instance:

 - OS: Ubuntu 20.04/22.04 (t2.micro for testing, but t3.small recommended for stability).

 - IAM Role: Attach AmazonEC2FullAccess, AmazonS3FullAccess, IAMFullAccess, and AmazonVPCFullAccess.


### 1. Install Dependencies

 Install Python, kubectl, AWS CLI, and KOPS
```
sudo apt-get update  
sudo apt-get install -y python3-pip apt-transport-https
```

Add Kubernetes repo and install kubectl

```
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
``` 
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
```
sudo apt-get update && sudo apt-get install -y kubectl
```

Install AWS CLI
```
pip3 install awscli --upgrade
```
```
echo 'export PATH="$PATH:/home/ubuntu/.local/bin/"' >> ~/.bashrc
```
```
source ~/.bashrc
```
Try this if pip is working on your ubuntu
> This works fine for me
```
sudo snap install aws-cli --classic
```
```
aws --version
```

### 2. Install KOPS
Download the latest KOPS binary
```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
```

Make it executable and move to PATH
```
chmod +x kops-linux-amd64
```
```
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

### 3. Configure AWS CLI
```
aws configure
```
> Follow prompts to enter AWS Access Key, Secret Key, Region (e.g., us-east-1), and output format (json)


4. Create S3 Bucket for KOPS State
bash
Copy
aws s3api create-bucket \
  --bucket kops-abhi-storage \
  --region us-east-1 \
  --create-bucket-configuration LocationConstraint=us-east-1  
5. Create the Kubernetes Cluster
bash
Copy
# Generate cluster configuration
kops create cluster \
  --name=demok8scluster.k8s.local \
  --state=s3://kops-abhi-storage \
  --zones=us-east-1a \
  --node-count=1 \
  --node-size=t2.micro \
  --master-size=t2.micro \
  --master-volume-size=8 \
  --node-volume-size=8  

# Edit cluster config (optional, to tweak settings)
kops edit cluster demok8scluster.k8s.local --state=s3://kops-abhi-storage  

# Build the cluster
kops update cluster demok8scluster.k8s.local --yes --state=s3://kops-abhi-storage  
Note: Cluster creation takes 10–15 minutes.

6. Verify Cluster Health
bash
Copy
kops validate cluster demok8scluster.k8s.local --state=s3://kops-abhi-storage  

# Check nodes
kubectl get nodes  
7. Cleanup (Avoid Unnecessary Costs!)
bash
Copy
kops delete cluster demok8scluster.k8s.local \
  --state=s3://kops-abhi-storage \
  --yes  

# Delete S3 bucket
aws s3 rb s3://kops-abhi-storage --force  
Troubleshooting Tips
Cluster Fails to Validate:

Check IAM permissions.

Ensure the S3 bucket exists and is accessible.

Nodes Not Ready:

Verify EC2 instances are running.

Check security groups for proper inbound/outbound rules.

KOPS Command Errors:

Ensure kops and kubectl versions are compatible.

Cost Optimization
Use Spot Instances: Add --node-spot-price and --master-spot-price to kops create cluster.

Terminate Clusters when not in use.

Avoid t2.micro for Production: Use t3.small or larger for reliability.

Why This Setup?
KOPS: Simplifies Kubernetes cluster lifecycle management on AWS.

S3 Bucket: Stores cluster state for recovery and updates.

t2.micro: Fits AWS Free Tier (but not recommended for production).

Next Steps:

Deploy a sample app: kubectl create deployment nginx --image=nginx

Explore KOPS Documentation for advanced configurations.

By following this guide, you’ve set up a cost-effective Kubernetes cluster on AWS – ideal for learning and experimentation! 🚀
