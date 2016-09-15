
Vagrant.configure("2") do |config|
  
  config.vm.box = "devops/ubuntu-14-04-x64"
  config.vm.hostname = "docker-java"
  config.vm.network "private_network", ip: "192.168.123.45"

  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--memory", "4096"]
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
  end

end


