# easy-kubernetes-nginx-php
Single node cluster to host php application with minimum effort

## Kubernetes Installation
Installion instruction for 
[kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) and 
[minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)


## Configmap
We will use configmap api to create nginx configuration. The configMap resource provides a way to inject configuration data into Pods. The data stored in a ConfigMap object can be referenced in a volume of type configMap and then consumed by containerized applications running in a Pod.

```
$ kubectl create -f configmap.yaml
configmap/nginx-config created
```

Checking configmaps
```
$ kubectl get configmaps
NAME           DATA   AGE
nginx-config   1      53s
```


## Deployment
One of the most common Kubernetes object is the deployment object. The deployment object defines the container spec required, along with the name and labels used by other parts of Kubernetes to discover and connect to the application.
```
$ kubectl create -f deployment.yaml
deployment.extensions/my-app created
```

Checking deployment and service
```
$ kubectl get deployments
NAME     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
my-app   1         1         1            1           24s
$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
my-app-77445b4858-dprqb   2/2     Running   0          30s
```


## Service
Kubernetes has powerful networking capabilities that control how applications communicate. Service is assigned a unique IP address (also called clusterIP). 

```
$ kubectl create -f service.yaml
service/my-app-service created
```

The Service selects all applications with the label webapp1. As multiple replicas, or instances, are deployed, they will be automatically load balanced based on this common label. The Service makes the application available via a NodePort.

Checking services
```
$ kubectl get services
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP        5d
my-app-service   NodePort    10.99.232.63   <none>        80:30080/TCP   26s
```


Now we should be able to curl the nginx Service on <CLUSTER-IP>:<PORT>
```
$ curl localhost:30080/index.php
This is my php application
```


## Scalling
We can scale up to 3 replicas from 1 modifiying deployment.yaml replicas field as "replicas: 3" and running the following kubectl command
```
$ kubectl apply -f deployment.yaml
```

Lets check deployment and pods
```
$ kubectl get deployments
NAME     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
my-app   3         3         3            3           1h
$ kubectl get pods
NAME                      READY   STATUS    RESTARTS   AGE
my-app-77445b4858-74rlw   2/2     Running   0          11s
my-app-77445b4858-dprqb   2/2     Running   0          1h
my-app-77445b4858-pszr4   2/2     Running   0          11s
```
