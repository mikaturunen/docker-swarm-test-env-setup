# Defines Vagrant environment for testing

Vagrant.configure("2") do |config|

	# Spin up a master machine for installing master to it
	config.vm.define :server do |mgmt_config|
		mgmt_config.vm.box = "ubuntu/trusty64"
		mgmt_config.vm.hostname = "manager"
		mgmt_config.vm.network :private_network, ip: "10.0.15.30"
		mgmt_config.vm.provider "virtualbox" do |vb|
			vb.name   = "swarm-manager"
			vb.memory = "256"
		end
		mgmt_config.vm.provision :shell, path: "bootstrap.sh"
	end

	# Spin up some client servers so we can test the client deployments
	# https://docs.vagrantup.com/v2/vagrantfile/tips.html
	(1..2).each do |i|
		config.vm.define "client#{i}" do |node|
			node.vm.box = "ubuntu/trusty64"
			node.vm.hostname = "swarm-client#{i}"
			node.vm.network :private_network, ip: "10.0.15.4#{i}"
			node.vm.network "forwarded_port", guest: 80, host: "809#{i}"
			node.vm.provider "virtualbox" do |vb|
				vb.name   = "swarm-client-#{i}"
				vb.memory = "256"
			end
			node.vm.provision :shell, path: "bootstrap.sh"
		end
	end

end
