# Prerequisites

#### setup commands
```
Install kubectl, eksctl, AWS CLI (provided below)
aws configure (have access key and secret access key)
eksctl create cluster --name <cluster-name> --region us-east-1 --fargate
---eksctl delete cluster --name <cluster-name> --region us-east-1
```
#### kubectl local setup
```
kubectl apply -f deploy.yaml
kubectl apply -f service.yaml
---update a kubeconfig for your cluster
aws eks update-kubeconfig --name <cluster-name> --region us-east-1
```
#### Create Fargate profile ---If wanted to change name space
```
eksctl create fargateprofile \
    --cluster <cluster-name> \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048
```
#### Deploy the deployment, service and Ingress
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
```

#### Commands
```
kubectl get all -n game-2048
kubectl get ingress -n game-2048
```
Ingress controller takes care of  creating load balancer for ingress-2048 (ie set up target group, port to forward)

#### configure IAM OIDC provider (pre req for ALB)
```
export cluster_name=<cluster-name>
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5) 

eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
To create ALB controller - these are pods which needs access to AWS service like ALB

#### Download IAM policy
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```

#### Create IAM Policy
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

#### Create IAM Role
```
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

#### Deploy ALB
```
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
  
kubectl get deployment -n kube-system aws-load-balancer-controller -w
kubectl get po -n kube-system
```
Check ALB is created in UI (DNS name accessible)
kubectl get ingress -n game-2048


## KUBECTL (Linux):

### Download the latest release with the command
```
 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
### Download the kubectl checksum file:
```
 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```
### Validate the kubectl binary against the checksum file:
```
 echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check   ### ### ### Valid output - kubectl: OK
```
### Install kubectl
```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
--If not root access(sudo)
```
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
```
### version you installed is up-to-date
```
kubectl version --client
```
## EKSCTL (Linux):

### for ARM systems
```
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
```
### Verify checksum
```
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
```
### Extract zip file and move file
```
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
```
### version
```
eksctl
```

## AWS CLI(Linux):

### Download the installation file
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```
### Unzip the installer
```
unzip awscliv2.zip
```
### Run the install program
```
sudo ./aws/install
```
--Without sudo
```
./aws/install -i /usr/local/aws-cli -b /usr/local/bin
```
### Verify path and version
```
which aws
ls -l /usr/local/bin/aws
aws --version
```