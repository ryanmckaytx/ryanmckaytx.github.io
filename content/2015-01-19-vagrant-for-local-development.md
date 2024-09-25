Title: Vagrant for Local Development
Date: 2015-01-19 15:46
Author: Ryan McKay
Slug: vagrant-for-local-development
Status: published

When I was hired at my current company about a year and a half ago, it was immediately obvious that this company took dev-ops much more seriously than any other place I had worked before. In fact, a lot of the dev-ops culture is pushed by the ops team, which is a dramatic, and very welcome, departure from my previous experience. This team follows a lot of the principles espoused in the Continuous Delivery book. In this post, I'm going to focus on the principle of making all pre-prod environments as production-like as possible, and in particular, what has been done in the local development environment.  

The Ops team uses puppet for server provisioning in all environments. This facilitates the separation of the vast majority of configuration that is the same across environments and servers within an environment from the relatively small proportion that that needs to be different.  

## The Problem
Our project currently comprises 7 microservices and one CLI app, all based on Spring Boot. Most of the services are RESTful; a couple are message driven. They all expose a management port, primarily for doing health checks and gathering metrics. Deployable artifacts are built by Jenkins and stored in Nexus. Deployment is handled by a standardized shell script which is placed on the target servers by Puppet during provisioning.

We also use several 3rd-party applications, including MySQL, Mongo, Rabbit MQ, Splunk, and a CLI ETL application. These are also provisioned by Puppet.

