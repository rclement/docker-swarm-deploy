# -*- mode: ruby -*-
# vi: set ft=ruby :

BOX_NAME = ENV["BOX_NAME"] || "bento/ubuntu-16.04"
BOX_CPUS = ENV["BOX_CPUS"] || "1"
BOX_MEMORY = ENV["BOX_MEMORY"] || "1024"

LEADER_HOSTNAME = ENV["LEADER_HOSTNAME"] || "leader"
LEADER_IP = ENV["LEADER_IP"] || "10.0.0.10"
LEADER_FORWARDED_PORT = (ENV["LEADER_FORWARDED_PORT"] || "9000").to_i

MANAGER_HOSTNAME = ENV["MANAGER_HOSTNAME"] || "manager.local"
MANAGER_IP = ENV["MANAGER_IP"] || "10.0.0.11"
MANAGER_FORWARDED_PORT = (ENV["MANAGER_FORWARDED_PORT"] || "9001").to_i

WORKER_HOSTNAME = ENV["WORKER_HOSTNAME"] || "worker.local"
WORKER_IP = ENV["WORKER_IP"] || "10.0.0.12"
WORKER_FORWARDED_PORT = (ENV["WORKER_FORWARDED_PORT"] || "9002").to_i

PUBLIC_KEY_PATH = "#{Dir.home}/.ssh/id_rsa.pub"

Vagrant.configure("2") do |config|
  config.ssh.forward_agent = true

  config.vm.provider :virtualbox do |vb|
    # Ubuntu's Raring 64-bit cloud image is set to a 32-bit Ubuntu OS type by
    # default in Virtualbox and thus will not boot. Manually override that.
    vb.customize ["modifyvm", :id, "--ostype", "Ubuntu_64"]
    vb.customize ["modifyvm", :id, "--cpus", BOX_CPUS]
    vb.customize ["modifyvm", :id, "--memory", BOX_MEMORY]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "off"]
    # enable NAT adapter cable https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=838999
    vb.customize ["modifyvm", :id, "--cableconnected1", "on"]
  end

  config.vm.define "leader" do |leader|
    leader.vm.box = BOX_NAME
    leader.vm.hostname = LEADER_HOSTNAME
    leader.vm.network :private_network, ip: LEADER_IP
    leader.vm.network :forwarded_port, guest: 80, host: LEADER_FORWARDED_PORT
  end

  config.vm.define "manager" do |manager|
    manager.vm.box = BOX_NAME
    manager.vm.hostname = MANAGER_HOSTNAME
    manager.vm.network :private_network, ip: MANAGER_IP
    manager.vm.network :forwarded_port, guest: 80, host: MANAGER_FORWARDED_PORT
  end

  config.vm.define "worker" do |worker|
    worker.vm.box = BOX_NAME
    worker.vm.hostname = WORKER_HOSTNAME
    worker.vm.network :private_network, ip: WORKER_IP
    worker.vm.network :forwarded_port, guest: 80, host: WORKER_FORWARDED_PORT
  end

  if Pathname.new(PUBLIC_KEY_PATH).exist?
    config.vm.provision :file, source: PUBLIC_KEY_PATH, destination: '/tmp/id_rsa.pub'
    config.vm.provision :shell, :inline => "rm -f /root/.ssh/authorized_keys && mkdir -p /root/.ssh && sudo cp /tmp/id_rsa.pub /root/.ssh/authorized_keys"
  end
end
