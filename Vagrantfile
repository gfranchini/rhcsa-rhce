# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  # Path where the extra disks should be stored when using VirtualBox
  vbox_vm_path = "" # VirtualBox ONLY!

  # Storage pool where the extra disks should be stored when using libVirt
  libvirt_storage_pool = "" # libVirt ONLY!

  # Memory configuration
  ipaserver_memory = 2048 # Recommended: 2048 MiB / Minimum: 512 MiB
  server_memory = 1024 # Recommended: 1024 MiB / Minimum: 512 MiB
  desktop_memory = 1024 # Recommended: 1024 MiB / Minimum: 512 MiB
  extra_disk_size = 2 # Recommended: 10 GiB / Minimum: 2 GiB

  # VM guest configuration
  config.vm.box = "centos/7"

  # Don't modify beyond here
  conname = "Wired connection 1"
  devname = "eth1"

  config.vm.provider "virtualbox" do |vbox, override|
    # vbox.gui = true
    if ENV['VBOX_VM_PATH']
      vbox_vm_path = ENV['VBOX_VM_PATH']
    end
  end

  config.vm.provider "libvirt" do |libvirt, override|
    if ENV['LIBVIRT_STORAGE_POOL']
      libvirt_storage_pool = ENV['LIBVIRT_STORAGE_POOL']
    end
    libvirt.driver = "kvm"
    libvirt.storage_pool_name = libvirt_storage_pool
  end

  config.vm.provision :shell, path: "scripts/default-provision"

  config.vm.define :ipaserver do |ipaserver_config|
    ipaserver_config.vm.hostname = "ipaserver.example.com"
    ipaserver_config.vm.network "private_network", ip: "172.25.0.254", auto_config: false
    ipaserver_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 172.25.0.254/24 ipv4.dns 172.25.0.254,8.8.8.8 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    ipaserver_config.vm.provision :shell, path: "scripts/ipa-provision"
    ipaserver_config.vm.provider "virtualbox" do |vbox, override|
      vbox.name = ipaserver_config.vm.hostname
      vbox.cpus = 1
      vbox.memory = ipaserver_memory
    end
    ipaserver_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = ipaserver_memory
    end
  end

  config.vm.define :server do |server_config|
    server_config.vm.hostname = "server.example.com"
    server_config.vm.network "private_network", ip: "172.25.0.11", auto_config: false
    server_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 172.25.0.11/24 ipv4.gateway 172.25.0.254 ipv4.dns 172.25.0.254 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    server_config.vm.network "private_network", ip: "172.25.0.12", auto_config: false
    server_config.vm.provision :shell, path: "scripts/server-provision"
    server_config.vm.provider "virtualbox" do |vbox, override|
      vbox.name = server_config.vm.hostname
      vbox.cpus = 1
      vbox.memory = server_memory
      if !File.exist?(vbox_vm_path + 'rhel_server_2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'rhel_server_2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'IDE', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'rhel_server_2.vdi']
    end
    server_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = server_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  end

  config.vm.define :client do |client_config|
    client_config.vm.hostname = "client.example.com"
    client_config.vm.network "private_network", ip: "172.25.0.21", auto_config: false
    client_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 172.25.0.21/24 ipv4.gateway 172.25.0.254 ipv4.dns 172.25.0.254 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    client_config.vm.network "private_network", ip: "172.25.0.22", auto_config: false
    client_config.vm.provision :shell, path: "scripts/server-provision"
    client_config.vm.provider "virtualbox" do |vbox, override|
      vbox.name = client_config.vm.hostname
      vbox.cpus = 1
      vbox.memory = server_memory
      if !File.exist?(vbox_vm_path + 'rhel_client_2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'rhel_client_2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'IDE', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'rhel_client_2.vdi']
    end
    client_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = server_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  end

  config.vm.define :desktop do |desktop_config|
    desktop_config.vm.hostname = "desktop.example.com"
    desktop_config.vm.network "private_network", ip: "172.25.0.10", auto_config: false
    desktop_config.vm.provision :shell, run: "always", inline: "(nmcli device connect '#{devname}' &) && sleep 10 && nmcli con modify '#{conname}' ipv4.addresses 172.25.0.10/24 ipv4.gateway 172.25.0.254 ipv4.dns 172.25.0.254 ipv4.route-metric 10 ipv4.method manual && nmcli con up '#{conname}'"
    desktop_config.vm.provision :shell, path: "scripts/desktop-provision"
    desktop_config.vm.provider "virtualbox" do |vbox, override|
      vbox.name = desktop_config.vm.hostname
      vbox.cpus = 1
      vbox.memory = desktop_memory
      if !File.exist?(vbox_vm_path + 'rhel_desktop_2.vdi')
        vbox.customize ['createhd', '--filename', vbox_vm_path + 'rhel_desktop_2.vdi', '--variant', 'Fixed', '--size', extra_disk_size * 1024]
      end
      vbox.customize ['storageattach', :id,  '--storagectl', 'IDE', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', vbox_vm_path + 'rhel_desktop_2.vdi']
    end
    desktop_config.vm.provider "libvirt" do |libvirt, override|
      libvirt.cpus   = 1
      libvirt.memory = desktop_memory
      libvirt.storage :file, :size => extra_disk_size.to_s + 'G'
    end
  end

end
