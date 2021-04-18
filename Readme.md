
eksctl.exe create cluster -f .\eks.cluster.yml

kubectl.exe apply -k .\aws-efs-csi-driver\deploy\kubernetes\overlays\stable\

Create EFS-SG opening NFS port 2049 -

```
# deploy EFS storage driver
kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=master"

# get VPC ID
aws eks describe-cluster --name getting-started-eks --query "cluster.resourcesVpcConfig.vpcId" --output text
# Get CIDR range
aws ec2 describe-vpcs --vpc-ids vpc-id --query "Vpcs[].CidrBlock" --output text

# security for our instances to access file storage
aws ec2 create-security-group --description efs-test-sg --group-name efs-sg --vpc-id VPC_ID
aws ec2 authorize-security-group-ingress --group-id sg-xxx  --protocol tcp --port 2049 --cidr VPC_CIDR

# create storage
aws efs create-file-system --creation-token eks-efs

# create mount point
aws efs create-mount-target --file-system-id FileSystemId --subnet-id SubnetID --security-group GroupID

# grab our volume handle to update our PV YAML
aws efs describe-file-systems --query "FileSystems[*].FileSystemId" --output text
```

kubectl create ns jenkins
kubectl.exe apply -n jenkins -f .\storage\
kubectl.exe apply -n jenkins -f .\deployment\

For Alb -

1. Create IAM OIDC PROVIDER -

   eksctl utils associate-iam-oidc-provider --cluster eks-jenkins --approve

   aws iam list-open-id-connect-providers

2. aws eks describe-cluster --name eks-jenkins --query "cluster.identity.oidc.issuer" --output text

3. aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://enableLB/iam-policy.json

    {
        "Policy": {
            "PolicyName": "AWSLoadBalancerControllerIAMPolicy",
            "PolicyId": "ANPAZICTBJJBJIT37EYUC",
            "Arn": "arn:aws:iam::635829307970:policy/AWSLoadBalancerControllerIAMPolicy",
            "Path": "/",
            "DefaultVersionId": "v1",
            "AttachmentCount": 0,
            "PermissionsBoundaryUsageCount": 0,
            "IsAttachable": true,
            "CreateDate": "2021-04-15T18:25:43+00:00",
            "UpdateDate": "2021-04-15T18:25:43+00:00"
        }
    }

4. eksctl create iamserviceaccount --cluster=eks-jenkins --namespace=kube-system --name=aws-load-balancer-controller --attach-policy-arn=arn:aws:iam::635829307970:policy/AWSLoadBalancerControllerIAMPolicy --override-existing-serviceaccounts --approve

5. Install the AWS Load Balancer Controller -

```sh
    kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.1.1/cert-manager.yaml

    # edit clustername in file
    kubectl apply -f .\enableLB\v2_1_3_full.yaml
```

6. verify controller -

   kubectl get deployment -n kube-system aws-load-balancer-controller

   NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
    aws-load-balancer-controller   1/1     1            1           36s

7. Tag subnets as mentioned in - https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html

8. kubectl.exe -n jenkins apply -f .\deployment\alb\jenkins.alb.yml

9. kubectl.exe get pods -n jenkins
    NAME                       READY   STATUS    RESTARTS   AGE
    jenkins-5958d987c7-vv5jv   1/1     Running   0          40m

10.  kubectl -n jenkins exec jenkins-5958d987c7-vv5jv --  cat /var/jenkins_home/secrets/initialAdminPassword
3047ef315c42406887e1217cd9c3fe11

Enable AutoScaler:

1.  aws iam create-policy --policy-name AmazonEKSClusterAutoscalerPolicy --policy-document file://enableAsg/asg-policy.json

2. eksctl create iamserviceaccount --cluster=eks-jenkins  --namespace=kube-system  --name=cluster-autoscaler  --attach-policy-arn=arn:aws:iam::635829307970:policy/AmazonEKSClusterAutoscalerPolicy  --override-existing-serviceaccounts  --approve

3.  update vesion and cluster name in file and deploy -
    kubectl.exe apply -f .\enableASG\cluster.autoscaler.yml

4. To view log -
    kubectl -n kube-system logs -f deployment.apps/cluster-autoscaler

Configure Jenkins Kubernetes Plugin for Agent:

1. kubectl cluster-info

2.
