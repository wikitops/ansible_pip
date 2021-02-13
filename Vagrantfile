# -*- mode: ruby -*-
# vi: set ft=ruby :

# List of supported operating systems
SUPPORTED_OS = {
  "debian9"    => {box: "generic/debian9",    user: "vagrant"},
  "debian10"   => {box: "generic/debian10",   user: "vagrant"},
  "ubuntu1904" => {box: "generic/ubuntu1904", user: "vagrant"},
  "ubuntu2004" => {box: "generic/ubuntu2004", user: "vagrant"},
  "centos7"    => {box: "generic/centos7",    user: "vagrant"}
}

# Vagrant instance management
$os                     = "ubuntu2004"
$num_instances          = 1
$instance_name_prefix   = "pip"
$vm_memory              = 1024
$vm_cpus                = 1
$subnet                 = "10.0.0.1" # For 10.0.0.1X
$box                    = SUPPORTED_OS[$os][:box]

# Ansible provisioner
$playbook               = "pip.yml"

# if $inventory is not set, try to use example
$inventory = File.join(File.dirname(__FILE__), "inventory") if ! $inventory

# if $inventory has a hosts file use it, otherwise copy over vars etc
# to where vagrant expects dynamic inventory to be.
if ! File.exist?(File.join(File.dirname($inventory), "hosts"))
  $vagrant_ansible = File.join(File.dirname(__FILE__), ".vagrant", "provisioners", "ansible")
  FileUtils.mkdir_p($vagrant_ansible) if ! File.exist?($vagrant_ansible)
  if ! File.exist?(File.join($vagrant_ansible,"inventory"))
    FileUtils.ln_s($inventory, File.join($vagrant_ansible,"inventory"))
  end
end

Vagrant.configure("2") do |config|

  # always use Vagrants insecure key
  config.ssh.insert_key = false
  config.ssh.username   = SUPPORTED_OS[$os][:user]

  # Configure box
  config.vm.box         = $box

  (1..$num_instances).each do |i|

    config.vm.provider "virtualbox" do |vb|
      vb.memory         = $vm_memory
      vb.cpus           = $vm_cpus
    end

    config.vm.define vm_name = "%s%02d" % [$instance_name_prefix, i] do |server|
      config.vm.hostname = vm_name
      server.vm.network "private_network", ip: "#{$subnet}#{i}"

      # Provision
      config.vm.provision "shell", path: "provision.sh"
      # config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "/home/vagrant/.ssh/authorized_keys"

      # Only execute the Ansible provisioner when all the machines are up and ready
      if i == $num_instances
        config.vm.provision "ansible" do |ansible|
          ansible.compatibility_mode  = "2.0"
          ansible.playbook            = $playbook
          if File.exist?(File.join(File.dirname($inventory), "hosts"))
            ansible.inventory_path    = $inventory
          end
          ansible.become              = true
          ansible.limit               = "all"
          ansible.host_key_checking   = false
          ansible.groups = {
            "pip" => ["#{$instance_name_prefix}0[1:#{$num_instances}]"]
          }
        end
      end
    end
  end
end
