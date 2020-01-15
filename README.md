# cilcalator
Cilcalator

# Crear un cluster (vía eksctl):
# https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html

eksctl create cluster \
--name test-cilcalator \
--version 1.14 \
--region us-east-1 \
--zones us-east-1a,us-east-1b \
--nodegroup-name cilcalator-workers \
--node-type t3.small \
--nodes 1 \
--nodes-min 1 \
--nodes-max 4 \
--ssh-access \
--ssh-public-key eks-shevia \
--managed

# Crear ALB-Ingress-Controller
# https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html

eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster test-cilcalator \
    --approve

aws iam create-policy \
    --policy-name ALBIngressControllerIAMPolicy \
    --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/iam-policy.json

kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/rbac-role.yaml

eksctl create iamserviceaccount \
    --region us-east-1 \
    --name alb-ingress-controller \
    --namespace kube-system \
    --cluster test-cilcalator \
    --attach-policy-arn arn:aws:iam::415538714327:policy/ALBIngressControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve

kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/alb-ingress-controller.yaml

kubectl edit deployment.apps/alb-ingress-controller -n kube-system

    spec:
      containers:
      - args:
        - --ingress-class=alb
        - --cluster-name=test-cilcalator
        - --aws-vpc-id=vpc-04bbf79b954792ab7
        - --aws-region=us-east-1

# Problemas con ingress sin ADDRESS
https://stackoverflow.com/questions/51511547/empty-address-kubernetes-ingress

# Crear un Ingress para el proycto
kubectl apply -f .k8s/cilcalator-ingress.yml

# Revisar el ingress creado
kubectl get ingress
kubectl describe ingress

#
kubectl describe service test-eksctl-ecr-master--web | grep Ingress


#Borrar Cluster
eksctl delete cluster --name=test-cilcalator
