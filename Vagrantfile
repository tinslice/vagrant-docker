# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV["LC_ALL"] = "en_US.UTF-8"

Vagrant.configure("2") do |config|
  config.vagrant.plugins = [ "nugrant", "vagrant-docker-compose", "vagrant-vbguest", "vagrant-disksize", "vagrant-docker-compose", "vagrant-reload"]
  
  config.user.defaults = {
    'services' => {
      'vm_name' => "docker-services",
      'config_location'=> './'
    },
    'docker' => {
      'registry' => "",
      'user' => "",
      'password' => ""
    }
  }

  config.vm.provision "default packages", type: "shell", preserve_order: true,inline: <<-SHELL
    apt-get update && apt-get -y upgrade 
    sudo apt-get -y install apt-transport-https lsb-release ca-certificates gnupg2 dirmngr software-properties-common curl jq nmap rsync joe vim mc tmux git build-essential vnstat htop xclip xsel
  SHELL

  config.vm.provision :docker, preserve_order: true, run: "always"
  config.vm.provision :docker_compose, preserve_order: true, run: "always"
  
  config.vm.provision "init", type: "shell", preserve_order: true, privileged: false, inline: <<-SHELL
    if ! grep -q github.com ~/.ssh/known_hosts; then
      ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts
    fi
  SHELL
    
  config.vm.provision "dotfiles", type: "shell", preserve_order: true, privileged: false, inline: <<-SHELL
    if [ ! -d ~/.dotfiles ]; then
      rm -rf .bash_profile .bashrc
      git clone https://github.com/tinslice/dotfiles.git ~/.dotfiles
      cd ~/.dotfiles
      ./install
    fi
  SHELL
    
  config.vm.provision "init-docker", type: "shell", preserve_order: true, privileged: false, run: "always",
      env: { "DOCKER_REGISTRY" => config.user.docker.registry, "DOCKER_USERNAME" => config.user.docker.user, "DOCKER_PASSWORD" => config.user.docker.password}, 
      inline: <<-SHELL
    if [ -n "$DOCKER_REGISTRY" ]; then
      echo "$DOCKER_PASSWORD" | docker login $DOCKER_REGISTRY -u "$DOCKER_USERNAME" --password-stdin
    fi
  SHELL

  config.vm.define "srv01" do |srv01|
    srv01.vm.box = "ubuntu/bionic64"
    srv01.vm.box_check_update = true
    srv01.ssh.forward_agent = true
    srv01.disksize.size = '16GB' 
    srv01.vm.network "private_network", ip: "192.168.90.10"
    srv01.vm.network "forwarded_port", host_ip: "127.0.0.1", host: 80, guest: 80 
    srv01.vm.network "forwarded_port", host_ip: "127.0.0.1", host: 443, guest: 443 
    srv01.vm.network "forwarded_port", host_ip: "127.0.0.1", host: 5432, guest: 5432 # postgres
    srv01.vm.network "forwarded_port", host_ip: "127.0.0.1", host: 8761, guest: 8761 # eureka
    srv01.vm.hostname = config.user.services.vm_name

    srv01.vm.provider :virtualbox do |vb|
      # Display the VirtualBox GUI when booting the machine
      vb.gui = true
      vb.name = config.user.services.vm_name + ".docker.vm"
      vb.memory = "4000"
      vb.cpus = 2
    end

    
    # srv01.vm.provision "copy-ssh", type: "file", preserve_order: true, source: "~/.ssh/id_rsa", destination: "~/.ssh/id_rsa"
  
    srv01.vm.synced_folder config.user.services.config_location, "/workspace", create: true, id: "synch-config-srv01"
  end
end
