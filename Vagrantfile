# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define "IncludeOS" do |config|
    config.vm.box = "hklabbers/Ubuntu-18.04-Dev-40GB"
    config.vm.box_version = "2019.03.29"
    config.vm.provider :virtualbox do |vb|
      vb.name = "IncludeOS"
    end

    config.vm.synced_folder ".", "/vagrant", disable: true

    config.vm.synced_folder ".", "/IncludeOS", create: true
    config.vm.provision "shell",
                        inline: "echo cd /IncludeOS >> /home/vagrant/.bashrc"

    config.vm.synced_folder "~/IncludeOS_install",
                            "/home/vagrant/IncludeOS_install", create: true

    config.vm.provision "shell", inline: "apt-get update && apt-get install -qq git"
    config.vm.provision "shell", path: "etc/install_on_vagrant.sh", privileged: false
  end
end
