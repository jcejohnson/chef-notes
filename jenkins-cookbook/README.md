tragus_jenkins Cookbook
=========================

This is *not* meant to be a general purpose Jenkins cookbook. Its only purpose is to illustrate using Docker and Test Kitchen with multiple nodes in a master/slave relationship.

Requirements
------------
- Docker
- Test Kitchen
- kitchen-docker
- kitchen-nodes

Use Case
--------

Jenkins, like many other tools, is composed of two parts: master & slave. The master drives the action and directs zero or more slaves to execute jobs. In my day job I work for an company with many internal organizations (a.k.a. - business units). Each of those organizations has one or more development teams and a dedicated Jenkins master with multiple slaves. I need to create Chef recipes that allow us to deploy and manage these multiple masters and their many slaves. In order to do that, I need a mechanism for testing the process -- specifically, the ability for the slaves to find and attach to (via jnlp) the correct master out of the dozens that are known to the Chef server.

Test Kitchen isn't necessarily designed for multi-node testing but with the kitchen-nodes provisioner and explicit use of the kitchen lifecycle components we can do a very good job of testing this kind of scenario.

Kitchen::Nodes
--------------

TODO -- Document how this is useful.

Beyond Jenkins
--------------

Of course, you could apply this same technique to other master/slave or server/client setups. Though I have used Jenkins as my example, the approach will work equally well for any similar application. On the other hand, not all master/slave or server/client application architectures will lend themselves to this. In particular, situations where the slave/client component discovers its master/server rather than being told where to attach (in this case, by Chef), is probably inappropriate.

Ubuntu & Centos
------

With Ubuntu 14.04:

```
platforms:
  - name: ubuntu-14.04
    driver:
      image: tragus/chef-omnibus
      require_chef_omnibus: false
```

converge failed when trying to start the slave:

```
/sbin/sv warning: jenkins-slave: unable to open supervise/ok: file does not exist
```

Since my goal isn't to debug Jenkins slave on Ubuntu in Docker, I took the easy way out and switched to CentOS 6.6.