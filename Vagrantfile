# -*- mode: ruby -*-
# vi: set ft=ruby :
# Imports :
# Require YAML module
require 'yaml'

# variables :
VAGRANT_DIR = File.expand_path(File.dirname(__FILE__))
VAGRANTFILE_API_VERSION = "2"
Vagrant.require_version ">=1.8.4"

# Read YAML file with box details
servers = YAML.load_file('servers.yml')

# Create boxes
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Iterate through entries in YAML file
  servers.each do |servers|

  	if servers["username"]
			config.ssh.username = servers["username"]
		end

  	if servers["password"]
			config.ssh.password = servers["password"]
			config.ssh.insert_key = false
		end

    config.vm.define servers["name"] do |srv|

    	if servers["box_url"] != nil
    		srv.vm.box_url = servers["box_url"]
    	else
    	  if servers["box_version"] and
      	  srv.vm.box_version = servers["box_version"]
      	  srv.vm.box_check_update = false
        end
    	end
    		srv.vm.box = servers["box"]

      if servers["ip"]
      	srv.vm.network "private_network", ip: servers["ip"]
			end

      if servers["provider"] != nil
      	case servers["provider"]
      	when "virtualbox"
					srv.vm.provider :virtualbox do |vb|
						vb.name = servers["name"]
						vb.memory = servers["ram"]
						vb.gui = servers["gui"]
					end
				else
					puts "Supported provide ATM is Virtualbox ... exiting"
				end
      end
      srv.vm.provision :ansible do |ansible|
            ansible.limit = "all"
            ansible.playbook = "./playbooks/default.yml"
            ansible.inventory_path = "./inventories/vagrant_hosts"
            ansible.sudo = "true"
            ansible.sudo_user = "root"
            ansible.host_key_checking = "false"
            #ansible.verbose = "#{ansible_verbosity}"
            ansible.extra_vars = {
              some_var: "some_value",
              dict: {
                kay1: "val1",
                kay2: "val2"
              }
            }
      end
    end
  end
end
