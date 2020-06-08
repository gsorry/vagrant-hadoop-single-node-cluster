# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.network "private_network", ip: "192.0.2.20"
  config.vm.provider "virtualbox" do |vb|
    vb.name = "node-single"
    vb.memory = "8192"
    vb.cpus = 6
  end
  config.vm.provision "shell", inline: <<-SHELL
    echo "Welcome Single!"
  SHELL
end
