# Deploying an E Commerce Three Tier application on AWS EKS | 8 Services and 2 Databases

Stan's Robot Shop is a sample microservice application you can use as a sandbox to test and learn containerized application orchestration and monitoring techniques.


## Table of Contents

- [Deploying an E Commerce Three Tier application on AWS EKS | 8 Services and 2 Databases](#deploying-an-e-commerce-three-tier-application-on-aws-eks--8-services-and-2-databases)
  - [Table of Contents](#table-of-contents)
    - [Prerequisites](#prerequisites)
    - [Getting Started](#getting-started)
    - [Endpoints](#endpoints)
    - [Credits](#credits)
  - [Contributing](#contributing)
    - [Happy Learning!](#happy-learning)

### Prerequisites

- [x] Active Terminal
- [x] Installed AWS CLI and Kubectl -(see instructions below)
- [x] AInstalled eksctl  -(see instructions below)

### Getting Started

1. Install AWS CLI - https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html
```
aws
```
2. Install kubectl - https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html
```
kubectl version
```
3. Install ekcctl - https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-config
```
eksctl version
```

1. Clone the repository:
   git clone https://github.com/okeymcokoli/e-commerce-webapp.git

2. Follow the steps below to deploy application:
    a. Install AWS EKS
    ```
    eksctl create cluster --name <cluster-name> --region <preferred-region>
    ```
    b. Configure IAM OIDC Provider.
    ```
    export cluster_name=<CLUSTER-NAME>
    ```
    ```
    oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5) 
    ```
    c. Check if there is an IAM OIDC provider configured already and install if not:
    ```
    aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
    ```
    ```
    eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
    ```
3. Set up ALB add on
   a. Download IAM policy
   ```
   curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
   ``` 

   b. Create IAM Policy
   ```
   aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy \ --policy-document file://iam_policy.json
    ```

    c. Create IAM Role
    ```
    eksctl create iamserviceaccount --cluster=<your-cluster-name> --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy --approve
    ```

4. Deploy ALB controller
   a. Add helm repo
   ```
   helm repo add eks https://aws.github.io/eks-charts
   ``` 

   b. Update the repo
   ```
   helm repo update eks
   ```

   c. Proceed with installation:
   ```
   helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
    --set clusterName=<your-cluster-name> \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set region=<region> \
    --set vpcId=<your-vpc-id>
    ```
    d. Verify that the deployment is successful
    ```
    kubectl get deployment -n kube-system aws-load-balancer-controller
    ```

5. Configure EBS CSI Plugin.
   ```
   eksctl create iamserviceaccount --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster <YOUR-CLUSTER-NAME> \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
    ```

    ```
    eksctl create addon --name aws-ebs-csi-driver --cluster <YOUR-CLUSTER-NAME> --service-account-role-arn arn:aws:iam::<AWS-ACCOUNT-ID>:role/AmazonEKS_EBS_CSI_DriverRole --force
    ```

6. Install the application
    a. create a helm namespace
   ```
   kubectl create ns <YOUR-NAMESPACE-NAME>
   ``` 
   
    b. cd into the helm folder and Install application using helm
   ```
   helm install <app-name> --namespace robot-shop .
   ```

    c. verify pods are up and running 
    ```
    kubectl get pods -n <YOUR NAMESPACE-NAME>
    ```

    d. Expose the application using ingress. cd .. <INGRESS.YAML FOLDER> 
    ```
    kubectl apply -f ingress.yaml
    ```

    This should create ingress for you and provide a DNS for the load balancer for accessing the application externally.

7.  Verify all ALB status is Active on AWS UI or copy the address from running cmd
   
   ```
   kubectl get ingress -n <YOUR NAMESPACE-NAME>
   ``` 



### Endpoints
8. Paste the address on your browser and access the application.


### Credits
9. Big shoutout to Instana/robot-shop and Abhishek.Veeramalla whose insights and codes were curled for this project

## Contributing
Contributions are welcome! Feel free to open issues, submit pull requests, or provide suggestions to improve this project.

### Happy Learning!