# -*- mode: ruby -*-
# vi: set ft=ruby :

WORKER_COUNT = 1
NETWORK_SWITCH_NAME = ""

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Turning off default key injection out of laziness/not wanting to deal with WSL file permissions caveats
  # Tl;DR Hyper-V needs this to be mounted in the Windows filesystem but that doesn't support the correct Metadata
  config.ssh.insert_key = false

  # TODO: since this is Ruby, see if I can set up to either generate or warn if keys aren't present
  # also how to synchronize with playbook?
  # NOTE: the insecure key is used just to bootstrap, the playbook will remove it afterwards
  config.ssh.private_key_path = ["#{Dir.home}/.ssh/vagrant-k8s-experiments", "#{Dir.home}/.vagrant.d/insecure_private_key"]

  # Control Plane/Master Node Resource Configuration
  config.vm.define "control", primary: true do |control|
    control.vm.box = "bento/ubuntu-20.04"

    control.vm.network "forwarded_port", id: "ssh", host: 50241, guest: 22

    # Hyper-V Specific Configuration
    control.vm.provider "hyperv" do |h|
      # Not needed yet, but utilized differencing disk to speed up creation
      h.linked_clone = true
     
      # Set CPU and Memory to minimum needed for K8S
      h.cpus = 2
      h.memory = 4096

      # Explicitly Configure Network/MAC address to try to get stable IPs
      control.vm.network "public_network", bridge: "Kubernetes", ip: "169.254.0.10"
      h.mac = "00155D5001E0" # Seems needs to be in the range hyper-V will assign it
    end

    # Provision node with playbooks
    control.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/k8s-cluster.yml"
    end
  end

  # Worker Node(s) Configuration
  (1..WORKER_COUNT).each do |i|
    config.vm.define "worker-#{i}" do |worker|
      worker.vm.box = "bento/ubuntu-20.04"
    
      worker.vm.network "forwarded_port", id: "ssh", host: 50251 + i, guest: 22

      # Hyper-V Specific Configuration
      worker.vm.provider "hyperv" do |h|
        # Not needed yet, but utilized differencing disk to speed up creation
        h.linked_clone = true
      
        # Set CPU and Memory to minimum needed for K8S
        h.cpus = 2
        h.memory = 4096

        # Explicitly Configure Network/MAC address to try to get stable IPs
        worker.vm.network "public_network", bridge: "Kubernetes", ip: "169.254.0.#{i + 20}"
        h.mac = "00155D5002%02x" % i # Seems needs to be in the range hyper-V will assign 
      end

      # Provision node with playbooks
      worker.vm.provision "ansible" do |ansible|
        ansible.playbook = "ansible/k8s-cluster.yml"
      end
    end
  end


  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080E

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
