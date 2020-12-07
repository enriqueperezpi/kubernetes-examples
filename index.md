## Kubernetes training

Welcome to this github page built to document training examples for kubernetes.


### frontend-backend-example-1

This is a very simple resource configuration that deploys a frontend and backend services running in the cluster. Communication between the services which uses type `NodePort` and `ClusterIP` to show the functionality and differences. We are deploying into namespace `app`. You may choose an existing one or remove namespace from metadata to deploy into default namespace.

#### Deploy backend

```bash
kubectl create -f backend.yaml
```
frontend.yaml
```markdown
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: apps
  name: backend
  labels:
    app: backend-app
spec:
  replicas: 4
  selector:
    matchLabels:
      app: backend-app
  template:
    metadata:
      labels:
        app: backend-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  namespace: apps
  name: backend
spec:
  selector:
    app: backend-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

#### Deploy Frontend

```bash
kubectl create -f frontend.yaml
```
frontend.yaml
```markdown
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: apps
  name: frontend
  labels:
    app: frontend-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend-app
  template:
    metadata:
      labels:
        app: frontend-app
    spec:
      containers:
      - name: frontend
        image: nginx:1.14.2
        volumeMounts:
        - mountPath: /etc/nginx # mount nginx-conf volumn to /etc/nginx
          readOnly: true
          name: nginx-conf
      volumes:
      - name: nginx-conf
        configMap:
          name: frontend-nginx-conf # place ConfigMap `nginx-conf` on /etc/nginx
          items:
            - key: nginx.conf
              path: nginx.conf
            - key: virtualhost.conf
              path: conf.d/virtualhost.conf # dig directory
---
apiVersion: v1
kind: Service
metadata:
  namespace: apps
  name: frontend
spec:
  type: NodePort
  selector:
    app: frontend-app
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
---
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: apps
  name: frontend-nginx-conf
data:
  nginx.conf: |
    user  nginx;
    worker_processes  1;

    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;


    events {
        worker_connections  1024;
    }


    http {
        #include /etc/nginx/mime.types;
        default_type  application/octet-stream;

        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';

        access_log  /var/log/nginx/access.log  main;
        sendfile        on;
        keepalive_timeout  65;
        include /etc/nginx/conf.d/virtualhost.conf;
    }

  virtualhost.conf: |
    upstream app {
      server backend:80;
      keepalive 1024;
    }
    server {
      listen 80 default_server;
      root /usr/local/app;
      access_log /var/log/nginx/app.access_log main;
      error_log /var/log/nginx/app.error_log;
      location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
      }
    }
```

The frontend service is exposing a physical port in the kubernetes worker nodes. If not specified this will be a random port between 30000 and 32767.

You can call to the ipaddress of the worker node with the port , frontend application will receive the request and will pass it to the backend service.