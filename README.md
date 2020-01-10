# cilcalator
Cilcalator

# Crear un cluster (vía eksctl):
# https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html

eksctl create cluster \
--name prod \
--version 1.14 \
--region us-west-2 \
--nodegroup-name standard-workers \
--node-type t3.medium \
--nodes 3 \
--nodes-min 1 \
--nodes-max 4 \
--ssh-access \
--ssh-public-key my-public-key.pub \
--managed

# Crear ALB-Ingress-Controller
# https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html

eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster prod \
    --approve

aws iam create-policy \
    --policy-name ALBIngressControllerIAMPolicy \
    --policy-document https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/iam-policy.json

kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/rbac-role.yaml

eksctl create iamserviceaccount \
    --region us-east-1 \
    --name alb-ingress-controller \
    --namespace kube-system \
    --cluster prod \
    --attach-policy-arn arn:aws:iam::111122223333:policy/ALBIngressControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve

kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.1.4/docs/examples/alb-ingress-controller.yaml

kubectl edit deployment.apps/alb-ingress-controller -n kube-system

    spec:
      containers:
      - args:
        - --ingress-class=alb
        - --cluster-name=prod
        - --aws-vpc-id=vpc-03468a8157edca5bd
        - --aws-region=us-east-1

# Crear un Ingress para el proycto
kubectl apply -f .k8s/cilcalator-ingress.yml

# Revisar el ingress creado
kubectl get ingress
kubectl describe ingress

#
kubectl describe service test-eksctl-ecr-master--web | grep Ingress
