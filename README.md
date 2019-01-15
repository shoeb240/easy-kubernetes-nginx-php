# easy-kubernetes-nginx-php
Single node cluster to host php application with minimum effort

## Kubernetes Installation
Installion instruction for 
[kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) and 
[minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

Start minikube:
```
minikube start
```

To install and run Kubernetes on Mac follow [this](https://rominirani.com/tutorial-getting-started-with-kubernetes-with-docker-on-mac-7f58467203fd)

## Configmap
We will use configmap object to create nginx configuration. The configMap resource provides a way to inject configuration data into Pods. The data stored in a ConfigMap object can be referenced in a volume of type configMap and then consumed by containerized applications running in a Pod.

[configmap.yaml](https://github.com/shoeb240/easy-kubernetes-nginx-php/blob/master/configmap.yaml)
```
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-config
data:
  nginx.conf: |
    events {
    }
    http {
      server {
        listen 80 default_server;
        listen [::]:80 default_server;
        
        # Set nginx to serve files from the shared volume!
        root /public_html;
        server_name _;
        location / {
          try_files $uri $uri/ =404;
        }
        location ~ \.php$ {
          #include fastcgi_params;
          fastcgi_param REQUEST_METHOD $request_method;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
          fastcgi_pass 127.0.0.1:9000;

          fastcgi_param   QUERY_STRING            $query_string;
            
          fastcgi_param   REQUEST_METHOD          $request_method;
          fastcgi_param   CONTENT_TYPE            $content_type;
          fastcgi_param   CONTENT_LENGTH          $content_length;

          fastcgi_param   SCRIPT_FILENAME         $document_root$fastcgi_script_name;
          fastcgi_param   SCRIPT_NAME             $fastcgi_script_name;
          fastcgi_param   PATH_INFO               $fastcgi_path_info;
          fastcgi_param   PATH_TRANSLATED         $document_root$fastcgi_path_info;
          fastcgi_param   REQUEST_URI             $request_uri;
          fastcgi_param   DOCUMENT_URI            $document_uri;
          fastcgi_param   DOCUMENT_ROOT           $document_root;
          fastcgi_param   SERVER_PROTOCOL         $server_protocol;

          fastcgi_param   GATEWAY_INTERFACE       CGI/1.1;
          fastcgi_param   SERVER_SOFTWARE         nginx/$nginx_version;

          fastcgi_param   REMOTE_ADDR             $remote_addr;
          fastcgi_param   REMOTE_PORT             $remote_port;
          fastcgi_param   SERVER_ADDR             $server_addr;
          fastcgi_param   SERVER_PORT             $server_port;
          fastcgi_param   SERVER_NAME             $server_name;

          fastcgi_param   HTTPS                   $https;

          # PHP only, required if PHP was built with --enable-force-cgi-redirect
          fastcgi_param   REDIRECT_STATUS         200;
        }
      }
    }
```

Run the follwoing kubectl command:
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

We will create two container for nginx and php in one pod. They share the same hostPath volume /public_html.

[deployment.yaml](https://github.com/shoeb240/easy-kubernetes-nginx-php/blob/master/deployment.yaml)
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: my-app
  labels:
        app: my-app

spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: my-app

    spec:
      containers:
      - image: nginx:1.7.9
        name: my-nginx
        volumeMounts:
          - name: my-host-path
            mountPath: /public_html
          - name: nginx-config-volume
            mountPath: /etc/nginx
          
      - image: php:7.2-fpm
        name: app-php
        volumeMounts:
          - name: my-host-path
            mountPath: /public_html

      volumes:
      - name: nginx-config-volume
        configMap:
          name: nginx-config
      - name: my-host-path
        hostPath:
          path: /Users/shoeb240/easy-kubernetes-nginx-php/public_html
          type: Directory
```
*Note: You must change spec.template.spec.volumes.hostPath.path according to your codebase absolute path.

Run kubectl command:
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

[service.yaml](https://github.com/shoeb240/easy-kubernetes-nginx-php/blob/master/service.yaml)
```
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
  labels:
    app: my-app
spec:
  type: NodePort
  ports:
  - port: 80
    nodePort: 30080
  selector:
    app: my-app
```

Run kubectl command:
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
