# -*- mode: ruby -*-
# vi: set ft=ruby :

domain = 'example.com'
box = 'btexpress/centos7'
psram = 8192
gitram = 2048

$installupdatesandsw = <<INSTALLUPDATESANDSW
    yum update -y
    yum install git -y
    yum install net-tools -y
INSTALLUPDATESANDSW

$installpuppet = <<INSTALLPUPPET
    rpm -ivh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
    yum install puppet -y
INSTALLPUPPET

$installpuppetserver = <<INSTALLPUPPETSERVER
    yum install puppetserver -y
INSTALLPUPPETSERVER

puppet_nodes = [
  {:hostname => 'pupserv',  :ip => '172.16.32.10', :box => box, :fwdhost => 8140, :fwdguest => 8140, :ram => psram},
  {:hostname => 'gitserv',  :ip => '172.16.32.11', :box => box, :ram => gitram},
]

Vagrant.configure("2") do |config|
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = false
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
#  config.hostmanager.include_offline = true
  puppet_nodes.each do |node|

    config.vm.define node[:hostname] do |node_config|
      node_config.vm.box = node[:box]
      node_config.vm.box_url = 'https://atlas.hashicorp.com/' + node_config.vm.box
      node_config.vm.hostname = node[:hostname] + '.' + domain
      node_config.vm.network :private_network, ip: node[:ip]

      if node[:fwdhost]
        node_config.vm.network :forwarded_port, guest: node[:fwdguest], host: node[:fwdhost]
      end

      memory = node[:ram] ? node[:ram] : 256;
      node_config.vm.provider :virtualbox do |vb|
        vb.customize [
          'modifyvm', :id,
          '--name', node[:hostname],
          '--memory', memory.to_s
        ]
      end
      config.vm.provision "shell", inline:  $installupdatesandsw
      config.vm.provision :reload
    end
    config.vm.define "pupserv" do |pupserv|
        pupserv.vm.provision "shell", inline:  $installpuppet
    	pupserv.vm.provision "shell", inline:  $installpuppetserver
	end
  end
end
