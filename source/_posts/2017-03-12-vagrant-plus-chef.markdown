---
layout: post
title: "Vagrant+Chef"
date: 2017-03-12 19:32:51 +0800
comments: true
categories: vagrant chef tool productivity
---

##引言

前面我介绍了Chef的环境搭建和简单的cookbook的创建执行，也介绍了如果用Vagrant来快速搭建一台虚拟机。但是所有的东西都停留在hello world的阶段，这篇文章就是结合这两个生产力工具来搭建一组~~可以实际使用~~用来做rails开发的server。

##Chef Provisioning
我们知道Vagrant包含多种不同类型的provisioner，而Chef就是其中内置的一种，所以我们是可以直接使用的。这里我们用到的是chef_solo, Chef Solo是Chef Client的单机版，运行不依赖于Chef Server。

第一步我们要下载我们需要安装的组件的cookbook，这里我们会用到第三方的cookbook，因为在[社区](https://supermarket.chef.io/cookbooks/)里已经有很多成熟的cookbook了。但是手工下载的方式不太方便，这里我们用到cookbook的依赖管理的工具: [Berkshelf](https://docs.chef.io/berkshelf.html)。它和Ruby的gem管理文件Gemfile语法类似，它的配置文件是Berksfile。我们在当前文件夹（我这里是**vagrant_test**）创建一个空的Berksfile。chef-dk里面已经包含Berkshelf的命令行工具了，对应的命令是**berks**, 但是Vagrant想要用Berkshelf的时候我们还要装个插件来实现，可以用下面的命令来安装最新Berkshelf的插件。
{% codeblock lang:bash%}
vagrant plugin install vagrant-berkshelf
{% endcodeblock %}

然后编辑Berksfile加上如下两行：
{% codeblock lang:ruby%}
source 'https://supermarket.chef.io'

cookbook 'rails', '~> 0.9.2'
{% endcodeblock %}
source语句指定了去supermarket.chef.io搜索下载cookbook, 第二行声明我们依赖于一个叫rails的cookbook，version为0.9.2，关于version的语法可以[戳这里](https://docs.chef.io/config_rb_metadata.html#cookbook-version-constraints)。

我们接着创建一个新的Vagrantfile, 这里我们打算创建两个虚拟机，一个作为rails的主server，另一个用来安装mysql。我们先看第一台虚拟机的配置：
{% codeblock lang:ruby%}
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  #base box for all machines
  config.vm.box = "bento/centos-6.7"

  #define a VM named rails
  config.vm.define "rails" do |rails|

    rails.vm.provider "virtualbox" do |v, override|
      v.name           = "rails"
      override.vm.network :private_network, ip: "192.168.50.20"
    end

    #configure chef provisoning per VM
    rails.vm.provision "shell", inline: "yum -y groupinstall 'Development Tools'"

    rails.vm.provision "chef_solo" do |chef|
      chef.node_name = "rails"
      chef.log_level = :debug
      chef.json = {
        "run_list" => ["rails"]
      }
    end
  end
end
{% endcodeblock %}
然后我们启动这台rails虚拟机，多虚拟机配置的启动需要指定虚拟机的名字，要不就是默认全部启动。
{% codeblock lang:bash%}
vagrant up rails
{% endcodeblock %}
这块儿由于是第一次启动这个虚拟机，它会首先下载需要的cookbook，然后映射到虚拟机里，启动虚拟机之后会首先下载安装Chef，这个过程很耗时间。

这里定义了两个provisioning，一个是用sh安装Rails依赖的开发工具包。另一个是利用rails这个cookbook来配置Rails的环境。
我这块遇到了在墙内会超时的问题。科学上网后解决了但是还是很耗时间，主要是下载Chef的安装包费时间。我们可以在其装好了chef和dev tools之后把这个box存起来作为以后的base box。命令如下:
{% codeblock lang:bash%}
vagrant package --base rails
vagrant box add mybase package.box 
{% endcodeblock %}
安装完box之后可以用下面的命令查看是否安装成功:
{% codeblock lang:bash%}
vagrant box list                  
bento/centos-6.7 (virtualbox, 2.2.7)
mybase           (virtualbox, 0)
{% endcodeblock %}
以后就可以用mybase作为box的name来复用已经下载安装好Chef和dev tools的box。

安装成功后我们可以用如下命令登录进去:
{% codeblock lang:bash%}
vagrant ssh rails
{% endcodeblock %}

然后我们定义另外一个虚拟机用来安装mysql，这里我们取名为mysql:
{% codeblock lang:ruby%}
config.vm.define "mysql" do |mysql|

    mysql.vm.provider "virtualbox" do |v, override|
      v.name           = "mysql"
      override.vm.network :private_network, ip: "192.168.50.21"
    end

    mysql.vm.provision "chef_solo" do |chef|
      chef.node_name = "mysql"
      chef.log_level = :debug
      chef.json = {
        "run_list" => ["vagrant_chef::mysql"]
      }
    end
 end
{% endcodeblock %}
因为对这个cookbook的调用不能直接通过runlist，需要在recipe里调用它的resource来进行，我们需要把当前目录变成一个cookbook。一个cookbook的最小因素就是一个用来自描述的metadata.rb
{% codeblock lang:ruby%}
name             "vagrant_chef"
maintainer       "YOUR_NAME"
maintainer_email "YOUR_EMAIL"
license          "All rights reserved"
description      "Installs/Configures mysql server"
version          "0.0.1"

depends "mysql"
{% endcodeblock %}
这里我们还要加上对mysql的依赖，只有这样才能保证使用mysql的resource之前对mysql的依赖都全部被加载好。然后我们需要在Berksfile里指定mysql的cookbook版本，这样Berkshelf插件才会把这个新依赖的cookbook抓下来:
{% codeblock lang:ruby%}
source 'https://supermarket.chef.io'
metadata

cookbook 'rails', '~> 0.9.2'

cookbook 'mysql', '~> 8.2.0'
{% endcodeblock %}
我们还加上了一行metadata，表示我们解析当前目录的metadata.rb, 把当前这个cookbook也加载进来。然后我们创建一个receipes文件夹，创建一个mysql.rb:
{% codeblock lang:ruby%}
mysql_service 'mysql' do
  port '3306'
  version '5.5'
  initial_root_password 'change me'
  action [:create, :start]
end
{% endcodeblock %}
这个Recipe用来安装设置mysql， 可以看到上面我们中对虚拟机的provisioning的配置里可以通过把**vagrant_chef::mysql**加到runlist来实现对这个Recipe的调用。
所有都做好我们可以调用
{% codeblock lang:bash%}
vagrant up mysql
{% endcodeblock %}
这样mysql这个虚拟机也会被创建设置好。

本来就想写个vagrant+chef的教程，结果大部分时间花在调试这两个别人写的cookbook。终于还是放弃了，所以这个教程的Chef安装部分是跑不起来的，仅供参考。但是其中你也能学到多个虚拟机的配置，组网，如何在Vagrant使用Chef, Berkshelf这些东西。

完整的Vagrantfile如下：

{% codeblock lang:ruby%}
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  #base box for all machines
  config.vm.box = "bento/centos-6.7"

  #define a VM named rails
  config.vm.define "rails" do |rails|

    rails.vm.provider "virtualbox" do |v, override|
      v.name           = "rails"
      override.vm.network :private_network, ip: "192.168.50.20"
    end

    #configure chef provisoning per VM
    rails.vm.provision "shell", inline: "yum -y groupinstall 'Development Tools'"

    rails.vm.provision "chef_solo" do |chef|
      chef.node_name = "rails"
      chef.log_level = :debug
      chef.json = {
        "run_list" => ["rails"]
      }
    end
  end

  config.vm.define "mysql" do |mysql|

    mysql.vm.provider "virtualbox" do |v, override|
      v.name           = "mysql"
      override.vm.network :private_network, ip: "192.168.50.21"
    end

    mysql.vm.provision "chef_solo" do |chef|
      chef.node_name = "mysql"
      chef.log_level = :debug
      chef.json = {
        "run_list" => ["vagrant_chef::mysql"]
      }
    end
  end
end

{% endcodeblock %}