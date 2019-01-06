---
title: "Monitoring with Prometheus"
date: 2019-01-04T16:29:13+01:00
draft: false
image: "uploads/monitoring.jpeg"
tags: ["devops"]
---

# Beginning monitoring with Prometheus from scratch.

Prometheus is a an open-source and popular [CNCF project](https://cncf.ci/). It collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts if some condition is observed to be true.

Getting started with Prometheus shouldn't be that hard. You can get up and [running quickly](https://itnext.io/kubernetes-monitoring-with-prometheus-in-15-minutes-8e54d1de2e13). And although you get some beautiful graphs in a second, you may not know what you are doing. So here is a blog that takes some steps back and takes the from scratch approach.

I would recommend [this](https://www.prometheusbook.com/) book by the way.

## Installing Prometheus


