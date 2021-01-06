## wordpress-complete-example

This may be considered an extension on the previous example '2' . In here we are deploying the blog and database to be ready for a fresh configuration, but with the main concept of using and ingress controller. This way we are exposing the blog service outside of the cluster with an ingress resource. 
For our example, we publish `localhost` and a dns `wordpress.foo.bar.com` with same result. Notice in here we are not using NodePort, but the nginx-controller is using 2 services, ClusterIP and LoadBalancer to handle the ingress requests. There are quite a few ingress controllers though you can find over internet, we will be using [nginx-ingress](https://kubernetes.github.io/ingress-nginx/).


#### Create a host record

For the example of the dns `wordpress.foo.bar.com`, we will be required to create a host entry locally to point to the IP address of the machine:
- Linux: ```/etc/hosts ```
- Windows: ```C:/System32/drivers/etc/hosts```
- Mac: ```/private/etc/hosts```

```bash
# DNS record to point to our cluster
{local_ip_address} wordpress.foo.bar.com
```

#### Create our custom Namespace
```bash
kubectl apply -f wordpress-complete-example/namespace.yaml
```

#### Deploy the nginx-ingress controller

We are using the default resources for the application which are deployed in an isolated namespace
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.43.0/deploy/static/provider/cloud/deploy.yaml
```

#### Deploy the backend database
```bash
kubectl apply -f wordpress-complete-example/app/backend-mariadb-deploy.yaml
```

#### Deploy the frontend blog
```bash
kubectl apply -f wordpress-complete-example/app/frontend-wordpress-deploy.yaml
```

#### Access to the blog

Now you can access to the blog by using one of the ingress hosts examples and set the blog up:
- example 1: [http://localhost](http://localhost)
- example 2: [http://wordpress.foo.far.com](http://wordpress.foo.bar.com)

![Setting up Wordpress!](https://github.com/enriqueperezpi/kubernetes-examples/blob/gh-pages/wordpress-installation.png "Setting up Wordpress")