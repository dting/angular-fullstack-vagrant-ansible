# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"

  # Uncomment to use hostmanager plugin
  # config.hostmanager.enabled = true
  # config.hostmanager.manage_host = true
  # config.hostmanager.ignore_private_ip = false

  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--memory", "1024"]
  end

  config.vm.define "web" do |web|
    # On host > ~/.ssh/ssh-add -K
    web.ssh.forward_agent = true

    web.vm.network "forwarded_port", guest: 9000, host: 9000, nic_type: "82545EM"
    web.vm.network "forwarded_port", guest: 35729, host: 35729, nic_type: "82545EM"

    # Add to host > ~/.ssh/config
    #   Host 192.168.111.222
    #     ForwardAgent yes
    web.vm.network :private_network, ip: "192.168.111.222"
    web.vm.hostname = "web.vagrant"

    web.vm.synced_folder ".", "/vagrant",
        type: "nfs",
        mount_options: ["nolock", "vers=3", "tcp"]
      web.nfs.map_uid = Process.uid
      web.nfs.map_gid = Process.gid

    web.vm.provision "ansible" do |ansible|
      ansible.playbook = "provisioning/web.yml"
    end
  end

  config.vm.define "db" do |db|
    db.vm.network :private_network, ip: "192.168.111.223"
    db.vm.hostname = "db.vagrant"

    db.vm.provision "ansible" do |ansible|
      ansible.playbook = "provisioning/db.yml"
    end
  end
end
