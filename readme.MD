# Ingress Controller

* Ingress controller is used to provide external access to the applications running in Kubernetes. we can set the routing rules through ingress resource either path based or host based. in EKS ingress resource can create ALB, Listener Role & Target group. Ingress is attached to service, so it fetch the pods & them to target group.

```
REGION_CODE=us-east-1
CLUSTER_NAME=expense
ACC_ID=825765392883
```

### Permissions

* OIDC provider (Authentication service to connect AWS and Expense Cluster)
```
eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve
```

* IAM Policy (EKS Cluster and ALB are different service, So EKS Cluster must be granted permission to create ALB)
```
curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.12.0/docs/install/iam_policy.json
```

* Create IAM Policy
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

* Provide access to EKS through IAM Policy
```
eksctl create iamserviceaccount \
--cluster=$CLUSTER_NAME \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::$ACC_ID:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region $REGION_CODE \
--approve
```

### Install Drivers

* Add the EKS chart repo to Helm
```
helm repo add eks https://aws.github.io/eks-charts
```

* Install AWS Load Balancer Controller
```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```