# -*- mode: ruby -*-
# vi: set ft=ruby :

# This is a sample Vagrantfile. Copy it into your project root.

# REQUIREMENTS:
# @see https://github.com/cogitatio/vagrant-hostsupdater
# @see https://github.com/dotless-de/vagrant-vbguest

# Load some project specific settings form YAML files.
require 'yaml'
project_settings = YAML.load_file('Vagrantsettings.yaml')
# project_settings['vm_hostname'] = project_settings['vm_shortname'] + '_' + project_settings['vm_environment']
# abort(project_settings.inspect) # print variables and exit

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = project_settings['box']['name']

  # Information about the box.
  config.vm.box_url = project_settings['box']['url']
  config.vm.box_download_checksum = project_settings['box']['download_checksum']
  config.vm.box_download_checksum_type = project_settings['box']['download_checksum_type']

  # Setup the VM hostname.
  config.vm.hostname = project_settings['vm_shortname']

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  ########################################
  # Setup VM access:

  # 1. Add port forwarding
  # config.vm.network :forwarded_port, guest: 80, host: 8080

  # 2. Add a new virtual network inteface card with a fixed IP.
  config.vm.network :private_network, ip: project_settings['vm_hostonlyip']

  # Settings for the vagrant-hostsupdater plugin.
  # @see https://github.com/cogitatio/vagrant-hostsupdater
  config.hostsupdater.aliases = []
  for vhost in project_settings['vhosts']
    config.hostsupdater.aliases = config.hostsupdater.aliases + [vhost['domain']]
    config.hostsupdater.aliases = config.hostsupdater.aliases + vhost['aliases']
  end
  # Cleanup /etc/hosts when shuting down the VM.
  config.hostsupdater.remove_on_suspend = true

  ########################################

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Mount all the project document root folders into the gues OS.
  for vhost in project_settings['vhosts']
    if (vhost['source'])
      config.vm.synced_folder vhost['source'], vhost['docroot']
    end
  end

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  config.vm.provider :virtualbox do |v|
    # Use the host's resolver mechanisms to handle DNS requests.
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]

    # Set the amount of RAM, in MB, allocated to the virtual machine.
    v.customize ["modifyvm", :id, "--memory", 4096]

    # Set the number of virutal CPUs allocated to the virtual machine.
    v.customize ["modifyvm", :id, "--cpus", 2]
  end

  # Settings for the vagrant-vbguest plugin.
  # @see https://github.com/dotless-de/vagrant-vbguest
  config.vbguest.auto_update = true
  # config.vbguest.iso_path = "#{ENV['HOME']}/Downloads/VBoxGuestAdditions.iso"
  # config.vbguest.no_remote = false

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL

  # Settings for the vagrant-vbguest plugin.
  # @see https://github.com/dotless-de/vagrant-vbguest
  config.vbguest.auto_update = true
  # config.vbguest.iso_path = "#{ENV['HOME']}/Downloads/VBoxGuestAdditions.iso"
  # config.vbguest.no_remote = false

  if (project_settings['puppet_force_reinstall'] == 1)
    # Remove installed puppet modules.
    if (project_settings['puppet_development'] == 0)
      config.vm.provision "shell",
        path: 'https://raw.githubusercontent.com/devgateway/happy-deployer/v' + project_settings['puppet_module_version'] + '/shell/remove-modules.sh'
    elsif (project_settings['puppet_development'] == 1)
      config.vm.provision "shell",
        path: project_settings['happy_deployer_path'] + '/shell/remove-modules.sh'
    end
  end

  # Prepare the box using a shell script.
  if (project_settings['puppet_development'] == 0)
    config.vm.provision "shell",
      path: 'https://raw.githubusercontent.com/devgateway/happy-deployer/v' + project_settings['puppet_module_version'] + '/shell/setup-hiera.sh'
  elsif (project_settings['puppet_development'] == 1)
    config.vm.provision "shell",
      path: project_settings['happy_deployer_path'] + '/shell/setup-hiera.sh'
  end

  # Install the happydev puppet module and dependencies if not already installed.
  if (project_settings['puppet_development'] == 0)
    test_commands = "test -d /etc/puppet/modules/happydev"
    action_commands = "puppet module install " + project_settings['puppet_module_name'] + " --version " + project_settings['puppet_module_version']
    config.vm.provision "shell",
      inline: "(#{test_commands}) || (#{action_commands})"

    # Fix issue for example42-php module in redhat based systems. Ignore if patch has already been applied.
    test_commands = "test -x /usr/bin/patch"
    install_command = "yum install --tolerant --assumeyes patch"
    patch_command = "patch --forward --reject-file=- --input=/etc/puppet/modules/happydev/20140513--example42-php--redhat-ini.patch /etc/puppet/modules/php/manifests/ini.pp"
    config.vm.provision "shell",
      inline: "(#{test_commands}) || ((#{install_command}) && (#{patch_command} || true))"
  end

  if (project_settings['puppet_development'] == 0 && project_settings['puppet_manifest_file_contents'])
    # Create a custom puppet manifest file.
    config.vm.provision "shell",
      inline: 'mkdir -p /etc/puppet/manifests && echo "' + project_settings['puppet_manifest_file_contents'] + '" > /etc/puppet/manifests/site.pp'
  end

  # Install ImageMagick.
  test_commands = "test -x /usr/bin/convert"
  action_commands = "yum install --tolerant --assumeyes ImageMagick"
  config.vm.provision "shell",
    inline: "(#{test_commands}) || (#{action_commands})"

  config.vm.provision 'puppet' do |puppet|
    if (project_settings['puppet_development'] == 0)
      # 1. Default Puppet settings:
      puppet.options = project_settings['puppet_options']

      puppet.manifests_path = ['vm', '/etc/puppet/manifests']
      puppet.manifest_file = 'site.pp'
    elsif (project_settings['puppet_development'] == 1)
      # 2. Development Puppet settings:
      puppet.options = project_settings['puppet_options_dev']

      puppet.manifests_path = project_settings['happy_deployer_path'] + '/puppet/manifests'
      puppet.manifest_file = project_settings['puppet_alternative_manifest_file']

      puppet.module_path = project_settings['happy_deployer_path'] + '/puppet/modules'
    end

    # Variables passed to puppet files.
    puppet.facter = {
      'fqdn' => project_settings['vhosts'].first['domain'],
      'environment' => project_settings['vm_environment'],
    }
  end
end
