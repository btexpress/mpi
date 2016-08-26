# -*- mode: ruby -*-
# vi: set ft=ruby :

domain = 'example.com'
box = 'btexpress/centos7'
#psram = 8192
#gitram = 2048
#pscpu = 4
#gitcpu = 1
psram = 3072
gitram = 1024
pscpu = 2
gitcpu = 1

$installupdatesandsw = <<INSTALLUPDATESANDSW
    yum update -y
    yum install lsof deltarpm git net-tools vim-enhanced tree -y
INSTALLUPDATESANDSW

$installpuppet = <<INSTALLPUPPET
    rpm -ivh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
    yum install puppet -y
INSTALLPUPPET

$installpuppetserver = <<INSTALLPUPPETSERVER
    yum install puppetserver -y
INSTALLPUPPETSERVER

servers = [
	{
	:hostname => 'pupserv',
	:ip => '172.16.32.10',
	:box => box,
	:ram => psram,
	:cpu => pscpu,
	:fwdhost => 8140,
	:fwdguest => 8140,
	},
	{
	:hostname => 'gitserv',
	:ip => '172.16.32.11',
	:box => box,
	:ram => gitram,
	:cpu => gitcpu,
	},
]

Vagrant.configure("2") do |config|
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = false
  config.hostmanager.manage_guest = true
  config.hostmanager.ignore_private_ip = false
#  config.hostmanager.include_offline = true
  servers.each do |machine|

    config.vm.define machine[:hostname] do |node|
      node.vm.box = machine[:box]
      node.vm.box_url = 'https://atlas.hashicorp.com/' + node.vm.box
      node.vm.hostname = machine[:hostname] + '.' + domain
      node.vm.network :private_network, ip: machine[:ip]

      if machine[:fwdhost]
        node.vm.network :forwarded_port, guest: machine[:fwdguest], host: machine[:fwdhost]
      end

      node.vm.provider :virtualbox do |vb|
        vb.customize [
          'modifyvm', :id,
          '--name', machine[:hostname],
          '--memory', machine[:ram],
          '--cpus', machine[:cpu],
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
