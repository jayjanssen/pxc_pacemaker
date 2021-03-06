# -*- mode: ruby -*-
# vi: s# -*- mode: ruby -*-
# vi: set ft=ruby :

# Assumes a box from https://github.com/jayjanssen/packer-percona

# This sets up 3 nodes with a common PXC, but you need to run bootstrap.sh to connect them.

require './lib/vagrant-common.rb'

# Our puppet config
puppet_config = {
	'percona_server_version' => '56',
	'innodb_buffer_pool_size' => '128M',
	'innodb_log_file_size' => '64M',
	'innodb_flush_log_at_trx_commit' => '0',
}

# Node names and ips (for local VMs)
pxc_nodes = {
	'node1' => '192.168.70.2',
	'node2' => '192.168.70.3',
	'node3' => '192.168.70.4',
}

Vagrant.configure("2") do |config|
	config.vm.box = "centos-6_5-64_percona"
	config.ssh.username = "root"

	# Create all three nodes identically except for name and ip
	pxc_nodes.each_pair { |node, ip|
		config.vm.define node do |node_config|
			node_config.vm.hostname = node
			node_config.vm.network :private_network, ip: ip

			provider_aws( node_config, "PXC #{node}", 'm1.small') { |aws, override|
				aws.block_device_mapping = [
					{
						'DeviceName' => "/dev/sdb",
						'VirtualName' => "ephemeral0"
					}
				]
				provision_puppet( override, 'pxc.pp', 
					puppet_config.merge( 'datadir_dev' => 'xvdb' )
				)
			}

			provider_virtualbox( node_config, '256' ) { |vb, override|		
				provision_puppet( override, 'pxc_pacemaker.pp', 
					puppet_config.merge('datadir_dev' => 'dm-2')
				)

			}

		end
	}
end
