Title: More Vagrant Learnings
Date: 2015-03-29 19:59
Author: Ryan McKay
Slug: more-vagrant-learnings
Status: published
Summary: Last time I wrote about [how we use Vagrant to manage our local development environment](http://againstentropy.blogspot.com/2015/01/vagrant-for-local-development.html) at work. Since then I've dug into Vagrant a little more deeply and found a few nuggets I wanted to share.  

Last time I wrote about [how we use Vagrant to manage our local development environment](http://againstentropy.blogspot.com/2015/01/vagrant-for-local-development.html) at work. Since then I've dug into Vagrant a little more deeply and found a few nuggets I wanted to share.  

## Package your own Vagrant Box
There are some great base boxes (images) available for vagrant. It will even install them for you. If you want to tweak the box, you can use one or more of the many available provisioning providers. However, sometimes you might want to just bake the tweak into a new base image, for example, if the tweak is something you want to apply to all your instances, or if it is time consuming. Vagrant makes it very easy to do this.

### Initialize, start VM
``` bash
$ vagrant init hashicorp/precise32
$ vagrant up
```

###Install new stuff in VM
``` bash
$ vagrant ssh
vagrant@precise32:~$ echo 'set editing-mode vi' > ~/.inputrc
vagrant@precise32:~$ sudo apt-get update
vagrant@precise32:~$ sudo apt-get install vim
vagrant@precise32:~$ exit
```

### Package up VM image as a new Vagrant box
``` bash
$ vagrant package
==> default: Attempting graceful shutdown of VM...
==> default: Clearing any previously set forwarded ports...
==> default: Exporting VM...
==> default: Compressing package to: /Users/ryanmckay/projects/vagrant/package.box

$ ls -l
total 801784
-rw-r--r--  1 ryanmckay  staff       3001 Mar 14 22:13 Vagrantfile
-rw-r--r--  1 ryanmckay  staff  410509096 Mar 15 20:32 package.box

$ vagrant box list
hashicorp/precise32 (virtualbox, 1.0.0)

$ vagrant box add --name 'ryanmckay/withvim' package.box 
==> box: Adding box 'ryanmckay/withvim' (v0) for provider: 
    box: Downloading: file:///Users/ryanmckay/projects/vagrant/package.box
==> box: Successfully added box 'ryanmckay/withvim' (v0) for 'virtualbox'!

$ vagrant box list
hashicorp/precise32 (virtualbox, 1.0.0)
ryanmckay/withvim   (virtualbox, 0)
```

## Provisioning
Vagrant provides a boatload of provisioning providers, e.g. puppet, chef, ansible, and even shell script. Normally configured provisioners are run when bringing up the VM for the first time. However, you can also run the provisioners with vagrant provision. For example, if you have the following:

### provision_counter.sh shell script
``` bash
echo "provisioning"
if [ -f /etc/provision_count ]; then
 counter=`cat /etc/provision_count`
else
 counter=0
fi
counter=$((counter+1))
echo -n $counter > /etc/provision_count
echo "provisioned: $counter"
```

### Configured in your Vagrantfile
``` 
config.vm.provision "shell", path: "provision_counter.sh"
```

### When you vagrant up, provisioners are run
``` bash
$ vagrant up
...
==> default: Running provisioner: shell...
    default: Running: /var/folders/k5/6mc1_m_n1675572ts12qr7588mtjh5/T/vagrant-shell20150329-86419-qkjn7x.sh
==> default: stdin: is not a tty
==> default: provisioning
==> default: provisioned: 1
```

### And you can run provisioners explicitly
``` bash
$ vagrant provision
==> default: Running provisioner: shell...
    default: Running: /var/folders/k5/6mc1_m_n1675572ts12qr7588mtjh5/T/vagrant-shell20150316-31070-d35qp6.sh
==> default: stdin: is not a tty
==> default: provisioning
==> default: provisioned: 2
```

You can even add/modify provisioning lines in the Vagrantfile of a running VM, and when you run 'vagrant provision', the new lines will be used.

## Reloading Vagrantfile
The Vagrantfile is the main config file for your vagrant VM. It specifies all the stuff that needs to be set up before the OS is running, as well as the hooks for provisioning once it has booted. The Vagrantfile can be reloaded by 'vagrant reload', which is basically 'halt' and then 'up'. It does not re-run provisioners unless you tell it to. Some examples of why you would want to do this are to modify your network setup or your shared folders. For example, if you want to:  

### Add a network port forward and a shared folder
``` 
config.vm.network "forwarded_port", guest: 80, host: 8080config.vm.synced_folder "src/", "/srv/website"
```

Then 'vagrant reload' would restart your VM with the new configuration without having to reprovision.

## Vagrant SSH
'vagrant ssh' is how you ssh into an interactive session on your VM. You can also use 'vagrant ssh -c command' to run a command in the VM instead of an interactive shell, similar to 'ssh host command'.

``` bash
$ vagrant ssh -c 'hostname'
precise32
```

In order to connect to your VM with regular ssh (or anything that depends on ssh, e.g. scp or rsync over ssh), you need to

### Export vagrant ssh-config
``` bash
$ vagrant ssh-config
Host default
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /Users/ryan.mckay/.vagrant.d/insecure_private_key
  IdentitiesOnly yes
  LogLevel FATAL

$ vagrant ssh-config > ssh-config
```

### Then provide ssh-config to ssh and friends
``` bash
$ ssh -F ssh-config default 'hostname'
precise32

$ touch scp_me
$ scp -F ssh-config scp_me default:/tmp
scp_me                              100%    0     0.0KB/s   00:00    
$ ssh -F ssh-config default 'ls /tmp'
scp_me

$ mkdir rsync_test
$ touch rsync_test/rsync_me
$ rsync -a -e 'ssh -F ssh-config' rsync_test default:/tmp/
$ ssh -F ssh-config default 'find /tmp'
/tmp
/tmp/rsync_test
/tmp/rsync_test/rsync_me
/tmp/scp_me
```