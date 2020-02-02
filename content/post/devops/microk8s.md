---
title: "Microk8s"
date: 2020-02-02T11:44:02+01:00
draft: false
image: ""
author: ""
---


# Installing MicroK8s (and use kubectl, not microk8s.kubectl)

Let's install MicroK8s, the easiest way to install Kubernetes on your computer. 
After that, let's configure our system to use kubectl instead of microk8s.kubectl. 

## Prerequisites

Your Linux distribution needs to support Snaps.
https://snapcraft.io/docs/installing-snapd


## Install it

Just head over to: https://microk8s.io/docs/ or go ahead and follow my lead:

```bash
sudo snap install microk8s --classic
```

Join the group

```bash
sudo usermod -a -G microk8s $USER
```

Add some extras

```bash
microk8s.enable dns dashboard
``` 

## Add the config to your existing config
 
As root:

```bash
sudo su
cd /var/snap/microk8s/1173/credentials/
cp client.config /home/<YOUR-USERNAME>/Desktop/client.config
```

As your non root user.
Make sure to backup your existing config first!

```bash
sudo chown $USER.$USER /home/$USER/Desktop/client.config
KUBECONFIG=config:~/Desktop/client.config:~/.kube/config-ok kubectl config view --flatten > ~/.kube/config
```

## View the dashboard

```bash
token=$(kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
kubectl -n kube-system describe secret $token

kubectl port-forward -n kube-system service/kubernetes-dashboard 10443:443
```
Log in with the token:

![kubernetes-moonstreet](https://i.imgur.com/8jMzSp8.png)

And behold the dashboard:

![dashboard-moonstreet](http://i.imgur.com/i72a4nV.png)

So there is a Kubernetes cluster in a few minutes.
