---
title: "Vagrant with the libvirt plugin"
date: 2020-09-20T10:37:24+02:00
draft: true
---

Here is a post about how to create virtual machines with Vagrant and the libvirt plugin.

## Why use libvirt as a vagrant provider
Because when on a modern Linux distro running Gnome, chances are it's already there!

I use Vagrant extensively to run quick, short lived virtual machines to test and try out things. I always used and use Virtualbox as a virtual machine provider for Vagrant.
Maybe I am late to the party but I recently discovered [Gnome Boxes](https://help.gnome.org/users/gnome-boxes/stable/supported-protocols.html.en) and I was wondering about the underlying architecture. Then I went on vacation and finally took the time to investigate.

The answer is: [KVM](https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine). Together with [QEMU](https://wiki.qemu.org/Main_Page) and [libvirt](https://libvirt.org/).
(I remember using Qemu back in the 2007 or so but thinking it was terribly slow and left it for what is was. But its 2020 now, so let's dive in again!)

## KVM, QEMU, libvirt: lost in the limbo

KVM (like Xen, ESXi and Hyper-V) is a type 1 hypervisor. A hypervisor is software that creates and runs virtual machines. Type 1 hypervisors run directly on the hardware. So first the hypervisor is loaded which then loads the OS. Or is it?
Not really as [this](https://medium.com/teamresellerclub/type-1-and-type-2-hypervisors-what-makes-them-different-6a1755d6ae2c) article states. KVM turns Linux to a type 1 hypervisor, but at the same time runs a fully functional OS. Which makes the type 1 or type 2 discussion a bit weird. 

KVM provides device abstraction but no processor emulation. It exposes the /dev/kvm interface, which a user mode host can then use to:
 * Bootstrap an iso (set up theguest VM address space)
 * Feed the guest simulated I/O
 * Map the guest's video display back onto the system host
 
On Linux, QEMU is one such userspace host. QEMU uses KVM when available to virtualize guests at near-native speeds, but otherwise falls back to software-only emulation.

The last piece of the puzzle is [libvirt](https://wiki.libvirt.org/page/FAQ#What_is_libvirt.3F)
Libvirt provides a convenient way to storage and network interface management for virtual machines. Libvirt includes an API library, a daemon (libvirtd), and a command line utility (virsh). 

Gnome Boxes uses libvirt, as does Vagrant. Hence the blog title: using Vagrant with the libvirt plugin.

![plaatje](/kvm.jpg)


## Let's run a VM already

If you run this on Fedora 32, you can see that lots of packages are already installed:

```sh
dnf list --installed | grep libvirt
dnf list --installed | grep qemu
```

We need Vagrant and the vagrant-libvirt plugin:

```sh
sudo dnf install vagrant
sudo vagrant plugin install vagrant-libvirt
```

Then make a folder and add a Vagrantfile in it

```sh

mkdir ~/vagrant/centos && cd ~/vagrant/centos

echo "$extra = <<-EXTRA
dnf update -y && dnf install vim net-tools git lsof tar -y
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
end" > Vagrantfile
```

And then do

```sh
vagrant up
```

Hooray!

![centos](/vagrant-centos.png)

## Afterthought: What about LXC and LXD?

You can also run [LXD](https://linuxcontainers.org/lxd/) instead of KVM. I think that LXD is the Canonical way, while KVM has been adopted by RedHat (fact check probably needed). 
Traditionally, LXD is used to create system containers, light-weight virtual machines that use Linux Container features and not hardware virtualization.
However with LXD you can create both system containers and virtual machines. [Here](https://www.cyberciti.biz/faq/how-to-install-setup-lxd-on-fedora-linux/) is a nice blog post explaining that. 
And it also states you can install LXD using snaps, which I am not very fond of. Again, that's just my opinion. 
