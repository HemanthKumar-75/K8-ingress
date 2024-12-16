# Ingress Controller

## 1. Associate IAM OIDC Provider with the EKS Cluster
```
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster expense-cluster \
    --approve
```
## 2. Download the IAM Policy for the AWS Load Balancer Controller
```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.10.0/docs/install/iam_policy.json
```
### 3. Create the IAM Policy in AWS
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```
4. Create an IAM Service Account for the Controller
```
eksctl create iamserviceaccount \
--cluster=expense-cluster \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::248189912713:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve
```
5. Add Helm Chart Repository
```
helm repo add eks https://aws.github.io/eks-charts
```
6. Install AWS Load Balancer Controller Using Helm
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=expense-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```