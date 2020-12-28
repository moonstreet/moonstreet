---
title: "Install the Nginx ingress controller on K3s and deploy a web app"
date: 2020-12-26T16:33:46+01:00
draft: false
---

When running K3s, by default Traefik is installed as an ingress controller. 
You need an ingress controller to expose (web) applications to the outside world.
I am however more comfortable with the Nginx ingress controller so let's just install that instead.

In this post I will first install K3s, then install the Nginx ingress controller. 
Finally I will deploy a little go application (which is going to be fabulous later).

Disclaimer: I am currently studying operators and CRD's so this setup is for testing them locally with a simple one node cluster.


```shell
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644 --disable traefik
```

Wait a bit. When done, we can copy the config file so we can use kubectl.

```shell
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```

Now we are ready to install Nginx ingress:

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/deploy.yaml
```

With running `kubectl get pods --all-namespaces -w` we can check the progress

```shell
[vagrant@centos02 ~]$ kubectl get pods --all-namespaces -w
NAMESPACE       NAME                                        READY   STATUS              RESTARTS   AGE
kube-system     coredns-66c464876b-2f5gq                    1/1     Running             0          21m
kube-system     local-path-provisioner-7ff9579c6-ms2qx      1/1     Running             0          21m
kube-system     metrics-server-7b4f8b595-2kxvv              1/1     Running             0          21m
ingress-nginx   ingress-nginx-controller-7474996559-79ztf   0/1     ContainerCreating   0          21s
ingress-nginx   ingress-nginx-admission-patch-km6v8         0/1     Completed           0          21s
ingress-nginx   ingress-nginx-admission-create-6jqz4        0/1     Completed           0          21s
```

Great, we have an ingress controller now. But, we do not have a load balancer. So we need to enable the ingress controller to use port 80 and 443 on the host. Let's patch the ingress controller. 

Create the patch first:

```shell
cat > ingress.yaml <<EOF 
spec:
  template:
    spec:
      hostNetwork: true
EOF
```

Then patch tthe ingress controller deployment like so:

```shell
kubectl patch deployment ingress-nginx-controller -n ingress-nginx --patch "$(cat ingress.yaml)"
```

If you now run `curl localhost` we see at there is a web server running. That means success!

```shell
[vagrant@centos02 ~]$ curl localhost                                                                                                                            
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

## Expose a web page

I am working on an application at the moment called 'carolyne'. You can find it here: https://gitlab.com/jacqinthebox/carolyne
I already pushed a container to the Docker hub, so we can just use that for now. 
Carolyne runs happily on port 4000.

Let's create a deployment and a service object quickly.

```shell
k create namespace apps
```
Then create the deployment

```shell
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: carolyne
  namespace: apps
  labels:
    app: carolyne
spec:
  replicas: 1
  selector:
    matchLabels:
      app: carolyne
  template:
    metadata:
      labels:
        app: carolyne
    spec:
      containers:
        - name: carolyne
          image: jacqueline/carolyne:latest
          ports:
            - containerPort: 4000
EOF
```
Let's create the service

```shell
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: carolyne-service
  namespace: apps
  labels:
    app: carolyne
spec:
  ports:
    - port: 4000
  selector:
    app: carolyne
EOF
```

Finally, let's create the ingress rules.

```shell
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /\$1
  name: apps-ingress-rules
  namespace: apps
spec:
  rules:
  - host: carolyne.moonstreet.local
    http:
      paths:
      - backend:
          service: 
            name: carolyne-service
            port: 
              number: 4000
        path: /(.*)
        pathType: Prefix
EOF
```

Need to add the hostname to my hostfile:

```shell
# sudo vim /etc/hosts
127.0.0.1       localhost carolyne.moonstreet.local
```

And now we can browse to that lovely web application I just crafted.

![1](/carolyne1.png)

![2](/carolyne2.png)


