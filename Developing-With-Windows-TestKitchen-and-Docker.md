Notes on Chef Development
=========================

My goal is to use my Windows 8.1 laptop as a development environment for Chef cookbooks targeting Linux hosts:

- Use SourceTree, Sublime Text and other nice GUI tools.
- Execute Test Kitchen commands in a cmd, PowerShell or GitBash window.
- Use Docker for the Test Kitchen targets because it is very fast and light-weight.


Disclaimer:
I am a Linux guy, not a Windows fan. I want to use the Windows laptop because that is what I and many of my peers use in our day jobs. It isn't reasonable in that environment to have a Linux workstation, dual-boot or run a full Linux desktop in a VM.


Preparing The Laptop
--------------------

### Install Some Tools

* SourceTree
* git
* gitbash / gitshell
* GitHub
* PuTTY/KiTTY
* Sublime Text 3
* [ChefDK](https://downloads.chef.io/chef-dk/)

Substitute favorite replacements as appropriate.

### Install and Test boot2docker

Download the installer from http://docs.docker.com/installation/windows/. Let it install all of the tools it provides then make sure it worked.

```
> boot2docker.exe version
Boot2Docker-cli version: v1.6.0
Git commit: 9894ae9
> "c:\Program Files\Oracle\VirtualBox\VBoxManage.exe" --version
4.3.26r98988
```

Launch boot2docker, set the environment variables it tells you to set and run the hello-world example. You should also be able to do *docker ps*, *docker images* and other docker commands in a cmd, PowerShell or GitBash window.

Note:  I initially installed VirtualBox before installing boot2docker and I had issues with the Windows docker client not talking to the boot2docker docker daemon. I played some tricks with port forwarding in VirtualBox and it worked but my Test Kitchen attempts failed. I removed VirtualBox and boot2docker then reinstall boot2docker and let it install VirtualBox and now things are working. YMMV


Using Test Kitchen
--------------------

### Test The Basics

Before we can dive in and start doing Real Work, we need to make sure that Test Kitchen will cooperate with the environment we've created.

Start working through the [Getting Started Guide](http://kitchen.ci/docs/getting-started). When you get to [Creating a Cookbook](http://kitchen.ci/docs/getting-started/creating-cookbook) use the Kitchen::Docker driver instead of Vagrant.

```
set PATH=%PATH%;C:\opscode\chefdk\embedded\bin:c:\Users\<YourUsername>\.chefdk\gem\ruby\2.1.0\bin
set DOCKER_HOST=tcp://192.168.###.###:2376
set DOCKER_CERT_PATH=C:\Users\<YourUsername>\.boot2docker\certs\boot2docker-vm
set DOCKER_TLS_VERIFY=1
kitchen init --driver=kitchen-docker
```

Modify .kitchen.yml to configure Kitchen::Docker. This is more or less my final .kitchen.yml; you will start with a simpler one as you work through the guide. The important thing is the *driver* section at the top.

```
---
driver:
  name: docker
  binary: docker
  socket: tcp://192.168.###.###:2376
  require_chef_omnibus: true

provisioner:
  name: chef_zero

platforms:
  - name: centos-6.6
  - name: ubuntu-10.04
  - name: ubuntu-12.04
  - name: ubuntu-14.04
    driver_config:
      image: tragus/chef-omnibus
      require_chef_omnibus: false

suites:
  - name: default
    run_list:
      - recipe[git::default]
    attributes:
  - name: server
    run_list:
      - recipe[git::server]
    attributes:
    excludes:
      - centos-6.6
```

Note: I have used my [tragus/chef-omnibus](https://registry.hub.docker.com/u/tragus/chef-omnibus/) image for the Ubuntu 14.04 platform. That image includes Chef 12.3.0 and a couple of other things to make the node provisioning process quicker.

If everything is setup correctly you can now do:

```
> kitchen list
  C:\...\chef-notes\git-cookbook>kitchen list
  Instance             Driver  Provisioner  Verifier  Transport  Last Action
  default-centos-66    Docker  ChefZero     Busser    Ssh        <Not Created>
  default-ubuntu-1004  Docker  ChefZero     Busser    Ssh        <Not Created>
  default-ubuntu-1204  Docker  ChefZero     Busser    Ssh        <Not Created>
  default-ubuntu-1404  Docker  ChefZero     Busser    Ssh        <Not Created>
  server-ubuntu-1004   Docker  ChefZero     Busser    Ssh        <Not Created>
  server-ubuntu-1204   Docker  ChefZero     Busser    Ssh        <Not Created>
  server-ubuntu-1404   Docker  ChefZero     Busser    Ssh        <Not Created>
>
> kitchen create all
  ...
```

The first time an image is pulled will take a moment or two. After that, provisioning will be very fast. You can use docker commands to see the images and containers, launch a shell and do the other things you do with Docker on Linux.

```
> docker images
  ...
> docker ps
  ...
> docker exec -i -t <image-name> /bin/bash
    root@...:/# cat /etc/os-release
    NAME="Ubuntu"
    VERSION="12.04.5 LTS, Precise Pangolin"
    ID=ubuntu
    ID_LIKE=debian
    PRETTY_NAME="Ubuntu precise (12.04.5 LTS)"
    VERSION_ID="12.04"
```

### Continue The Tutorial

Continue the [Getting Started Guide](http://kitchen.ci/docs/getting-started). When you get to [Adding a Dependency](http://kitchen.ci/docs/getting-started/adding-dependency), before you *gem install berkshelf*, you will need to add two directories to your PATH if you didn't already do so. How you do it depends on whether you're using cmd, PowerShell or GitBash. This is how I did it in a cmd window:

```
PATH=%PATH%;c:\opscode\chefdk\embedded\bin\gem;c:\Users\<YourUsername>\.chefdk\gem\ruby\2.1.0\bin
```

### Missing Gems & Such

Depending on your installation, you may have a thing or two missing. This is easily fixed by installing the missing gems.

```
gem install kitchen-docker
gem install kitchen-nodes
```

Closing Thoughts
----------------

These notes have focused on the integration of Docker (via boot2docker) and the Test Kitchen component of ChefDK using a Windows laptop as the development workstation. You can certainly use Docker on Windows without Test Kitchen and you can use Test Kitchen on Windows without Docker. I happen to like the combination because it is super-fast and light-weight for test-driven Chef development and the GUI tools are easy to install, configure and use. The end result is very similar to a Mac or Linux workstation setup for those of us not fortunate enough to have such a thing.
