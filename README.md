## To create an IAM OIDC identity provider for your cluster with eksctl
Determine whether you have an existing IAM OIDC provider for your cluster.
Retrieve your cluster's OIDC provider ID and store it in a variable.
```
 oidc_id=$(aws eks describe-cluster --name mynode --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
```
## Determine whether an IAM OIDC provider with your cluster's ID is already in your account.

```
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```

## Create an IAM OIDC identity provider for your cluster with the following command. Replace my-cluster with your own value
```
eksctl utils associate-iam-oidc-provider --cluster mynode --approve
```
## Install EBS driver:
```
eksctl create iamserviceaccount \
  --name ebs-csi-controller-sa \
  --namespace kube-system \
  --cluster mynode \
  --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --approve \
  --role-only \
  --role-name AmazonEKS_EBS_CSI_DriverRole

  ```
## To create ebs addon:
```
aws eks create-addon --cluster-name mynode --addon-name aws-ebs-csi-driver \
  --service-account-role-arn arn:aws:iam::670053109278:role/AmazonEKS_EBS_CSI_DriverRole
```
## Install Helm:
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```
## Export path:
```
export PATH=$PATH:/usr/local/bin
echo $PATH
```
## Add prometheus Helm repo
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
# Add grafana Helm repo
```
helm repo add grafana https://grafana.github.io/helm-charts
```
## To see helm repos:
```
helm repo list
```
## Create a namespace 
```
kubectl create namespace prometheus
```
## To install prometheus
```
helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
```
## To verify pods in promethues namespace:
```
kubectl get pods --namespace prometheus
```
```
kubectl get daemonset --namespace prometheus
```
## To port forward:
```
kubectl port-forward -n prometheus deploy/prometheus-server 8080:9090
```

## To change clusterip to load balancer: change ClusterIP to LoadBalancer
```
kubectl edit svc prometheus-server --namespace prometheus
```
## To verify:
```
kubectl get deployment --namespace prometheus

kubectl get svc --namespace prometheus

```

# Grafana
```
mkdir environment
mkdir ${HOME}/environment/grafana
```
```
cat << EoF > ${HOME}/environment/grafana/grafana.yaml
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.prometheus.svc.cluster.local
      access: proxy
      isDefault: true
EoF
```
```
kubectl create namespace grafana
```
## Deploy with username admin password EKS!sAWSome
```
helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='EKS!sAWSome' \
    --values ${HOME}/environment/grafana/grafana.yaml \
    --set service.type=LoadBalancer
```
## Run the following command to check if Grafana is deployed properly:
```
kubectl get all -n grafana
```

## Cluster Monitoring Dashboard : For creating a dashboard to monitor the cluster:
```
Click '+' button on left panel and select ‘Import’.
Enter 3119 dashboard id under Grafana.com Dashboard.
Click ‘Load’.
Select ‘Prometheus’ as the endpoint under prometheus data sources drop down.
Click ‘Import’
```
## Pods Monitoring Dashboard : For creating a dashboard to monitor all the pods:

```
Click '+' button on left panel and select ‘Import’.
Enter 6417 dashboard id under Grafana.com Dashboard.
Click ‘Load’.
Enter Kubernetes Pods Monitoring as the Dashboard name.
Click change to set the Unique identifier (uid).
Select ‘Prometheus’ as the endpoint under prometheus data sources drop down.s
Click ‘Import’.
```
