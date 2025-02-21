# Vagrantfile
Vagrant.configure("2") do |config|
  # Box image (e.g., Ubuntu 20.04 or Alpine)
  config.vm.box = "ubuntu/focal64"

  config.vm.define "master" do |master|
    master.vm.hostname = "k3s-master"
    master.vm.network "private_network", ip: "192.168.56.10"
    master.vm.provider "virtualbox" do |vb|
      vb.name = "k3s-master"
      vb.memory = 1024
      vb.cpus = 1
    end
    master.vm.provision "shell", inline: <<-SHELL
      curl -sfL https://get.k3s.io | sh -s - server
      # Save K3s token for agent
      cat /var/lib/rancher/k3s/server/node-token > /vagrant/node-token
    SHELL
  end

  config.vm.define "agent" do |agent|
    agent.vm.hostname = "k3s-agent"
    agent.vm.network "private_network", ip: "192.168.56.11"
    agent.vm.provider "virtualbox" do |vb|
      vb.name = "k3s-agent"
      vb.memory = 1024
      vb.cpus = 1
    end
    agent.vm.provision "shell", inline: <<-SHELL
      # Wait for node-token to exist (master might not be fully up)
      while [ ! -f /vagrant/node-token ]; do
        sleep 2
      done
      TOKEN=$(cat /vagrant/node-token)
      MASTER_IP="192.168.56.10"
      curl -sfL https://get.k3s.io | K3S_URL="https://$MASTER_IP:6443" K3S_TOKEN="$TOKEN" sh -s - agent
    SHELL
  end
end
