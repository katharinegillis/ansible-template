# Development environment Vagrantfile.

Vagrant.require_version ">= 1.8.0"
VAGRANTFILE_API_VERSION = "2"

# Get the environment configuration from Ansible.
require "yaml"
env = YAML.load_file(File.join(File.dirname(__FILE__), "ansible/host_vars/localhost.yml"))

# Check for required plugins.
if env.has_key?("domain_aliases") && !Vagrant.has_plugin?("vagrant-hostsupdater")
	puts "The vagrant-hostsupdater plugin is required. Please install it with \"vagrant plugin install vagrant-hostsupdater\"."
	exit
end

if Vagrant::Util::Platform.windows? && !Vagrant.has_plugin?("vagrant-winnfsd")
	puts "The vagrant-winnfsd plugin is required. Please install it with \"vagrant plugin install vagrant-winnfsd\"."
	exit
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
	# Set up the Ubuntu 16.04 box.
	config.vm.box = "bento/ubuntu-16.04"
	config.vm.provider "virtualbox" do |v|
		v.linked_clone = true
	end

	# Set up the network.
	config.vm.network "private_network", ip: env["ip"]
	config.vm.hostname = env["domain"]
	if env.has_key?("domain_aliases")
		config.hostsupdater.aliases = env["domain_aliases"]
	end

	# Set up the shared folders.
	shared_folders = env["shared_folders"]
	shared_folders.each do |shared_folder|
		config.vm.synced_folder shared_folder["src"], shared_folder["dest"], shared_folder["options"]
	end

	# Provision the machine.
	config.vm.provision "ansible" do |ansible|
		ansible.playbook       = "ansible/playbook.yml"
		ansible.inventory_path = "ansible/inventories/localhost"
		ansible.limit          = "all"
	end
end