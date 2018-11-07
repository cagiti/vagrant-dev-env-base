# -*- mode: ruby -*-
# vi: set ft=ruby :

# Check and Install required plugins if missing
installed_plugins = false
required_plugins=%w( vagrant-vbguest vagrant-reload vagrant-cachier vagrant-env )
required_plugins.each do |plugin|
  if !Vagrant.has_plugin?plugin
    system "vagrant plugin install #{plugin}"
    installed_plugins = true
  end
end

if installed_plugins
  puts "Please re-run 'vagrant up' command"
  exit
end

Vagrant.require_version ">= 2.2.0"

Vagrant.configure(2) do |config|
  config.trigger.before :up do |trigger|
    trigger.name = "Start"
    trigger.info =
        "******************************************************\n" +
        "* WARNING!!! Do not login to the dev-env             *\n" +
        "* before provisioning completes!!                    *\n" +
        "******************************************************\n"
  end

  config.trigger.after :up do |trigger|
    trigger.name = "Finished"
    trigger.info =
        "******************************************************\n" +
        "* You can now login to your dev-env                  *\n" +
        "* The username and password are both 'vagrant'       *\n" +
        "******************************************************\n"
  end

  # Latest version available at https://app.vagrantup.com/finkit/boxes/development-environment-base
  config.vm.box_url="https://app.vagrantup.com/peru/boxes/ubuntu-18.04-server-amd64"
  config.vm.box = "peru/ubuntu-18.04-server-amd64"
  config.vm.box_version = "20181101.01"

  config.vm.hostname = "dev-env"
  config.vm.network "private_network", type: "dhcp"

  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = true
    config.vbguest.iso_path = "http://download.virtualbox.org/virtualbox/%{version}/VBoxGuestAdditions_%{version}.iso"
  end

  if Vagrant.has_plugin?("vagrant-cachier")
    # The vagrant-cachier plugin (optional) will speed up rebuilds by reusing downloaded artifacts
    # Configure cached packages to be shared between instances of the same base box.
    config.cache.scope = :box
  end

  # Run Ansible from the Vagrant VM
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "ansible/main.yml"
    ansible.install_mode = "pip"
    ansible.galaxy_role_file = "ansible/requirements.yml"
  end

  config.vm.provision :reload

  config.vm.provider "virtualbox" do |v|
    v.gui = false
    v.name = "dev-env"
    v.customize ["modifyvm", :id, "--cpus", ENV['DEVENV_PROCESSORS'] || "4"]
    v.customize ["modifyvm", :id, "--cpuexecutioncap", ENV['DEVENV_CPUEXECUTIONCAP'] || "100"]
    v.customize ["modifyvm", :id, "--monitorcount", "1"]
    v.customize ["modifyvm", :id, "--memory", ENV['DEVENV_MEMORY'] || "8192"]
    v.customize ["modifyvm", :id, "--vram", "128"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
    v.customize ["modifyvm", :id, "--accelerate3d", "on"]
    v.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
    v.customize ["guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/ --timesync-set-threshold", 10000]
  end
  config.vm.synced_folder ".", "/vagrant", nfs_export: true
  config.vm.synced_folder "~/.dev_env", "/dev_env", nfs_export: true, create: true
end
