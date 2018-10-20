# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "debian/stretch64"

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 2
    libvirt.memory = 1024
  end

  config.vm.provider "virtualbox" do |virtualbox|
    virtualbox.cpus = 2
    virtualbox.memory = 1024
  end

  config.vm.network "forwarded_port", guest: 25, host: 2525
  config.vm.network "forwarded_port", guest: 143, host: 2143
  config.vm.network "forwarded_port", guest: 587, host: 2587

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "site.yml"
    ansible.config_file = "vagrant-ansible.cfg"
    ansible.groups = { "mta_servers" => ["default"], "mda_servers" => ["default"] }
    ansible.host_vars = { "default" => { "debug" => true } }
  end

end
