# -*- mode: ruby -*-
# vi: set ft=ruby :

WORKER_COUNT = 2
HYPER_V_SWITCH = "Default Switch"
VAGRANT_BOX = "bento/fedora-40"
# VAGRANT_BOX_VERSION = "202404.23.0"
# VAGRANT_BOX = "bento/ubuntu-24.04"
VAGRANT_BOX_VERSION = "202407.22.0"

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

  # Define .kube directory for this project
  # Could have Ruby create this for me, but doing it in Ansible solved an odd issue I was having
  project_kubeconfig_path = "#{File.dirname(__FILE__)}/.kube/kubeconfig-admin"

  config.vm.define "control", primary: true do |control|
    control.vm.box = VAGRANT_BOX
    if !VAGRANT_BOX_VERSION.empty? 
      control.vm.box_version = VAGRANT_BOX_VERSION
    end 

    hostname = "vagrant-k8s-control"
    bridged_ip = "10.211.55.10"
    private_ip = "10.37.132.10"
    control.vm.hostname = hostname
    control.vm.synced_folder ".", "/vagrant", disabled: true

    ## Configure general private networking

    # NOTE: not letting Vagrant configure the networking because it doesn't do so correctly for Fedora
    # See https://github.com/hashicorp/vagrant/issues/12762
    control.vm.network "private_network", ip: private_ip, auto_config: false
    
    # Port Forward for API server and NGINX ingress
    control.vm.network "forwarded_port", guest: 6443, host: 6443
    control.vm.network "forwarded_port", guest: 443, host: 8443
    control.vm.network "forwarded_port", guest: 80, host: 8080
    
    # Hyper-V Specific Configuration
    control.vm.provider "hyperv" do |h, override|
      h.vmname = hostname

      # Not needed yet, but utilized differencing disk to speed up creation
      h.linked_clone = true
    
      # Set CPU and Memory to minimum needed for K8S
      h.cpus = 2
      h.memory = 4096

      # Set Hyper-V Switch to use; note Default Switch IPs aren't stable
      override.vm.network "public_network", bridge: HYPER_V_SWITCH
    end

    control.vm.provider "parallels" do |p, override|
      p.name = hostname

      p.cpus = 2
      p.memory = 4096

      # p.update_guest_tools = true

      # temporary addition while I figure/wait out Parallels Guest Tools to support Linux 6.6/Fedora 38 and greater
      # nolock is temporary as well while I try to figure out locking issues with macOS nfsd
      p.check_guest_tools = false
      override.vm.synced_folder ".", "/vagrant", type: "nfs", nfs_version: 3, nfs_udp: false, mount_options: ["nolock"]
    end

    # Provision node with playbooks
    control.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/k8s-control.yml"
      ansible.extra_vars = {
        project_kubeconfig_path: project_kubeconfig_path,
        advertise_ip: private_ip,
        private_ip: private_ip
      }
    end 
  end

  # Worker Node(s) Configuration
  (1..WORKER_COUNT).each do |i|
    config.vm.define "worker-#{i}" do |worker|
      worker.vm.box = VAGRANT_BOX
      if !VAGRANT_BOX_VERSION.empty? 
        worker.vm.box_version = VAGRANT_BOX_VERSION
      end 

      hostname = "vagrant-k8s-worker-#{i}"
      private_ip = "10.37.132.#{i + 10}"
      worker.vm.hostname = hostname
      worker.vm.synced_folder ".", "/vagrant", disabled: true

      
      # Configure general private networking
      # NOTE: not letting Vagrant configure the networking because it doesn't do so correctly for Fedora
      # See https://github.com/hashicorp/vagrant/issues/12762
      worker.vm.network "private_network", ip: private_ip, auto_config: false
      # Hyper-V Specific Configuration
      worker.vm.provider "hyperv" do |h, override|
        h.vmname = hostname

        # Not needed yet, but utilized differencing disk to speed up creation
        h.linked_clone = true
      
        # Set CPU and Memory to minimum needed for K8S
        h.cpus = 2
        h.memory = 4096

         # Set Hyper-V Switch to use; note Default Switch IPs aren't stable
        override.vm.network "public_network", bridge: HYPER_V_SWITCH
      end

      worker.vm.provider "parallels" do |p, override|
        p.name = hostname
  
        p.cpus = 2
        p.memory = 4096
        
        # p.update_guest_tools = true

        # temporary addition while I figure/wait out Parallels Guest Tools to support Linux 6.6/Fedora 38 and greater
        # nolock is temporary as well while I try to figure out locking issues with macOS nfsd
        p.check_guest_tools = false
        override.vm.synced_folder ".", "/vagrant", type: "nfs", nfs_version: 3, nfs_udp: false, mount_options: ["nolock"]
      end

      # Provision nodes with playbooks
      if i == WORKER_COUNT
        worker.vm.provision "ansible" do |ansible|
          ansible.playbook = "ansible/k8s-worker.yml"
          ansible.limit = (1..WORKER_COUNT).to_a.map { |i| "worker-#{i}" }
          ansible.extra_vars = {
            project_kubeconfig_path: project_kubeconfig_path,
          }
          ansible.host_vars = (1..WORKER_COUNT).to_h { |i| ["worker-#{i}", { "private_ip" => "10.37.132.#{i + 10}" }] }
        end
      end
    end
  end
end
