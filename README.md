# easy-kubernetes-nginx-php
Single node cluster to host php application with minimum effort

##Configmap
We will use configmap api to create nginx configuration.

configmap.yaml
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
Kubernetes has powerful networking capabilities that control how applications communicate. 

```
$ kubectl create -f service.yaml
service/my-app-service created
```

Checking services
```
$ kubectl get services
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1      <none>        443/TCP        5d
my-app-service   NodePort    10.99.232.63   <none>        80:30080/TCP   26s
```


Now we can access our app
```
$ curl localhost:30080/index.php
This is my php application
```
