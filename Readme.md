# Install Basic Scaling Cluster K8S

### Check pods on cluster

Run 

```bash
$ kubectl get pods --all-namespaces
```

Then return this

```bash
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
kube-system   aws-node-f45pg             1/1     Running   0          xxm
kube-system   coredns-86d9946576-2h2zk   1/1     Running   0          xxm
kube-system   coredns-86d9946576-4cvgk   1/1     Running   0          xxm
kube-system   kube-proxy-864g6           1/1     Running   0          xxm
```

```bash
$ kubectl create namespace kube-system
```

## Install Metrics Server

Then run the below command to create all of the resources, in kube-system/metrics-server

```bash
$ cd resources/kube-system/metrics-server
```

```bash
$ kubectl apply -f . --namespace kube-system
```

Verify deployment metrics server

```bash
$ kubectl get pods --all-namespaces
```

Use kubectl to see CPU and memory metrics:

```bash
$ kubectl top pods -n kube-system
```

## Install Cluster AutoScaler

### Prerequisites for Cluster Autoscaler
In AutoscalingGroup add this tags on lauchConfiguration for nodes created with values for discovery

```
k8s.io/cluster-autoscaler/enabled
k8s.io/cluster-autoscaler/MeuClusterName
```

### Create IAM OIDC provider
IAM OIDC is used for authorizing the Cluster Autoscaler to launch or terminate instances under an Auto Scaling group.

Execute this steps on console.
1 - In the EKS cluster console, navigate to the configuration tab and copy the OpenID connect URL.
2 - Then, go to the IAM console, and select Identity provider.
3 - Click "Add provider" select "OpenID Connect" and click "Get thumbprint"
4 - Then enter the "Audience" (sts.amazonaws.com in our example pointing to the AWS STS, also known as the Security Token Service) and add the provider.
5 - Click "Add provider" in the AWS Console.

### Create IAM policy for Cluster Autoscaler

To create the policy with the necessary permissions if do not exists 

AmazonEKSClusterAutoscalerPolicy.json
```
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Action": [
              "autoscaling:DescribeAutoScalingGroups",
              "autoscaling:DescribeAutoScalingInstances",
              "autoscaling:DescribeLaunchConfigurations",
              "autoscaling:DescribeTags",
              "autoscaling:SetDesiredCapacity",
              "autoscaling:TerminateInstanceInAutoScalingGroup",
              "ec2:DescribeLaunchTemplateVersions"
          ],
          "Resource": "*",
          "Effect": "Allow"
      }
  ]
} 
```
### Create IAM role for Cluster Autoscaler

Now create role for EKS in console will create a Role and select web Identity. And choose OIDC provider created.

Then, Iam Role and make sure the policy is attached.

And edit the "Trust relationships." and add cluster permission next, change the OIDC and click “Update Trust Policy” to save it.


### Deploy Kubernetes Cluster Autoscaler

Then run the below command to create all of the resources, in kube-system/metrics-server

```bash
$ cd resources/kube-system/cluster-autoscaler
```

```bash
$ kubectl apply -f . --namespace kube-system
```

Verify deployment cluster auto scaler

```bash
$ kubectl get pods --all-namespaces | grep cluster-autoscaler
```

Next, verify the logs by issuing this command

```bash
$ kubectl logs -l app=cluster-autoscaler -n kubesystem -f
```

## Install Kube State Metrics

Then run the below command to create all of the resources, in kube-system/kube-state-metrics

```bash
$ cd resources/kube-system/kube-state-metrics
```

```bash
$ kubectl apply -f . --namespace kube-system
```

Verify deployment Kube state metrics

```bash
$ kubectl get pods --all-namespaces | grep kube-state-metrics
```

Kube state metrics service exposes all the metrics on `/metrics`. 
We will access this service using port-forward for a quick check. 
Once you port-forward the service, use any browser to access the port.

```bash
$ kubectl port-forward svc/kube-state-metrics 30135:8080 -n kube-system
```

Click on `/metrics`

## Deploy Application on Cluster
Before to create application set context and namespace with commands:

```bash
$ kubectl create namespace production
```

```bash
$ kubectl config set-context MeuCluster --cluster=MeuCluster --namespace=production
```

Then to deploy application go to resources, in app-test.

```bash
$ kubectl config view | grep namespace
```

```bash
$ cd resources/app-test/
```

```bash
$ kubectl apply -f . --namespace=production
```

To view the service information
```bash
$ kubectl get svc
```

Check deployment status
```bash
$ kubectl get deploy
```

Use this command for start a container and send an infinite loop of queries to the ‘php-apache’ service, listening on port 8080.
```bash
$ kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://hpa-demo-deployment; done"
```

Check status scaling container
```bash
$ kubectl get hpa -w
```
