# -*- mode: ruby -*-
# vi: set ft=ruby :

# A Vagrantfile to automatically set a small cluster running slurm
# (extrongly based on https://mussolblog.wordpress.com/2013/07/17/
# setting-up-a-testing-slurm-cluster/ with some of my own additions)
# AUTHOR: César Ignacio García Osorio (cgosorio@ubu.es)



#Define the list of machines
slurm_cluster = {
    :controller => {
        :hostname => "controller",
        :ipaddress => "10.10.10.3"
    },
    :server1 => {                                                              
        :hostname => "server1",
        :ipaddress => "10.10.10.4"
    },
    :server2 => {                                                              
        :hostname => "server2",
        :ipaddress => "10.10.10.5"
    },
# ADD more server as needed, but then slurm.conf must be updated
}

#Common provisioning inline script
$script = <<SCRIPT
apt-get update
apt-get install -y -q vim slurm-llnl
echo "10.10.10.3    controller" >> /etc/hosts
echo "10.10.10.4    server1" >> /etc/hosts
echo "10.10.10.5    server2" >> /etc/hosts
wget https://www.dropbox.com/s/skq3m8s0621yzz3/slurm.conf
mv slurm.conf /etc/slurm-llnl/
wget https://www.dropbox.com/s/whjqg1vae70ys8e/munge.key
mv munge.key /etc/munge/
chown munge /etc/munge/munge.key
mkdir /var/slurm; mkdir /var/slurm/log
# To solve the problem reported here: https://github.com/dun/munge/issues/31
sudo /bin/sh -c 'echo "OPTIONS=\"--force\"" >> /etc/default/munge'
SCRIPT

#Provisioning script to start needed services on servers
$ssup = <<SCRIPT
/etc/init.d/munge start
/etc/init.d/slurm-llnl start
SCRIPT

#Provisioning script to start needed services on controller
$csup = <<SCRIPT
/etc/init.d/munge start
slurmctld &
#locale-gen es_ES
loadkeys es
SCRIPT


#Controller provisioning inline script
$ctlscript = <<SCRIPT
# A minimal X to make possible to use sview
apt-get install -y -q xorg xterm lxterminal menu
#apt-get install -y -q lxde # this give a problem with dictionaries-common
#apt-get install -y -q fluxbox # this is not as lighter as jwm
apt-get install -y -q jwm
apt-get install -y -q slurm-llnl-sview
echo "Setting the X"
cat <<EOT > .profile
#CGO added this (warning! if this is added, do not make xterm a login shell)
startx
EOT
cat <<EOT > .xinitrc
#!/bin/sh
#This is to set the spanish layout on X and automatically start lxterminal
setxkbmap es -option terminate:ctrl_alt_bksp
lxterminal &
. /etc/X11/Xsession
EOT
echo "Setting the autologin"
apt-get install -y -q rungetty
## comment out "exec /sbin/getty..."
#sed '/\/sbin\/getty/s/^/#/' -i /etc/init/tty1.conf
# delete the line "exec /sbin/getty..."
sed '/\/sbin\/getty/d' -i /etc/init/tty1.conf
# Add line "exec /sbin/rungetty..."
#sed -e '$aexec /sbin/rungetty --autologin vagrant tty1' -i /etc/init/tty1.conf
echo "exec /sbin/rungetty --autologin vagrant tty1" >> /etc/init/tty1.conf
SCRIPT

Vagrant.configure("2") do |global_config|
    slurm_cluster.each_pair do |name, options|
        global_config.vm.define name do |config|
            #VM configurations
            config.vm.box = "ubuntu/trusty64"
            config.vm.hostname = "#{name}"
            config.vm.network :private_network, ip: options[:ipaddress]

            #VM specifications
            config.vm.provider :virtualbox do |v|
                v.customize ["modifyvm", :id, "--memory", "512"]
            end

            #VM provisioning
            config.vm.provision :shell, :inline => $script

#            #The box ubuntu/trusty64 does not have CD drive
#            #  this was to add a CD drive, only needed in case
#            #  one wants to install the Virtualbox guests additions
#            if name == :controller
#                config.vm.provider :virtualbox do |v|
#                   v.customize ["storagectl", :id, "--name", "IDE Controller", "--add", "ide"]
#                   v.customize ["storageattach", :id, "--storagectl", "IDE Controller", "--port", "0", "--device", "0", "--type", "dvddrive", "--medium", "emptydrive"]
#                   v.customize ["modifyvm", :id, "--boot1", "disk", "--boot2", "dvd"]
#                end
#            end

            #Set controller as not headless and with Xs
            if name == :controller
                config.vm.provider :virtualbox do |v|
                   v.gui = true # to have a gui
                end
                config.vm.provision :shell, :inline => $ctlscript
            end

            #Start daemons and sevices
            if name == :controller
                config.vm.provision :shell, :inline => $csup, run: "always"
            else
                config.vm.provision :shell, :inline => $ssup, run: "always"
            end
        end
    end
end
