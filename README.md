# prometheus-community-helm-stack
A repository that contains how to deploy the prometheus community helm stack on an eks kubernetes cluster. 

## Create an eks cluster

eksctl create cluster --name=observability \
                      --region=eu-west-3 \
                      --zones=eu-west-3a,eu-west-3b \
                      --without-nodegroup


eksctl utils associate-iam-oidc-provider \
    --region eu-west-3 \
    --cluster observability \
    --approve

## create a node group
eksctl create nodegroup --cluster=observability \
                        --region=eu-west-3 \
                        --name=observability-ng-private \
                        --node-type=t3.medium \
                        --nodes-min=3 \
                        --nodes-max=5 \
                        --node-volume-size=20 \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
                        --node-private-networking


## Update ./kube/config file
aws eks update-kubeconfig --name observability --region eu-west-3

## add the help repo for prometheus community
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

kubectl create ns monitoring

## install the helm chart
helm install monitoring prometheus-community/kube-prometheus-stack \
-n monitoring \
-f ./custom_kube_prometheus_stack.yml

kubectl get all -n monitoring

## port forward the required services to view them on your browser
kubectl port-forward service/prometheus-operated -n monitoring 9090:9090 --address 0.0.0.0

kubectl port-forward service/monitoring-grafana -n monitoring 8080:80 --address 0.0.0.0 

grafana:
username: admin
password: prom-operator

kubectl port-forward service/alertmanager-operated -n monitoring 9093:9093 --address 0.0.0.0


Configure alert manager
username
password: use base64 encode of your gmail app password

To encode your gmail App password, run this command: 
echo -n 'your gmail app password' | base64

## To deploy the application: 


## create a namespace named dev
kubectl create ns dev

## use kustomize to create the deployment files:

kubectl apply -k kubernetes-manifest/

kubectl get all -n dev 

test some of the endpoints: /healthy, /logs, /example

## Deploy the alert manager config files  using Kustomize

kubectl apply -k alerts-alertmanager-servicemonitor-manifest/


Wait for 4-5 minutes and then check the Prometheus UI to confirm that the custom metrics implemented in the Node.js application are available:
http_requests_total: counter
http_request_duration_seconds: histogram
http_request_duration_summary_seconds: summary
node_gauge_example: gauge for tracking async task duration


