# encoding: utf-8
# -*- mode: ruby -*-
# vi: set ft=ruby :
# Box / OS
VAGRANT_BOX = 'hashicorp/bionic64'
# Memorable name for your
VM_NAME = 'vagrant'
# VM User — 'vagrant' by default
VM_USER = 'vagrant'
# Username on your Mac
MAC_USER = 'User'
# Host folder to sync
HOST_PATH = '\Users\User\oz-vagrant'
# Where to sync to on Guest — 'vagrant' is the default user name
GUEST_PATH = '/home/' + VM_USER + '/' + VM_NAME
# # VM Port — uncomment this to use NAT instead of DHCP
# VM_PORT = 8080
Vagrant.configure(2) do |config|
  # Vagrant box from Hashicorp
  config.vm.box = VAGRANT_BOX
  # Actual machine name
  config.vm.hostname = "consul-server"
  # Set VM name in Virtualbox
  config.vm.provider "virtualbox" do |v|  
    v.name = "consul-server"
    v.memory = 1024
    v.cpus = 1
  end
  #DHCP — comment this out if planning on using NAT instead
  config.vm.network "private_network", type: "dhcp"
  # # Port forwarding — uncomment this to use NAT instead of DHCP
  # config.vm.network "forwarded_port", guest: 80, host: VM_PORT
  # Sync folder
  config.vm.synced_folder HOST_PATH, GUEST_PATH
  # Disable default Vagrant folder, use a unique path per project
  config.vm.synced_folder '.', '/home/'+VM_USER+'', disabled: true
  # Install Git, Node.js 6.x.x, Latest npm
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y git
    apt-get install -y consul
    curl -fsSL https://apt.releases.hashicorp.com/gpg | apt-key add -
    apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
    apt-get update && sudo apt-get install consul
    apt-get update && sudo apt-get install consul-enterprise.x86_64
    sudo apt-get install unzip
    mv /home/vagrant/vagrant/counting-service_linux_amd64.zip* /home/vagrant/counting-service.zip
    mv /home/vagrant/vagrant/dashboard-service_linux_amd64.zip* /home/vagrant/dashboard-service.zip
    unzip counting-service.zip*
    unzip dashboard-service.zip*
    mv dashboard-service_linux_amd64* dashboard-service
    mv counting-service_linux_amd64* counting-service
    mkdir ./consul.d
    
    echo '{
    "service": {
      "name": "web",
      "tags": [
        "rails"
      ],
      "port": 80,
      "check": {
        "args": [
          "curl",
          "localhost"
        ],
        "interval": "10s"
      }
    }
  }' > ./consul.d/web.json
    
    echo 'service {
      name = "counting"
      id = "counting-1"
      port = 9003

      connect {
        sidecar_service {}
      }

      check {
        id       = "counting-check"
        http     = "http://localhost:9003/health"
        method   = "GET"
        interval = "1s"
        timeout  = "1s"
      }
    }' >./counting.hcl
  
    
    echo  'service {
      name = "dashboard"
      port = 9002

      connect {
        sidecar_service {
          proxy {
            upstreams = [
              {
                destination_name = "counting"
                local_bind_port  = 5000
              }
            ]
          }
        }
      }

      check {
        id       = "dashboard-check"
        http     = "http://localhost:9002/health"
        method   = "GET"
        interval = "1s"
        timeout  = "1s"
      }
    }' >./dashboard.hcl
    consul agent -dev -enable-script-checks -config-dir=./consul.d &
    consul services register counting.hcl
    consul services register counting.hcl
    consul services register dashboard.hcl 
    consul intention create dashboard counting
    PORT=9002 COUNTING_SERVICE_URL="http://localhost:5000" ./dashboard-service &
    PORT=9003 ./counting-service &
    consul connect proxy -sidecar-for counting-1 > counting-proxy.log &
    consul connect proxy -sidecar-for dashboard > dashboard-proxy.log &
  SHELL
end