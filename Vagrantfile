# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT

CONSUL_VERSION=0.7.0

sudo yum update -y && sudo yum install curl unzip wget bind-utils -y

wget -P /tmp https://releases.hashicorp.com/consul/${CONSUL_VERSION}/consul_${CONSUL_VERSION}_linux_amd64.zip
wget -P /tmp https://releases.hashicorp.com/consul/${CONSUL_VERSION}/consul_${CONSUL_VERSION}_web_ui.zip

sudo unzip /tmp/consul_${CONSUL_VERSION}_linux_amd64.zip -d /opt/consul_${CONSUL_VERSION}_linux_amd64
sudo unzip /tmp/consul_${CONSUL_VERSION}_web_ui.zip -d /opt/consul_${CONSUL_VERSION}_web_ui

sudo ln -sf /opt/consul_${CONSUL_VERSION}_linux_amd64/consul /usr/bin/consul

sudo mkdir -p /etc/consul.d/
sudo mkdir -p /var/log/consul/
sudo chmod a+w /etc/consul.d/ /var/log/consul/

SCRIPT

VAGRANTFILE_API_VERSION = '2'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.ssh.insert_key = false
  config.vm.provision 'shell', inline: $script

  box_name = 'opscode_centos-7.2_chef-provisionerless'
  config.vm.box = box_name
  config.vm.box_url = 'https://opscode-vm-bento.s3.amazonaws.com/' \
  "vagrant/virtualbox/#{box_name}.box"

  config.vm.define 'node_1' do |node_1|
    node_1.vm.network :private_network, ip: '192.168.33.35'
    node_1.vm.hostname = 'node1'

    node_1.vm.provider :virtualbox do |vb|
      vb.customize ['modifyvm', :id, '--memory', '512']
    end

    node_1.vm.provision(
      'shell',
      inline: 'consul agent -server -bootstrap-expect 3 -data-dir /tmp/consul' \
      ' -node=agent-one -bind=192.168.33.35 -config-dir /etc/consul.d' \
      ' > /var/log/consul/consul.log 2>&1 &'
    )
  end

  config.vm.define 'node_2' do |node_2|
    node_2.vm.hostname = 'node2'
    node_2.vm.network :private_network, ip: '192.168.33.36'

    node_2.vm.provider :virtualbox do |vb|
      vb.customize ['modifyvm', :id, '--memory', '512']
    end

    node_2.vm.provision(
      'shell',
      inline: 'consul agent -data-dir /tmp/consul' \
      ' -node=agent-two -bind=192.168.33.36 -config-dir' \
      ' /etc/consul.d -join 192.168.33.35 > /var/log/consul/consul.log 2>&1 &'
    )
  end

  config.vm.define 'node_3' do |node_3|
    node_3.vm.network :private_network, ip: '192.168.33.37'
    node_3.vm.hostname = 'node3'

    node_3.vm.provider :virtualbox do |vb|
      vb.customize ['modifyvm', :id, '--memory', '512']
    end

    node_3.vm.provision(
      'shell',
      inline: 'consul agent -data-dir /tmp/consul' \
      ' -node=agent-three -bind=192.168.33.37 -config-dir' \
      ' /etc/consul.d -join 192.168.33.36 > /var/log/consul/consul.log 2>&1 &'
    )
  end
end