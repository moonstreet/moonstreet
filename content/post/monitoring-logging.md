---
title: "Monitoring with Prometheus"
date: 2019-01-04T16:29:13+01:00
draft: false
image: "uploads/monitoring.jpeg"
tags: ["devops"]
---

# Beginning monitoring with Prometheus from scratch.

Dear all, this is my first blogpost on this website. 
Currently I am working for a SaaS project that has some Microservices which are hosted on Kubernetes. And as a devops engineer I am not only responsible for the continuously delivery part but also for things like monitoring. 

It seems like Prometheus is the de-facto monitoring solution for Kubernetes nowadays. Prometheus is very popular according to the [CNCF project](https://cncf.ci/)  and is also mentioned in the Kubernetes [docs](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/).  

So what does Prometheus do? Well, it collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts if some condition is observed to be true.

Getting started with Prometheus shouldn't be that hard. You can get up and [running quickly](https://itnext.io/kubernetes-monitoring-with-prometheus-in-15-minutes-8e54d1de2e13). And although you get some beautiful graphs in a second, you may not know what you are doing. So here is a blog that takes some steps back and takes the from scratch approach.

I would recommend [this](https://www.prometheusbook.com/) book by the way.

## Installing Prometheus

Grab the latest version from [here](https://github.com/prometheus/prometheus/releases)

```sh
tar -xzf prometheus-2.6.0.linux-amd64.tar.gz
sudo cp prometheus-2.6.0.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.6.0.linux-amd64/promtool /usr/local/bin/
```

You can now go ahead and see if the installation went well:

```sh
prometheus --version
```

This should return the current version.

![](/uploads/monitoring-logging-1.png)

Now go ahead and copy the promotheus.yml file from the extracted folder to /etc/prometheus and start the server:

```sh
sudo mkdir /etc/prometheus
sudo cp prometheus.yml /etc/prometheus/
prometheus --config.file "/etc/prometheus.yml"
```

If you browse to `localhost:9090` you will see the Prometheus dashboard and you can execute some of the queries in the dropdown list.
What it basically does right now is monitoring itself.

![](/uploads/monitoring-logging-2.png)


### Monitoring a node

Download and install the node exporter here:
https://github.com/prometheus/node_exporter/releases/download/v0.17.0/node_exporter-0.17.0.linux-amd64.tar.gz


```sh
tar -xzf node_exp*
sudo cp node_exporter-*/node_exporter /usr/local/bin
```

Configure the Textfile collector

```sh
sudo mkdir -p /var/lib/node_exporter/textfile_collector
echo 'metadata{role="docker_server",datacenter="RDAM"} 1' | sudo tee \
 /var/lib/node_exporter/textfile_collector/metadata.prom
```

Then we can launch node_exporter:

```sh
node_exporter --collector.textfile.directory /var/lib/node_exporter/textfile_collector \
--collector.systemd --collector.systemd.unit-whitelist="(docker|ssh|rsyslog).service"
```

Then scrape it. Our new prometheus.yml file will look like so:

```yml
scrape_configs:
- job_name: 'prometheus'
  static_configs:
    - targets: ['localhost:9090']
- job_name: 'node'
  static_configs:
    - targets: ['192.168.2.66:9100']
```











