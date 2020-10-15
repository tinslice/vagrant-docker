# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV["LC_ALL"] = "en_US.UTF-8"

Vagrant.require_version ">= 2.2.10"

Vagrant.configure("2") do |config|
  # add "vagrant-hostsupdater" to update hosts file on `vagrant up`
  config.vagrant.plugins = [ "nugrant", "vagrant-vbguest", "vagrant-disksize", "vagrant-docker-compose", "vagrant-reload"]

  config.user.defaults = {
    'vm_box'        => 'ubuntu/bionic64',
    'vm_name'       => 'test.vm',
    'os_disk_size'  => '16GB',
    'show_gui'   => false,
    'ssh' => {
      'public_key' => ""
    },
    'nodes' => [
      {
        'name'      => 'node-01',
        'memory'    => 1024,
        'cpu'       => 1,
        'ip'        => '192.168.90.10'
      }
    ],
    'docker' => {
      'install' => false,
      'registry' => "",
      'user' => "",
      'password' => ""
    }
  }

  config.vm.provision "default packages", type: "shell", preserve_order: true,inline: <<-SHELL
    apt-get update && apt-get -y upgrade
    sudo apt-get -y install apt-transport-https lsb-release ca-certificates gnupg2 dirmngr software-properties-common curl jq nmap rsync joe vim mc tmux git build-essential vnstat htop xclip xsel
  SHELL

  if config.user.docker.install then
    config.vm.provision :docker, preserve_order: true, run: "always"
    config.vm.provision :docker_compose, preserve_order: true, run: "always"
  end

  config.vm.provision "init", type: "shell", preserve_order: true, privileged: false, inline: <<-SHELL
    if ! grep -q github.com ~/.ssh/known_hosts; then
      ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts
    fi
  SHELL

  config.vm.provision "dotfiles", type: "shell", preserve_order: true, privileged: false, run: "always", inline: <<-SHELL
    if [ ! -d ~/.dotfiles ]; then
      rm -rf .bash_profile .bashrc
      git clone https://github.com/tinslice/dotfiles.git ~/.dotfiles
      cd ~/.dotfiles
      ./install
    else
      cd ~/.dotfiles
      git pull
    fi
  SHELL

  if config.user.docker.user != "" then
    config.vm.provision "init-docker", type: "shell", preserve_order: true, privileged: false, run: "always",
        env: { "DOCKER_REGISTRY" => config.user.docker.registry, "DOCKER_USERNAME" => config.user.docker.user, "DOCKER_PASSWORD" => config.user.docker.password},
        inline: <<-SHELL
      if [ -n "$DOCKER_REGISTRY" ]; then
        echo "$DOCKER_PASSWORD" | docker login $DOCKER_REGISTRY -u "$DOCKER_USERNAME" --password-stdin
      fi
    SHELL
  end

  if config.user.ssh.public_key != "" then
    config.vm.provision "copy-ssh-public-key", type: "file", preserve_order: true, source: config.user.ssh.public_key , destination: "~/.ssh/id_rsa_host.pub"
    config.vm.provision "shell", privileged: false, run: "always", inline: <<-SHELL
      KEY=$(cat ~/.ssh/id_rsa_host.pub)
      if [ -z "$(grep "$KEY" ~/.ssh/authorized_keys )" ]; then
        echo $KEY >> ~/.ssh/authorized_keys
      fi
    SHELL
  end

  (0.. (config.user.nodes.size-1)).each do |i|
    config.vm.define config.user.nodes[i].name do |node|
      node.vm.box = config.user.vm_box
      node.vm.box_check_update = true
      node.ssh.forward_agent = true
      node.disksize.size = config.user.os_disk_size

      node.vm.hostname = config.user.nodes[i].name
      node.vm.network "private_network", ip: config.user.nodes[i].ip

      node.vm.provider :virtualbox do |vb|
        # Display the VirtualBox GUI when booting the machine
        vb.gui = config.user.show_gui
        vb.name = config.user.nodes[i].name + "." + config.user.vm_name
        vb.memory = config.user.nodes[i].memory
        vb.cpus = config.user.nodes[i].cpu
      end
    end
  end

end
