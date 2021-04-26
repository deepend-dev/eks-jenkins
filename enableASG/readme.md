
# Enable AutoScaler
[link](https://docs.aws.amazon.com/eks/latest/userguide/cluster-autoscaler.html#ca-deployment-considerations)

1.  aws iam create-policy --policy-name AmazonEKSClusterAutoscalerPolicy --policy-document file://enableAsg/asg-policy.json

2. eksctl create iamserviceaccount --cluster=eks-jenkins  --namespace=kube-system  --name=cluster-autoscaler  --attach-policy-arn=arn:aws:iam::635829307970:policy/AmazonEKSClusterAutoscalerPolicy  --override-existing-serviceaccounts  --approve

3. kubectl.exe apply -f .\enableASG\cluster.autoscaler.yml

kubectl patch deployment cluster-autoscaler -n kube-system  -p '{"spec":{"template":{"metadata":{"annotations":{"cluster-autoscaler.kubernetes.io/safe-to-evict": "false"}}}}}'
4. To view log -
    kubectl -n kube-system logs -f deployment.apps/cluster-autoscaler
