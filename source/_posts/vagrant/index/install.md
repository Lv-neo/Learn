---
title: 使用vagrant构建开发测试环境
date: 2018-12-17 19:15:26
categories: vagrant
tags: vagrant
---

# 使用vagrant构建开发测试环境

## 安装virtualBox和vagrant
[VirtualBox官方下载](https://www.virtualbox.org/wiki/Downloads)
[Vargant官方下载](https://www.vagrantup.com/downloads.html)

## 下载一个box

笔者选择的centos7.2
[vagrantup](https://www.vagrantup.com/docs/boxes.html)
[vagrantbox](http://www.vagrantbox.es/)

>速度太慢，请用离线下载或者下载工具，国内也有镜像资源

## 添加box，初始化vagrant box

选择一个目录,开始构建

```
$ cd ~/project/

$ vagrant box add /Users/Neo/Downloads/macTools/vagrant-centos-7.2.box --name centos7.2
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'centos7.2' (v0) for provider: 
    box: Unpacking necessary files from: file:///Users/Neo/Downloads/macTools/vagrant-centos-7.2.box
==> box: Successfully added box 'centos7.2' (v0) for 'virtualbox'!

$ vagrant box list
centos7.2 (virtualbox, 0)

$ vagrant init centos7.2
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.


```

这样目录中就会多出一个文件叫 Vagrantfile 所有的机关也就都在这里了，比如这里面有一句 config.vm.box = "centos7.2"这样


>我不用线上的原因是太慢，如果是团队开发，一个人构建好之后丢到服务器上最合适

## 执行vagrant up

```
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'centos7.2'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: project_default_1494056264311_10321
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
==> default: Forwarding ports...
    default: 22 => 2222 (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Connection timeout. Retrying...
    default: Warning: Remote connection disconnect. Retrying...
    default: 
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default: 
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default: 
    default: Guest Additions Version: 4.3.30
    default: VirtualBox Version: 5.0
==> default: Mounting shared folders...
    default: /vagrant => /Users/Neo/project
```

> 如果你是用的网上的box，可能需要十几分钟以上，建议是放到内网或者本地

## 访问虚拟机

```
$  vagrant ssh
[vagrant@localhost ~]$ uname -r
3.10.0-327.4.5.el7.x86_64
```


我们已经进入虚拟机了，登陆进来的用户名是 vagrant，这个用户挺好，执行 sudo 是不需要输入密码的，开发中实际使用挺好用的。不过如果我非要创建自己的用户也是可以的。

## 修改配置

关键步骤来了，vagrant默认配置不满足我们的需求


### 修改内存

```
v.memory = 1024
```

> 你也可以进入VirtualBox修改，不过VagrantFile都可以搞定


### 修改网络

网络模式有3种

* forwarded_port
* private_network
* public_network

很简单，按照你的需求设置，个人开发环境建议设置为private_network


```
config.vm.network "private_network", ip: "192.168.222.111"
```


>协同开发，大家可以下载使用同一个VagrantFile，大部分配置相同，是不是很爽



### 修改共享目录

我们需要制定共享目录的用户和权限，开发的代码就在共享目录里了。

在这之前必须进入vagrant增加你需要的用户www，如果你直接用vagrant，就不需要


```
config.vm.synced_folder "/Users/Neo/project", "/vagrant",create:true, :owner => "www", :group => "www", :mount_options => ["dmode=775","fmode=664"]
```


### 修改登录

我比较喜欢crt或者xshell统一管理机器,在结尾加上

```
  config.ssh.username = 'vagrant'
  config.ssh.password = 'vagrant'
  config.ssh.insert_key = 'true'
```

### 重启


```
vagrant reload
```


### ssh免密登录

```
# 现在我们进入vagrant 账号root密码vagrant

[root@localhost ~]# cd ~/
[root@localhost ~]# mkdir .ssh
# 纯净的没vim，加上自己的sshkey
[root@localhost ~]# vi authorized_keys
# 保存退出
```


在通过crt Publickey方式，完美无密码进入


## 打包box

装完你（团队）需要的环境之后，退出vagrant，我们打包成一个开发环境的box供大家使用

```
vagrant package --output xxxx.box --vagrantfile file
```


## 获取远程仓库

```
vagrant box add centos7.2 http://127.0.0.1/xxxx.box
```


## 附录：完整的vagrantFile

```
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
  config.vm.box = "centos7.2"

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
  config.vm.network "private_network", ip: "192.168.222.111"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "/Users/Neo/project", "/vagrant",create:true, :owner => "www-data", :group => "www-data", :mount_options => ["dmode=775","fmode=664"]

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
  config.ssh.username = 'vagrant'
  config.ssh.password = 'vagrant'
  config.ssh.insert_key = 'true'
end

```

