# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
$dockercompose_script = <<SCRIPT
# Get Docker Engine - Community for CentOS
# https://docs.docker.com/install/linux/docker-ce/centos/
sudo yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-engine
sudo yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# sudo yum install -y docker-ce \
# docker-ce-cli \
# containerd.io
sudo yum install -y docker-ce-19.03.2 \
docker-ce-cli-19.03.2 \
containerd.io
sudo systemctl enable docker
sudo systemctl start docker && sudo docker --version
# sudo groupadd docker
sudo usermod -aG docker vagrant # add user to the docker group
# Install Docker Compose
# https://docs.docker.com/compose/install/
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose && sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
sudo docker-compose --version
docker --version
hostnamectl status
SCRIPT
auto = ENV['AUTO_START_SWARM'] || true # create swarm auto mode
multi_manager =  true # create swarm multi manager mode
numworkers = 2
vmmemory = 512
# Increase numcpu if you want more cpu's per vm
numcpu = 1
instances = []
(1..numworkers).each do |n|
  instances.push({:name => "worker#{n}", :ip => "192.168.10.2#{n+2}"})
end

manager_ip = "192.168.10.2"

File.open("./hosts", 'w') { |file|
  instances.each do |i|
    file.write("#{i[:ip]} #{i[:name]}.local #{i[:name]}\n")
  end
  file.write("#{manager_ip} manager.local manager\n")
}


# Vagrant version requirement
Vagrant.require_version ">= 1.8.4"

Vagrant.configure("2") do |config|

    config.vm.provider "virtualbox" do |v|
     	v.memory = vmmemory
  	  v.cpus = numcpu
      v.customize ["modifyvm", :id, "--groups", "/swarm-centos-sandbox"] # create vbox group
    end

    config.vm.define "manager" do |i|
      i.vm.box = "bento/centos-7.7"
      i.vm.hostname = "manager"
      i.vm.network "private_network", ip: "#{manager_ip}"
      i.vm.provision "shell", inline: $dockercompose_script, privileged: false
      if File.file?("./hosts")
        i.vm.provision "file", source: "hosts", destination: "/tmp/hosts"
        i.vm.provision "shell", inline: "cat /tmp/hosts >> /etc/hosts", privileged: true
      end
      if auto
        i.vm.provision "shell", inline: "docker swarm init --advertise-addr #{manager_ip}"
        i.vm.provision "shell", inline: "docker swarm join-token -q worker > /vagrant/token"
        i.vm.provision "shell", inline: "docker swarm join-token -q manager > /vagrant/manager_token"
      end
    end

     if multi_manager == true
        nummanagers = 1 # of extra managers
        instances_managers = []

        (1..nummanagers).each do |m|
          instances_managers.push({:name => "manager#{m}", :ip => "192.168.10.#{m+2}"})
        end # (1..nummanagers)
        # File.open("./hosts", 'w') { |file|
        File.open('./hosts', 'a'){ |file|
          instances_managers.each do |i|
            file.write("#{i[:ip]} #{i[:name]}.local #{i[:name]}\n")
          end
        } # File.open("./hosts", 'w')

        instances_managers.each do |instancemgr|
                config.vm.define instancemgr[:name] do |i|
                  i.vm.box = "bento/centos-7.7"
                  i.vm.hostname = instancemgr[:name]
                  i.vm.network "private_network", ip: "#{instancemgr[:ip]}"
                  i.vm.provision "shell", inline: $dockercompose_script, privileged: false

                  if File.file?("./hosts")
                    i.vm.provision "file", source: "hosts", destination: "/tmp/hosts"
                    i.vm.provision "shell", inline: "cat /tmp/hosts >> /etc/hosts", privileged: true
                  end # if File.file?("./hosts")

                  if auto
                      i.vm.provision "shell", inline: "docker swarm join --token `cat /vagrant/manager_token` #{manager_ip}:2377"
                  end # if auto

                end
        end # instances_managers.eac
     end # if multi_manager


      instances.each do |instance|
      config.vm.define instance[:name] do |i|
        i.vm.box = "bento/centos-7.7"
        i.vm.hostname = instance[:name]
        i.vm.network "private_network", ip: "#{instance[:ip]}"
        i.vm.provision "shell", inline: $dockercompose_script, privileged: false
          if File.file?("./hosts")
            i.vm.provision "file", source: "hosts", destination: "/tmp/hosts"
            i.vm.provision "shell", inline: "cat /tmp/hosts >> /etc/hosts", privileged: true
          end
          if auto
            i.vm.provision "shell", inline: "docker swarm join --advertise-addr #{instance[:ip]} --listen-addr #{instance[:ip]}:2377 --token `cat /vagrant/token` #{manager_ip}:2377"
          end
      end
     end

end
