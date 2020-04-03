# eks-config

## Steps to install

1. Edit cluster.yaml to your liking for starting up a cluster.
2. Create the cluster
```sh
# create cluster
eksctl create cluster -f cluster.yaml
```
3. Setup the cluster as an IAM OIDC provider.
```sh
eksctl utils associate-iam-oidc-provider --region us-west-2 --cluster dev-cluster --approve
```
4. Create the policy if it doesn't already exist.
```sh
aws iam create-policy \
    --policy-name ALBIngressControllerIAMPolicy \
    --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/iam-policy.json
```
5. Use gitops to connect to deploy services and connect repo to your new repo name
```sh
EKSCTL_EXPERIMENTAL=true eksctl enable profile --cluster <YOUR_CLUSTER_NAME> --region <REGION> --git-url git@github.com:bdbmammoth/eks-config --git-email <YOUR_EMAIL> git@github.com:<YOUR_ORG>/<NEW_REPO>
```
6. From the output of the above, copy the ssh token from above and add as a deploy token to github and give write access.

7. attach the policy to the ALB
```sh
eksctl create iamserviceaccount \
    --region us-west-2 \
    --name alb-ingress-controller \
    --namespace kube-system \
    --cluster dev-cluster \
    --attach-policy-arn arn:aws:iam::981873564135:policy/ALBIngressControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
```

## Connecting to grafana
```sh
kubectl -n monitoring port-forward [grafana-pod-name] 3000

# goto port 3000 on localhost
# password default is `prom-operator`
```