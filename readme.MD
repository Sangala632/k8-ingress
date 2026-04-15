# AWS Load Balancer Controller (Ingress Controller) Setup

This guide walks you through the prerequisites and IAM setup required before installing the AWS Load Balancer Controller on an Amazon EKS cluster.

## Prerequisites

Set the following environment variables for your AWS environment before running the setup commands:

```bash
export REGION_CODE="us-east-1"
export CLUSTER_NAME="roboshop-dev"
export ACC_ID="838180513114"
```

## Setup Steps

### 1. Associate OIDC Provider
First, you need to associate the IAM OIDC provider with your EKS cluster to allow the cluster to authenticate with IAM:

```bash
eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve
```

### 2. Download IAM Policy
Download the recommended IAM policy document for the AWS Load Balancer Controller:

```bash
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v3.2.1/docs/install/iam_policy.json
```

### 3. Create IAM Policy
Create the IAM policy in your AWS account using the document you just downloaded:

```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

### 4. Create IAM Service Account
Create an IAM role, bind it to a Kubernetes `ServiceAccount` in the `kube-system` namespace, and attach the policy you created:

```bash
eksctl create iamserviceaccount \
    --cluster=$CLUSTER_NAME \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --attach-policy-arn=arn:aws:iam::${ACC_ID}:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --region $REGION_CODE \
    --approve
```
### 5. Install Load Balancer Controller
Add the Amazon EKS Helm chart repository and install the AWS Load Balancer Controller using Helm:

```bash
# Add and update the EKS chart repository
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks

# Install the AWS Load Balancer Controller
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=$CLUSTER_NAME \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller
```

