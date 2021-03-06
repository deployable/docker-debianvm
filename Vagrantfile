# -*- mode: ruby -*-
# vi: set ft=ruby :

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
  config.vm.box = "debian/stretch64"
  config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network :forwarded_port, guest: 22, host: 2232

  # Create a private network, which allows host-only access, fix the ip
  config.vm.network "private_network", ip: "192.168.98.20", auto_config: false
  #config.vm.provision "shell", run: "always", inline: "set -uex; ip ad sh dev br0 || ; ip ad sh dev br0 | grep -q 192\.168\.98\.20 || (ip ad add 192.168.98.20/24 dev br0 && ip link set dev br0 up)"


  # Create a public network, bridged network.
  #config.vm.network "public_network"

  # Share an additional folder to the guest VM
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #   # Customize the amount of memory on the VM:
    vb.memory = "1200"
    vb.customize ['modifyvm', :id, '--nictype1', 'Am79C970A']
    vb.customize ['modifyvm', :id, '--nictype2', 'Am79C970A']
    vb.customize ['modifyvm', :id, '--nicpromisc2', 'allow-all']
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    set -uex
    apt-get update -y
    apt-get upgrade -y
 
    # vbox guest additions
    vbox_version="5.1.16"
    guest_install=1
    if lsmod | grep -qi vboxguest; then 
      guest_version=$(modinfo vboxguest | awk '/^version:/ { print $2 }' 2>/dev/null)
      if [ "$guest_version" == "$vbox_version" ]; then
        guest_install=0
      fi
    fi; 
    if [ "$guest_install" == "1" ]; then
      apt-get install -y build-essential module-assistant linux-headers-amd64
      wget --progress=dot:giga -O VBoxGuestAdditions_${vbox_version}.iso -c http://download.virtualbox.org/virtualbox/${vbox_version}/VBoxGuestAdditions_${vbox_version}.iso
      mount -o loop,ro VBoxGuestAdditions_${vbox_version}.iso /mnt
      sh /mnt/VBoxLinuxAdditions.run
      umount /mnt
    fi
    
    # Docker
    apt-get install apt-transport-https ca-certificates dirmngr -y
    apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
    grep -q 'deb https://apt.dockerproject.org/repo debian-stretch main' /etc/apt/sources.list.d/docker.list || echo 'deb https://apt.dockerproject.org/repo debian-stretch main' > /etc/apt/sources.list.d/docker.list
    apt-get update
    apt-get install docker-engine -y
    # Listen on Tcp
    [ -d /etc/systemd/system/docker.service.d ] || mkdir /etc/systemd/system/docker.service.d
    if [ ! -f /etc/systemd/system/docker.service.d/override.conf ]; then 
      echo '[Service]
Environment="DOCKERD_EXTRA_OPTS=-H tcp://0.0.0.0:2375"' > /etc/systemd/system/docker.service.d/override.conf
    fi
    if ! grep DOCKERD_EXTRA_OPTS /lib/systemd/system/docker.service; then
      sed -i '/ExecStart/s/$/ $DOCKERD_EXTRA_OPTS/' /lib/systemd/system/docker.service
    fi
    systemctl restart docker.service
    docker run alpine echo "hello world!" 

    # Pipework
    wget -O /usr/local/bin/pipework https://raw.githubusercontent.com/jpetazzo/pipework/ae42f1b5fef82b3bc23fe93c95c345e7af65fef3/pipework
    chmod 755 /usr/local/bin/pipework 

    # Setup bridge/eth2 interface
    apt-get install bridge-utils
    grep -q 'auto eth1' || echo 'auto eth1' >> /etc/network/interfaces
    grep -q 'iface eth1 inet manual' || echo 'iface eth1 inet manual' >> /etc/network/interfaces
    grep -q 'auto brocker0' || echo 'auto brocker0' >> /etc/network/interfaces
    echo 'iface brocker0 inet static
    bridge_ports eth1
        address 192.168.98.30
        broadcast 192.168.98.255
        netmask 255.255.255.0' >> /etc/network/interfaces
    ifdown eth1 && ifup eth1
    if ! ip link sh brocker0; then ifup brocker0; fi

    docker network create --driver=bridge \
       --subnet 192.168.98.0/24 \
       --gateway=192.168.98.30 \
       --ip-range=192.168.98.128/25 \
       vmhost \
       -o "com.docker.network.bridge.name"="brocker0"    
  SHELL
end
