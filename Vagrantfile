# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :lvm_1 => {
        :box_name => "ubuntu/jammy64", #ubuntu/jammy64
        :ip_addr => '192.168.56.101',
        :vm_name => 'lvm-1'
  
  },

  
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxconfig[:vm_name]
      box.vm.network "private_network", ip: boxconfig[:ip_addr]

      # (0..4).each do |i|
      # box.vm.disk :disk, size: "250MB", name: "hdd-#{i}"
      # end
      box.vm.disk :disk, size: "10GB", name: "hdd-0"
      box.vm.disk :disk, size: "2GB", name: "hdd-1"
      box.vm.disk :disk, size: "1GB", name: "hdd-2"
      box.vm.disk :disk, size: "1GB", name: "hdd-3"
      
      box.vm.provider :virtualbox do |vb|
        vb.cpus = 1
        vb.memory = 1024
 #      box.vm.disk :disk, size: "1GB", name: "extra_storage"

      end
      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
        sudo sed -i 's/\#PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        systemctl restart sshd
        apt install -y mdadm smartmontools hdparm fdisk
      SHELL

    end
  end
end