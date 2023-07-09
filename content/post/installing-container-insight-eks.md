+++
author = "Atif Ramzan"
title = "How to install aws container insights for EKS"
date = "2023-05-24"
description = "In this post we will be installing aws container insights for eks"
categories = [
    "aws"
]
thumbnail = "images/container-insights-1.png"
shareImage = "images/container-insights-1.png"
featureImage = "images/container-insights-1.png"

+++

In this post we will be installing aws container insights for eks. We are assuming that you have already running EKS cluster.

## 1. Attaching Policy to EKS NodeGroup IAM Role
For this first we need to attach an policy to our nodegroup iam role `CloudWatchAgentServerPolicy`

## 2. Installing AWS Container Insights for EKS
Install the container insights using the below commands
```bash
export AWS_REGION=us-east-2
export CLUSTER_NAME=<your-cluster-name>

curl -s https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluentd-quickstart.yaml | sed "s/{{cluster_name}}/${CLUSTER_NAME}/;s/{{region_name}}/${AWS_REGION}/" | kubectl apply -f -
```
It will create some resources while its execution
```
namespace/amazon-cloudwatch created
serviceaccount/cloudwatch-agent created
clusterrole.rbac.authorization.k8s.io/cloudwatch-agent-role created
clusterrolebinding.rbac.authorization.k8s.io/cloudwatch-agent-role-binding created
configmap/cwagentconfig created
daemonset.apps/cloudwatch-agent created
configmap/cluster-info created
serviceaccount/fluentd created
clusterrole.rbac.authorization.k8s.io/fluentd-role created
clusterrolebinding.rbac.authorization.k8s.io/fluentd-role-binding created
configmap/fluentd-config created
daemonset.apps/fluentd-cloudwatch created
```
For verification use the below command
```
$ kubectl -n amazon-cloudwatch get daemonsets
NAME                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
cloudwatch-agent     1         1         1       1            1           kubernetes.io/os=linux   54m
fluentd-cloudwatch   1         1         1       1            1           <none>                   54m
```
{{% notice note "Important Note" %}}
Now the container insights will show you nothing at first, you need to configure and deploy an application on eks cluster to see the results
{{% /notice %}}

## 3. Deploying an Application using HELM
We will be installing wordpress application using helm.
Create a namespace for wordpress
```
$ kubectl create namespace wordpress
```

Add the repository for Bitnami Helm Charts.
```
$ helm repo add bitnami https://charts.bitnami.com/bitnami
```
Deploy wordpress in its own namespace.

```
$ helm -n wordpress install wordpress-app bitnami/wordpress
```
To verify the deployment is successfull or not use the below command.

```
$ kubectl -n wordpress-cwi rollout status deployment wordpress-app
```
Once the above command is successfull it will show the below output

```
Waiting for deployment "wordpress-app" rollout to finish: 0 of 1 updated replicas are available...
deployment "understood-zebu-wordpress" successfully rolled out
```

## 4. Get the URL to Access the Application
Use the below command to access the URL
```
$ export SERVICE_URL=$(kubectl get svc -n wordpress wordpress-app --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
$ echo "Public URL: http://$SERVICE_URL/"
```
To access the wordpress admin interface use the below command

```
$ export ADMIN_URL="http://$SERVICE_URL/admin"
$ export ADMIN_PASSWORD=$(kubectl get secret --namespace wordpress-cwi understood-zebu-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)

$ echo "Admin URL: http://$SERVICE_URL/admin
Username: user
Password: $ADMIN_PASSWORD
"
```
Once you logged with this user you container insights metrics will be enabled. Just go to Cloudwatch 
- Goto Cloudwatch --> `Insights` 
- Then `Container Insights` 
- Select `Performance Monitoring` from drop down.
- Select `EKS Clusters`

![Jane Doe](../images/container-insights-1.png)