The provisioning and deployment infrastructure is almost identical in production and the non-local pre-prod environments. However, until several months ago, provisioning and deploying in the local development environment (i.e. developers' laptops) was still a manual affair. This was achieved by following meticulously crafted documentation about how to set up a new workstation. Invariably, differences crept in, e.g. different versions of 3rd party apps, different configuration, even different package managers. Mostly these differences were benign, but sometimes they did cause problems. And that type of problem is much more difficult to diagnose and fix than a bug in the application layer.

## Enter Vagrant
[Vagrant](https://www.vagrantup.com/) enables you to "Create and configure lightweight, reproducible, and portable development environments." Vagrant VM images are called "boxes". You can produce your own box, or find one at <https://atlas.hashicorp.com/boxes/search>. Once you find one (e.g. hashicorp/precise32), getting it up and running is simple:  

``` bash
vagrant init hashicorp/precise32vagrant up
```

Then you can ssh to it with:  

``` bash
vagrant ssh
```

And to stop it with various levels of severity:  

``` bash
vagrant suspend/halt/destroy
```

When you run init, vagrant creates a basic Vagrantfile in your current directory which has a little bit of configuration in it, and a lot of commented out stuff so you can see how to do common things.  

## How we use Vagrant
Our gradle build keeps updated a vagrant.config file which contains a simple ruby structure with information about our deployable services and cli apps:

``` bash
PROJECTS = {
        'core-foo' => {
                :version           => '3.211',
                :project_root      => '/Users/ryan.mckay/projects/shared-services/core-foo',
                :use_puppet_config => true,
                :use_puppet_deploy => true,
                :im_just_a_jar => false,
        },
...
```

### Our Vagrantfile
**Load configuration about our deployables**
``` bash
load 'vagrant.config'
```

**Configure some vm settings like memory and number of cpus**  
``` bash
config.vm.provider :virtualbox do |vb|
    # Use VBoxManage to customize the VM. For example to change memory:
    vb.customize ["modifyvm", :id, "--memory", "5120"]
    vb.customize ["modifyvm", :id, "--cpus", "4"]
end 
```

**Run external provisioning script**  
``` bash
config.vm.provision :shell, :path => "../scripts/05-vagrant.sh"
```

**Deployment (calls deployment script landed by puppet for each deployable)**
``` bash
 PROJECTS.each { |artifact_name, artifact_config|
    args = [
      artifact_name,
      artifact_config[:version],
      artifact_config[:im_just_a_jar] ? 'jar' : 'service'
    ]
    config.vm.provision :shell, :path => "../scripts/10-deploy.sh", :args => args
  }
```

**Make sure everything came up**  
``` bash
config.vm.provision :shell, :path => "../scripts/99-runtests.sh"
```

**We use a private custom base image**  
``` bash
  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
   config.vm.box_url = "http://foo.com/images/debian7-base.box"
```

**Forward application ports to host system with 10,000 offset**  
``` bash
   # Application ports
  (8000..9999).step(10).each do |port|
    config.vm.network :forwarded_port, :host => 10000 + port, :guest => port
    config.vm.network :forwarded_port, :host => 10001 + port, :guest => 1 + port
  end
```

**3rd party service port forwarding**  
``` 
  # default intellij debugging port
  config.vm.network :forwarded_port, :host => 5005, :guest => 5005

  # mysql port
  config.vm.network :forwarded_port, :host => 3306, :guest => 3306

  # mongodb port
  config.vm.network :forwarded_port, :host => 27017, :guest => 27017

  #rabbitmq 
  config.vm.network :forwarded_port, :host => 5672, :guest => 5672
  config.vm.network :forwarded_port, :host => 15672, :guest => 15672

  #splunk mangagement port
  config.vm.network :forwarded_port, :host => 18089, :guest => 8089
```

**Host/VM synced folder**  
``` 
  # create sync folder for integration test data
  local_sync_folder = "/tmp/foo_integration"
  FileUtils.mkdir_p local_sync_folder
  File.chmod(0775, local_sync_folder)
  config.vm.synced_folder local_sync_folder, "/mnt/filer/foo/foodev"
```

**Deploy locally built artifacts if available**  
``` 
  PROJECTS.each { |service_name, service_config|
    if (service_config.key?(:project_root) && service_config[:use_puppet_config] != true)
        host_config_dir = service_config[:project_root] + "/src/config"
        if (File.directory?(host_config_dir))
          config.vm.synced_folder host_config_dir, "/var/bv/conf/#{service_name}/local"
          config.vm.provision :shell, :inline => "ln -sf /var/bv/conf/#{service_name}/local/dev-local.properties /var/bv/conf/#{service_name}/properties"
          config.vm.provision :shell, :inline => "ln -sf /var/bv/conf/#{service_name}/local/dev-migrate.properties /var/bv/conf/#{service_name}/migrate.properties"
        else
          puts "Project root found for #{service_name}, but src/config not detected. Local config directory was not mounted"
        end
    end
    if (service_config.key?(:project_root) && service_config[:use_puppet_deploy] != true)
      guest_folder = "/var/bv/apps/#{service_name}/local"
      local_artifact = localArtifactNameFor(service_name)
      host_lib_dir = service_config[:project_root] + "/build/libs"
      if (File.directory?(host_lib_dir))
        config.vm.synced_folder host_lib_dir, guest_folder
        command_to_run = "ln -sf #{guest_folder}/#{local_artifact} /var/bv/apps/#{service_name}/current.jar"
        if (!service_config[:im_just_a_jar])
           command_to_run += " && /etc/init.d/#{service_name} stop && /etc/init.d/#{service_name} start"
        end
        config.vm.provision :shell, :inline => command_to_run
      else
        puts "Project root found for #{service_name}, but build/libs not detected. Local build directory was not mounted."
      end
    end
  }

end

def localArtifactNameFor(service_name)
  version = PROJECTS[service_name][:version]
  major_version = version.split('.')[0]
  return "#{service_name}-#{major_version}.99999.jar"
end
```

## Conclusion
Now that we have started using Vagrant, spinning up a new developer workstation takes a matter of minutes, and you know you got it exactly right. We can run integration and manual tests, and be confident in the result. And we are regularly exercising a good portion of the provisioning, deployment, and monitoring infrastructure that is used in production.