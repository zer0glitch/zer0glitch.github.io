---
layout: post
title: "Using vagrant and kvm/libvirt for clustered development environments"
description: Using vagrant and kvm/libvirt for clustered development environments
---

Initial Setup
====

**Installation**

* [Installing Ansible](http://docs.ansible.com/ansible/intro_installation.html)
* [Installing Ansible libvirt/kvm/Ansible](https://galaxy.ansible.com/zer0glitch/vagrant-libvirt/)
* [Full Example](https://github.com/zer0glitch/vagrant-cluster-example)


- Install EPEL Repo

```
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
```

Testing the Vagrant configuration
====

For this test we will be using a standard base box. [Box info](https://atlas.hashicorp.com/centos/boxes/7/)

**Initialize the box**
This will download and stall the basebox on the intialization from the vagrant host.

```
vagrant init centos/7
```

**Exam the Vagrantfile**

I have removed most of the comments and used parameters for our example, this will allow us to have a vagrant machien running Centos7, but before we do that, we need to define a strategry for defining libvirt as the provider.

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"
end
```

We have 3 options of defining the libvirt provider.

1. Utilize the --provider tag on the vagrant up command

```
vagrant up --provider=libvirt
```

2. Environment variable

```
VAGRANT_DEFAULT_PROVIDER=libvirt;vagrant up
```

3. Confiration in our Vagrantfile.  This will specify the vagrant machine will only run on libvirt.  This may or may not be something you are looking for.  A developer may not be running libvirt, and need a Virtualbox default, but in a hosted environment, this may be the best method for standing up multiple machines.

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"

  # configures the machine for 
  config.vm.provider :libvirt do |libvirt|
    libvirt.nested = true
    libvirt.driver = "kvm"
  end
end
```

**Starting the Vagrant box to test**

```
vagrant up
```

SSH to the machine and make sure everything is up

```
virsh list 
```
You should see a VM running start with the name of the directory that you started in.

```
vagrant ssh
sudo -i
```

This should give you a root prompt on your VM.

**Setting up a cluster**

First we must down the VM by running

```
vagrant halt
```

Since Vagrant is based on ruby we can run a ruby loop to create multiple VMs inside of our Vagrant file.

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|

  # We are going to create 3 VMs
  (1..3).each do |i|

    # set a name for the VM
    vm_name="Server#{i}"

    config.vm.define vm_name do |node|
      node.vm.box = "centos/7"
    end
  end

  # configures the machine for 
  config.vm.provider :libvirt do |libvirt|
    libvirt.nested = true
    libvirt.driver = "kvm"
  end
end
```

**Starting the Vagrant box to test**

```
vagrant up
```

**Check the VM**

```
virsh list
```

You should see 3 servers running like the following:

```
101   test_Server3                  running
102   test_Server1                  running
103   test_Server2                  running
```

Running a `vagrant status` will also give you information about the running VMs.

```
[root@localhost test]# vagrant status
Current machine states:

Server1                  running (libvirt)
Server2                  running (libvirt)
Server3                  running (libvirt)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

Now if we try to access the machines with a `vagrant ssh` we will get the following error:

```
This command requires a specific VM name to target in a multi-VM environment.
```

We can access the VMs with the following, and see the hostname of the server:

```
[root@localhost test]# vagrant ssh Server1
[vagrant@localhost ~]$ hostname
localhost.localdomain
```

Let us give the machine a better hostname so we can identify the machine better.

```
 -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|

  # We are going to create 3 VMs
  (1..3).each do |i|

    # set a name for the VM
    vm_name="Server#{i}"

    config.vm.define vm_name do |node|
      node.vm.box = "centos/7"
      node.vm.hostname = vm_name
    end
  end

  # configures the machine for
  config.vm.provider :libvirt do |libvirt|
    libvirt.nested = true
    libvirt.driver = "kvm"
  end
end
```

Check the hostname of Server1

Running vagrant ssh with the -c option will allow us to run a command `vagrant ssh Server1 -c hostname`, and we should see a result like this.

```
server1
Connection to 192.168.121.208 closed.
```

Now we can work on a more complex example.  We are going to create a single database server and two web servers.  Our configuration will be as follows:

```
Two Web Servers:
- httpd
- postgres client

One Database Server
- postgres server/client
```

To accomplish this we are going to have multipe configuration sections. Our result Vagrant file looks like this:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|


  # Configure the Single Database server
  config.vm.define "database" do |node|
    node.vm.box = "centos/7"
    node.vm.hostname = "database"
  end

  # We are going to create 3 VMs
  (1..2).each do |i|

    # set a name for the VM
    vm_name="web#{i}"

    config.vm.define vm_name do |node|
      node.vm.box = "centos/7"
      node.vm.hostname = vm_name
    end
  end

  # configures the machine for
  config.vm.provider :libvirt do |libvirt|
    libvirt.nested = true
    libvirt.driver = "kvm"
  end
end
```

So we have made our life easier.  We have started with a preconfigured VM, so no waiting for the installation to complete, we have the VMs stood up in the matter of mintunes and now we are ready to provision the machine.  This is an additional power of Vagrant.  We can automaticall provision and configure our machines.  We will be using the ansible provisioner.  To conigure the servers with the roles they need.


Sample playbooks can be downloaded from [here](https://github.com/zer0glitch/vagrant-cluster-example).

The vagrant file will be modified to do the following things:
- Add provisioning sections for the servers
- create server groups


**New Vagrant file**

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|

  # The web servers
  web_servers = []

  # Configure the Single Database server
  config.vm.define "database" do |node|
    node.vm.box = "centos/7"
    node.vm.hostname = "database"

#    config.vm.provision :ansible do |ansible|
#      ansible.groups = {
#        "database" => ["database"],
#      }
#      ansible.playbook = "database.yml"
#      ansible.verbose = true
#      ansible.sudo = true
#      ansible.limit = "all"
#    end

  end

  # We are going to create 3 VMs
  (1..2).each do |i|

    # set a name for the VM
    vm_name="web#{i}"

    # add the server to web server list
    web_servers.push(vm_name)

    config.vm.define vm_name do |node|
      node.vm.box = "centos/7"
      node.vm.hostname = vm_name
    end

    config.vm.provision :ansible do |ansible|
      ansible.groups = {
        "webservers" => web_servers,
      }
      ansible.playbook = "web.yml"
      ansible.verbose = true
      ansible.sudo = true
      ansible.limit = "all"
    end

  end

  # configures the machine for
  config.vm.provider :libvirt do |libvirt|
    libvirt.nested = true
    libvirt.driver = "kvm"
  end
end
```

**database.yml**

```
---
- hosts: database
  tasks:

  - yum: name=postgresql-server state=installed
```

**web.yml**

```
---
- hosts: webservers
  tasks:

  - yum: name=postgresql state=installed
  - yum: name=httpd state=installed
```
