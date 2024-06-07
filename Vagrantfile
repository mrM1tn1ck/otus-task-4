# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'

$script = <<SCRIPT
sudo su
# apt update && apt upgrade -y
apt install -y mdadm smartmontools hdparm gdisk
mdadm --zero-superblock --force /dev/sd{c,d,e,f}
mdadm --create --verbose /dev/md0 -l 10 -n 4 /dev/sd{c,d,e,f}
sleep 5
echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
parted -s /dev/md0 mklabel gpt
parted /dev/md0 mkpart primiry ext4 0% 25%
parted /dev/md0 mkpart primiry ext4 25% 50%
parted /dev/md0 mkpart primiry ext4 50% 75%
parted /dev/md0 mkpart primiry ext4 75% 100%
for i in $(seq 1 4); do mkfs.ext4 /dev/md0p$i; done
mkdir -p /raid/part{1,2,3,4}
for i in $(seq 1 4); do mount /dev/md0p$i /raid/part$i; done
SCRIPT

Vagrant.configure("2") do |config|
    # for all
    config.vm.synced_folder ".", "/vagrant"

    config.vm.define "mdadm" do |mdadm|
        mdadm.vm.box = "ubuntu/jammy64"
        mdadm.vm.hostname = "mdadm"
        mdadm.vm.network "private_network", ip: "192.168.10.10", adapter: 2, netmask: "255.255.255.0", virtualbox_inet: "vmnet"
        mdadm.vm.provider :virtualbox do |vm|
            vm.name = "mdadm"
            vm.memory = "2048"
            vm.cpus = 1
        end
        # mdadm.vm.disk :disk, size: "1GB", name: "extra_storage"
        (0..3).each do |i|
            mdadm.vm.disk :disk, size: "250MB", name: "disk-#{i}"
        end
        mdadm.vm.provision "shell" do |script|
            script.inline = $script
        end    
    end
end