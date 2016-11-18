---
layout: post
title: "Vagrant 入门"
date: 2016-11-16 11:48:43 +0800
comments: true
categories: vagrant tool
keywords: vagrant
---


##介绍
Vagrant是一个虚拟机管理工具，通过它你可以用几个简单的命令快速搭建一个配置好的虚拟机环境。它提供了多种虚拟机provider所以你可以用同一个设置文件去驱动设置不同的虚拟机。<br><br>
它的核心文件叫做Vagrantfile，所有的虚拟机的配置都是在这里。我们可以在里面定义各个provider的配置，如果去用provisioner去设置虚拟机，安装应用程序。有了这个配置文件，你就可以很方便的快速重复构建一个测试环境。这让Docker出现之前成为了开发员很热衷的一个环境设置工具。
我这篇文章将会一步一步来搭建一个测试环境。这样从头来讲解Vagrant的基本使用方法，希望可以帮到那些想要学习Vagrant的人。

##初始设置
首先你需要去[下载](https://www.vagrantup.com/downloads.html)适合你操作系统的Vagrant安装包。安装完之后我们还需要安装[VirtualBox](https://www.virtualbox.org/)来作为演示用的provider。因为Virtualbox是Vagrant的provider，当然当你熟悉Vagrant之后可以通过修改全局的配置文件来修改默认的provider。一切都安装好了之后我们需要打开一个shell窗口，新建一个文件夹作为这次演示用的。
{% codeblock lang:bash%}
mkdir vagrant_leanring
cd vagrant_leanring 
vagrant init
{% endcodeblock %}
这里用到了第一个命令 *vagrant init* 这个命令会在当然目录生成如下这样的一个Vagrantfile，我们可以以这个为模板来修改运行我们自己的演示。
{% codeblock lang:ruby%}
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "base"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
end

{% endcodeblock %}
我们可以看到里面有一些很好的注释来帮助我们快速的生成自己需要的配置。这里能看到这个文件的格式是Ruby语法的，因为Vagrant这个工具是用Ruby来实现，所以为了更好的了解这个配置文件我们会在下一章讲解一下基本的Ruby语法。因为Ruby也是学习Chef需要的语言，我会在下一篇blog里讲解一下基础的Chef知识。
##Vagrant需要了解的Ruby语法
我们通过上面的Vagrantfile看到Ruby里面的注释是用*#*来开始的。然后我们看到了
{% codeblock lang:ruby%}
Vagrant.configure(2) do |config|
end
{% endcodeblock %}
这个其实是调用了Vagrant这个mudule上的configure方法，这个方法接受两个参数：第一个参数是2（表示Version），第二个参数是一个代码块(block)，因为根据Ruby的语法, 方法最后一个参数为block的时候我们可以放到括号外面，这种调用也等价于
{% codeblock lang:ruby%}
Vagrant.configure(2) { |config|
}
{% endcodeblock %}
这是block的两种表示方法，通过这种block的形式我们可以很方便的把一段代码作为参数传给一个方法。这里这个方法的作用就是去设置Vagrant的各项参数。我们可以看到被注释的代码里面也很多再次用到了嵌套的block的方式。<br><br>
需要更多的了解Ruby这门语言的话，这里有个很好的学习Ruby的[网站](http://ruby-doc.com/docs/ProgrammingRuby/)，虽然是英文的但是值得一读。
##Box
现在block的配置里唯一打开注释的就是
{% codeblock lang:ruby%}
config.vm.box = "base"
{% endcodeblock %}
Box是你的虚拟机启动时用的base image,所以你就不必从头去设置一个虚拟机。你可以去社区下载已经基本配置好的Box，然后基于它去搭建虚拟机。你可以去[HashiCorp's Atlas box catalog](https://atlas.hashicorp.com/boxes/search)去查看社区已有的Box。这个*base*就是其中一个Box，*base*是这个Box的id。我们可以用下面的命令去添加一个Box到我们的本地环境中：
{% codeblock lang:bash%}
vagrant box add hashicorp/precise32
{% endcodeblock %}
如果你用的OS是Mac的话，默认下载好的Box会被安装在*~/.vagrant.d/boxes*。然后我们修改Vagrantfile使用我们导入好的Box，因为base目前是不存在的：
{% codeblock lang:ruby%}
config.vm.box = "hashicorp/precise32"
{% endcodeblock %}
##Commands
 - *vagrant up* 是启动一个虚拟机，默认的话会在Virtualbox里导入你指定的Box，然后启动。

如果这时候你没有用*vagrant box add*导入Box的话，这时候Vagrant也会自动下载导入对于的Box。这个命令成功执行完毕之后你会在VirtualBox看到多了一个虚拟机。这个虚拟机的名字是*vagrant_learning_default_xxxxxx_xxx*。虚拟机名字的前面可以看出来是文件夹的名字，后面的default表示是默认的虚拟机因为一个Vagrantfile可以配置多个虚拟机，这个后面我们会看到。<br>
启动好了之后我们可以用*vagrant ssh*直接进到机器里。
与up相对应的命令有<br>

 - *vagrant destroy* 销毁删除一个创建好的虚拟机。一般我们彻底不用的时候才会这样做，更常用的
 - *vagrant halt* 关闭一个虚拟机，就是把虚拟机关机，这样的话就不会占用Cpu和内存。

还有不太常用的命令像 *vagrant suspend/resume*是挂起或者恢复虚拟机。更多的可以参考官方文档。

##Provider
Provider是Vagrant用来和不同种类虚拟机通信的驱动，我们在一个Vagrantfile可以配置多种Provider。安装好Vagrant好默认的Provider就是VirtualBox，所以我们在执行命令时不用传入Provider的名字。实际情况下我们有可能在一个Vagrantfile配置多个Provider来应对开发测试的不同阶段，例如我们可以配置一个VirtualBox的Provider做本地开发/测试，然后再定义一个EC2的Provider来部署到Amazon的云里，这样的配置就像下面这样:
{% codeblock lang:ruby%}

config.vm.provider "virtualbox" do |vb|
	# Display the VirtualBox GUI when booting the machine
  vb.gui = true

  # Customize the amount of memory on the VM:
  vb.memory = "1024"

  # Configure the virtual machine name displayed in VirtualBox
  vb.name = "vagrant_leanring"
end

config.vm.provider :aws do |aws,overide|
    aws.security_groups           =  %w(everything_open)
    aws.access_key_id             =  "YOUR KEY"
    aws.secret_access_key         =  "YOUR SECRET KEY"
    aws.ami                       =  "ami-xxx"
    aws.instance_type             =  "m1.medium"
    aws.keypair_name              =  "KEYPAIR NAME"

    overide.ssh.username              =  "root"
    overide.ssh.private_key_path      =  "PATH TO YOUR PRIVATE KEY" 
    overide.vm.box                    =  'dummy'
end

{% endcodeblock %}

这样配置以后你可以用*vagrant up --provider=aws*来远程启动AWS云里的虚拟机。当然之前你需要安装AWS的[Provider](https://github.com/mitchellh/vagrant-aws)和配置这个Provider需要的dummy Box。
{% codeblock lang:bash%}
vagrant plugin install vagrant-aws

vagrant box add dummy https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box
{% endcodeblock %}
接下来的教程我们还是以VirtualBox这个Provider为例来讲解。

##Shared folders
我们接下来可以打开注释里的*config.vm.synced_folder*并修改映射到已经存在的一个文件夹。以我的机器为例：
{% codeblock lang:ruby%}
config.vm.synced_folder "../training", "/vagrant_data"
{% endcodeblock %}
这样的话可以把本机Host里的一个文件夹映射到Guest虚拟机里。修改这个设置之后需要重启虚拟机来生效，命令是
{% codeblock lang:ruby%}
vagrant reload
{% endcodeblock %}
这样启动完之后登陆后你可以看到映射的文件夹:
{% codeblock lang:bash%}
vagrant ssh

vagrant@precise32:~$ ll /vagrant_data/
total 16
drwxr-xr-x  1 vagrant vagrant  238 Jul  4 10:40 ./
drwxr-xr-x 24 root    root    4096 Nov 18 02:25 ../
drwxr-xr-x  1 vagrant vagrant  170 Jul  4 10:39 cookbooks/
drwxr-xr-x  1 vagrant vagrant  272 Jan  8  2016 Docker/
-rw-r--r--  1 vagrant vagrant 8196 Jul  4 10:40 .DS_Store
drwxr-xr-x  1 vagrant vagrant  102 Jul  4 10:39 intro/
drwxr-xr-x  1 vagrant vagrant  102 Apr  5  2016 .vagrant/
{% endcodeblock %}
##Network
默认的虚拟机的网络是NAT形式的，我们可以从Vagrant启动的log里看出来：
{% codeblock lang:bash%}
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 => 2202 (adapter 1)
{% endcodeblock %}
我们可以看到默认做了一个端口映射，所以我们可以通过访问本机端口2202来访问虚拟机的22(ssh)端口:
{% codeblock lang:bash%}
ssh vagrant@127.0.0.1 -p 2202
The authenticity of host '[127.0.0.1]:2202 ([127.0.0.1]:2202)' can't be established.
ECDSA key fingerprint is SHA256:q7StLmS/+YYwF42lL4HQJdMQGcAPpkKgzVlxORTisGE.
Are you sure you want to continue connecting (yes/no)? yes

vagrant@127.0.0.1's password: 

Last login: Fri Nov 18 02:26:09 2016 from 10.0.2.2
vagrant@precise32:~$ 
{% endcodeblock %}
这个就是vagrant ssh的实现原理，默认的用户名是*vagrant*密码也是*vagrant*。这样的网络结构做一些简单的部署测试可以的，比方说我们可以定义本机的8080端口映射到虚拟机的80端口：
{% codeblock lang:ruby%}
config.vm.network "forwarded_port", guest: 80, host: 8080
{% endcodeblock %}
修改网络拓扑结构和配置之后也有重启虚拟机。从启动log我们可以看到端口成功映射了
{% codeblock lang:bash%}
==> default: Forwarding ports...
    default: 80 => 8080 (adapter 1)
    default: 22 => 2202 (adapter 1)
==> default: Running 'pre-boot' VM customizations...
{% endcodeblock %}
还有一种常用的网络结构是*private_network*，这种网络结构会创建一个只有主机能和虚拟机通信的私网，这种是通过指定虚拟机内网IP的形式来配置:
{% codeblock lang:ruby%}
config.vm.network "private_network", ip: "192.168.33.10"
{% endcodeblock %}
这样的配置启动后，Host本机会拥有IP *192.168.33.1*, 这样本机和虚拟机就是一个网段，通信就没有障碍了。最后一种形式是*public_network*,这种就是为虚拟机建一个Bridge的网络，相当于虚拟机变成你现有网络里另一台机器。但是这种是受限于网络要求的，像我们公司就是不允许这种网络结构的，你的虚拟机得不到IP。<br>
当然这里网络配置都是针对本地的虚拟机，像是AWS，Openstack这种云里的虚拟机所在自己的Provider配置里来配置的。一般网络结构受限于云和Provider插件支持的类型。
##Provisioner
现在虚拟机已经启动好了，我们需要配置,部署一些软件上去，这就用到了Provisioner。Vagrantfile也支持多个Provisioner，它们会按配置顺序挨个执行。例如我们可以打开注释里的：
{% codeblock lang:ruby%}
 config.vm.provision "shell", inline: <<-SHELL
   sudo apt-get update
   sudo apt-get install -y apache2
 SHELL
{% endcodeblock %}
修改完之后我们又要用到一个新命令:
{% codeblock lang:bash%}
vagrant provision
{% endcodeblock %}
这个命令会执行所有定义的provision命令block。之后我们会在控制台看到很长的一段输出，成功执行之后我们就装好了Apache, 我们可以打开浏览器试一试。因为我们已经配置好了端口映射，所以访问本机的8080端口应该能到Apache的欢迎页面。

>It works!

>This is the default web page for this server.

>The web server software is running but no content has been added, yet.

<br>
这篇教程用到的完整Vagrantfile（去掉了一些注释）如下:
{% codeblock lang:ruby%}
v# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "hashicorp/precise32"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "../training", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = true

    # Customize the amount of memory on the VM:
    vb.memory = "1024"

    # Configure the virtual machine name displayed in VirtualBox
    vb.name = "vagrant_leanring"
  end

  config.vm.provider :aws do |aws,overide|
      aws.security_groups           =  %w(everything_open)
      aws.access_key_id             =  "YOUR KEY"
      aws.secret_access_key         =  "YOUR SECRET KEY"
      aws.ami                       =  "ami-xxx"
      aws.instance_type             =  "m1.medium"
      aws.keypair_name              =  "KEYPAIR NAME"

      overide.ssh.username              =  "root"
      overide.ssh.private_key_path      =  "PATH TO YOUR PRIVATE KEY" 
      overide.vm.box                    =  'dummy'
  end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo apt-get install -y apache2
  SHELL
end

{% endcodeblock %}
<br><br>
这篇教程到这就告一段落了，我想我还会写第二部分讲述如何用一个Vagrantfile配置多个虚拟机，然后讲完Chef后还会写一个结合Vagrant和Chef的教程。
