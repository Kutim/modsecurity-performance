# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  mem  = 1024
  cpus = 2
  host = RbConfig::CONFIG['host_os']

  # provide more resources to vm
  if host =~ /darwin/
    mem = [4096, `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 2].min
  elsif host =~ /linux/
    cpus = [2, `getconf _NPROCESSORS_ONLN`.to_i / 2].max
    mem = [cpus * 512, `awk '/MemTotal/ {print $2}' /proc/meminfo`.to_i / 1024 / 2].min
  end

  config.vm.box = "ubuntu/zesty64"
  config.vm.hostname = "vagrant"

  # use NFS for performant file sharing
  config.vm.network :private_network, ip: '192.168.50.50'
  config.vm.synced_folder ".", "/vagrant", type: "nfs", nfs_version: 3

  # mount states and pillars to the appropriate locations
  config.vm.synced_folder "states/", "/srv/salt/", type: "nfs", nfs_version: 3
  config.vm.synced_folder "pillars/", "/srv/pillar/", type: "nfs", nfs_version: 3

  # provider specific settings
  config.vm.provider "virtualbox" do |vbox, override|
    vbox.cpus = cpus
    vbox.memory = mem
  end

  config.vm.provider :libvirt do |libvirt, override|
    libvirt.cpus = cpus
    libvirt.memory = mem
    libvirt.driver = "kvm"
    libvirt.cpu_mode = "host-passthrough"
  end

  config.vm.provision :salt do |salt|
    salt.minion_config     = "minion"
    salt.install_type      = "stable"
    salt.bootstrap_options = "-n -X"
    salt.masterless        = true
    salt.run_highstate     = true
  end

  config.vm.define "default", primary: true do |default|
    default.vm.provider :libvirt do |libvirt, override|
      override.vm.box = "wholebits/ubuntu17.04-64"
    end
  end

  config.vm.define "debian", autostart: false do |debian|
    debian.vm.box = "debian/jessie64"
  end
end
