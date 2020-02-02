---
title: "Add Helm 2 to MicroK8s"
date: 2020-02-02T13:02:13+01:00
draft: false
image: ""
author: ""
---

Next thing is to install some nice application on our cluster. Why not use something that's ready to use? Enter Helm.
But I'm old fashioned, I still did not take the plunge and use Helm v3. need to to that upgrade, but that's for a later blogpost.

So let's enable Helm 2.14 on our MicroK8s cluster.

```bash
~  (⎈ microk8s:default)                                                                                                                                                                                                               ⍉
▶ helm init
$HELM_HOME has been configured at /home/jacqueline/.helm.
Error: error installing: the server could not find the requested resource
```
No fun.

Luckily the answer is right here:
https://stackoverflow.com/questions/58075103/error-error-installing-the-server-could-not-find-the-requested-resource-helm-k

So let's go ahead and run
```bash
helm init --service-account tiller --output yaml | sed 's@apiVersion: extensions/v1beta1@apiVersion: apps/v1@' | sed 's@  replicas: 1@  replicas: 1\n  selector: {"matchLabels": {"app": "helm", "name": "tiller"}}@' | kubectl apply -f -
```

```bash
kubectl -n kube-system create serviceaccount tiller

kubectl create clusterrolebinding tiller \
  --clusterrole=cluster-admin \
  --serviceaccount=kube-system:tiller
```

Delete the current replicaset

```bash
kubectl delete -n kube-system replicaset tiller-deploy-77855d9dcf
```

And then it works.

```bash
~  (⎈ microk8s:kube-system)                                                                                                                                                            
▶ kubectl get pods --all-namespaces                                                                         
NAMESPACE     NAME                                              READY   STATUS    RESTARTS   AGE
kube-system   coredns-7b67f9f8c-jbtxg                           1/1     Running   0          3h55m
kube-system   dashboard-metrics-scraper-687667bb6c-tcfm7        1/1     Running   0          3h55m
kube-system   heapster-v1.5.2-5c58f64f8b-55tgt                  4/4     Running   0          3h55m
kube-system   kubernetes-dashboard-5c848cc544-k6p5b             1/1     Running   0          3h55m
kube-system   monitoring-influxdb-grafana-v4-6d599df6bf-786zc   2/2     Running   0          3h55m
kube-system   tiller-deploy-77855d9dcf-t5wfx                    1/1     Running   0          17m

```

