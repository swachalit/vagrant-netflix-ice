# -*- mode: ruby -*-
# vi: set ft=ruby :

def local_ip
  `ipconfig getifaddr en0`.strip
end

VAGRANTFILE_API_VERSION = "2"

# Ensure role dependencies are in place
if [ "up", "provision" ].include?(ARGV.first) && File.directory?("roles") &&
  !(File.directory?("roles/hectcastro.ice") || File.symlink?("roles/hectcastro.ice"))
  system("ansible-galaxy install -r roles.txt -p roles")
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = "netflix-ice"
  config.vm.network "forwarded_port", guest: 8080, host: 8080

  config.vm.provider("virtualbox") { |v| v.memory = 2048 }

  config.vm.synced_folder "data/ice_processor/", "/mnt/ice_processor",
                          mount_options: ["dmode=777","fmode=666"]
  config.vm.synced_folder "data/ice_reader/", "/mnt/ice_reader",
                          mount_options: ["dmode=777","fmode=666"]

  # Wire up the proxy
  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http     = "http://#{local_ip}:8123/"
    config.proxy.https    = "http://#{local_ip}:8123/"
    config.proxy.no_proxy = "localhost,127.0.0.1"
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "site.yml"
    ansible.groups = {
      "development" => [ "default" ]
    }
  end
end
