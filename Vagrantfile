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
  config.vm.box = "debian/jessie64"
  config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access, fix the ip
  config.vm.network "private_network", ip: "192.168.98.20"
  config.vm.network "private_network", ip: "192.168.98.30", auto_config: false
  #config.vm.provision "shell", run: "always", inline: "set -uex; ip ad sh dev br0 || ; ip ad sh dev br0 | grep -q 192\.168\.98\.30 || (ip ad add 192.168.98.30/24 dev br0 && ip link set dev br0 up)"

  # Create a public network, bridged network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #   # Customize the amount of memory on the VM:
    vb.memory = "1024"
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
    vbox_version="5.1.14"
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
    apt-get install apt-transport-https ca-certificates -y
    apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
    grep -q 'deb https://apt.dockerproject.org/repo debian-jessie main' /etc/apt/sources.list.d/docker.list || echo 'deb https://apt.dockerproject.org/repo debian-jessie main' > /etc/apt/sources.list.d/docker.list
    apt-get update
    apt-get install docker-engine -y
    systemctl start docker.service
    docker run busybox echo "hello world!" 

    # Setup bridge/eth2 interface
    apt-get install bridge-utils
    grep -q 'auto eth2' || echo 'auto eth2' >> /etc/network/interfaces
    grep -q 'iface eth2 inet manual' || echo 'iface eth2 inet manual' >> /etc/network/interfaces
    grep -q 'auto brosx0' || echo 'auto brosx0' >> /etc/network/interfaces
    echo 'iface brosx0 inet static
    bridge_ports eth2
        address 192.168.98.30
        broadcast 192.168.98.255
        netmask 255.255.255.0
        gateway 192.168.98.1' >> /etc/network/interfaces
    ifdown eth2 && ifup eth2
    ifdown brosx0 && ifup brosx0
    
  SHELL
end
