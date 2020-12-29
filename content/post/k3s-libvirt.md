---
title: "K3s on a Centos vm"
date: 2020-09-27T09:17:02+02:00
draft: false
showShare: false
categories:
- Technology 
tags:
- linux
- k3s
---

This post is about installing lightweight Kubernetes (K3s) on our new vm.

## K3s

The previous post was all about being able to install Kubernetes quickly. For that I will use [k3s](https://k3s.io/) from Rancher because it is fast. 
Let's go!

## Prepare

We do this for testing so we can safely disable the firewall and SELinux on our CentOS vm.
This is because SELinux is still experimental in K3s.
We also need to enable ssh password authentication so we can issue scp commands from our host machine.
For this we just add the following lines to the extra script part of our Vagrantfile: 

```ruby
$extra = <<-EXTRA
dnf update -y && dnf install vim net-tools git lsof tar -y
sed -i 's/enforcing/disabled/g' /etc/selinux/config
sed -i 's/PasswordAuthentication\s.*no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
systemctl disable firewalld
systemctl restart sshd
reboot
EXTRA

Vagrant.configure("2") do |config|
  config.vm.define "node01" do |node01_config|
    node01_config.vm.box ="generic/centos8"
    node01_config.vm.hostname = "centos01"
    node01_config.vm.provider :libvirt do |domain|
      domain.memory = 4096
      domain.cpus = 2
      domain.nested = true
      domain.volume_cache = 'none'
    end
  end
  config.vm.provision "shell", inline: $extra
end
```

Now log on the the virtual machine and issue these commands:

```sh
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```

Now wait a bit. Once ready, you can issue:

```sh
kubectl get pods --all-namespaces
```

## Manage K3s from the host

```sh
 scp vagrant@192.168.122.232:/home/vagrant/.kube/config .
```
 
Edit the config and change the 127.0.0.1 address from the server to the ip address of your vm. (Mine is 192.168.122.232)

Then run: 
 
```sh
kubectl get pods --kubeconfig config --all-namespaces
```

Hooray!

![k3s](/vagrant-k3s.png)