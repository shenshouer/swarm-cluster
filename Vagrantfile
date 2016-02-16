require 'fileutils'
require 'yaml'

# Size of the cluster created by Vagrant
num_instances=4

# Read YAML file with mountpoint details
MOUNT_POINTS = YAML::load_file('synced_folders.yaml')

module OS
  def OS.windows?
    (/cygwin|mswin|mingw|bccwin|wince|emx/ =~ RUBY_PLATFORM) != nil
  end

  def OS.mac?
   (/darwin/ =~ RUBY_PLATFORM) != nil
  end

  def OS.unix?
    !OS.windows?
  end

  def OS.linux?
    OS.unix? and not OS.mac?
  end
end

# Change basename of the VM
instance_name_prefix="swarm"
consul_ip=""

Vagrant.configure("2") do |config|
  # always use Vagrants insecure key
  config.ssh.insert_key = false

  config.vm.box = "coreos-alpha"

  config.vm.provider :virtualbox do |v|
    # On VirtualBox, we don't have guest additions or a functional vboxsf
    # in CoreOS, so tell Vagrant that so it can be smarter.
    v.check_guest_additions = false
    v.memory = 2048
    v.cpus = 2
    v.functional_vboxsf     = false
  end

  # Set up each box
  (1..num_instances).each do |i|
    vm_name = "%s-%02d" % [instance_name_prefix, i]
    config.vm.define vm_name do |host|
      host.vm.hostname = vm_name
	host.vm.synced_folder ".", "/vagrant", disabled: true
	begin
	  MOUNT_POINTS.each do |mount|
	    mount_options = ""
	    disabled = false
	    nfs =  true
	    if mount['mount_options']
		mount_options = mount['mount_options']
	    end
	    if mount['disabled']
		disabled = mount['disabled']
	    end
	    if mount['nfs']
		nfs = mount['nfs']
	    end
	    if File.exist?(File.expand_path("#{mount['source']}"))
		if mount['destination']
		  host.vm.synced_folder "#{mount['source']}", "#{mount['destination']}",
		    id: "#{mount['name']}",
		    disabled: disabled,
		    mount_options: ["#{mount_options}"],
		    nfs: nfs
		end
	    end
	  end
	rescue
	end
      ip = "172.18.19.#{i+100}"
      host.vm.network :private_network, ip: ip

      host.vm.provision :shell, :inline => "/usr/bin/timedatectl set-timezone Asia/Shanghai ", :privileged => true
      sedInplaceArg = OS.mac? ? " ''" : ""
      if i == 1
        consul_ip = ip
        system "cp master-consul-config.yaml.tmpl master-consul-config.yaml"
        host.vm.provision :file, :source => "./master-consul-config.yaml", :destination => "/tmp/vagrantfile-user-data"
        host.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      elsif i == 2
        system "cp master-config.yaml.tmpl master-config.yaml"
        system "sed -e 's|__CONSUL_IP__|#{consul_ip}|g' -i#{sedInplaceArg} ./master-config.yaml"
        host.vm.provision :file, :source => "./master-config.yaml", :destination => "/tmp/vagrantfile-user-data"
        host.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      else
        system "cp node-config.yaml.tmpl node-config.yaml"
        system "sed -e 's|__CONSUL_IP__|#{consul_ip}|g' -i#{sedInplaceArg} ./node-config.yaml"
        host.vm.provision :file, :source => "./node-config.yaml", :destination => "/tmp/vagrantfile-user-data"
        host.vm.provision :shell, :inline => "mv /tmp/vagrantfile-user-data /var/lib/coreos-vagrant/", :privileged => true
      end

      #host.vm.provision :docker, images: ["swarm:latest"]
    end
  end
end
