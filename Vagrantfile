# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT

echo "Installing dependencies ..."
sudo apt-get update
sudo apt-get install -y curl jq

echo "Installing Vault Enterprise ..."
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main" 
sudo apt-get install -y vault-enterprise 
sudo systemctl enable vault.service

echo "Setting VAULT_ADDR environment variable ..."
echo export VAULT_ADDR="http://$(hostname -i):8200" >> /home/vagrant/.profile

echo "Adding license ..."
echo "MY_LICENSE_DATA_HERE" > /etc/vault.d/license.hclic
echo "VAULT_LICENSE_PATH=/etc/vault.d/license.hclic" >> /etc/vault.d/vault.env

echo "Setting VAULT_RAFT_NODE_ID environment variable ..."
echo VAULT_RAFT_NODE_ID="$(hostname)" >> /etc/vault.d/vault.env

echo "Setting new Vault configuration ..."
sudo mv /etc/vault.d/vault.hcl /etc/vault.d/vault_hcl.old

tee /etc/vault.d/vault.hcl << EOF

ui = true
mlock = false
cluster_addr  = "http://$(hostname -I | awk '{ print $2 }'):8201"
api_addr      = "http://$(hostname -I | awk '{ print $2 }'):8200"

storage "raft" {
	path = "/opt/vault/data"

	retry_join {
	  leader_api_addr = "http://172.20.20.10"
	}
	retry_join {
	  leader_api_addr = "http://172.20.20.11"
	}
	retry_join {
	  leader_api_addr = "http://172.20.20.12"
	}
}

listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = 1
}

EOF

sudo chown -R vault:vault /etc/vault.d/*

sudo systemctl start vault.service
sleep 5
if [ "`hostname`" = "one" ]
	then
		VAULT_ADDR="http://$(hostname -I | awk '{ print $2 }'):8200" vault operator init -key-shares=1 -key-threshold=1 > /home/vagrant/unseal.keys
		VAULT_ADDR="http://$(hostname -I | awk '{ print $2 }'):8200" vault operator unseal $(cat /home/vagrant/unseal.keys | grep Unseal | awk '{ print $4 }')
	elif [ "`hostname`" = "prone" ]
	then
		VAULT_ADDR="http://$(hostname -I | awk '{ print $2 }'):8200" vault operator init -key-shares=1 -key-threshold=1 > /home/vagrant/unseal.keys
		VAULT_ADDR="http://$(hostname -I | awk '{ print $2 }'):8200" vault operator unseal $(cat /home/vagrant/unseal.keys | grep Unseal | awk '{ print $4 }')
	elif [ "`hostname`" = "drone" ]
	then
		VAULT_ADDR="http://$(hostname -I | awk '{ print $2 }'):8200" vault operator init -key-shares=1 -key-threshold=1 > /home/vagrant/unseal.keys
		VAULT_ADDR="http://$(hostname -I | awk '{ print $2 }'):8200" vault operator unseal $(cat /home/vagrant/unseal.keys | grep Unseal | awk '{ print $4 }')
	else
		VAULT_ADDR="http://127.0.0.1:8200" vault operator raft join "http://172.20.20.10:8200"
fi

SCRIPT

# Specify a Vault box for the LAB
#LAB_BOX_NAME = ENV['LAB_BOX_NAME'] || "bytesguy/ubuntu-server-21.10-arm64"

#Vagrant.configure("2") do |config|
#  config.vm.box = LAB_BOX_NAME

#  config.vm.provision "shell", inline: $script
  
#    config.vm.define "one" do |one|
#	one.vm.hostname = "one"
#        one.vm.network "private_network", ip: "172.20.20.10"
#    end

Vagrant.configure("2") do |config|
  config.vm.box = "multipass"
  config.vm.hostname = "multipass"
  config.vm.synced_folder ".", "/vagrant", type: "rsync"

  # Can be omited
#  config.ssh.username = "grunt"
#  config.ssh.private_key_path = ["#{ENV['HOME']}/.ssh/id_rsa"]

  config.vm.provider "multipass" do |multipass, override|
    multipass.hd_size = "10G"
    multipass.cpu_count = 1
    multipass.memory_mb = 2048
    multipass.image_name = "bionic"
  end
end


#end
