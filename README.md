# Vagrant 

Typing `vagrant` from the command line will display a list of all available commands.

Be sure that you are in the same directory as the Vagrantfile when running these commands!

# Creating a VM
- `vagrant init`           -- Initialize Vagrant with a Vagrantfile and ./.vagrant directory, using no specified base image. Before you can do vagrant up, you'll need to specify a base image in the Vagrantfile.
- `vagrant init <boxpath>` -- Initialize Vagrant with a specific box. To find a box, go to the [public Vagrant box catalog](https://app.vagrantup.com/boxes/search). When you find one you like, just replace it's name with boxpath. For example, `vagrant init ubuntu/trusty64`.

# Starting a VM
- `vagrant up`                  -- starts vagrant environment (also provisions only on the FIRST vagrant up)
- `vagrant resume`              -- resume a suspended machine (vagrant up works just fine for this as well)
- `vagrant provision`           -- forces reprovisioning of the vagrant machine
- `vagrant reload`              -- restarts vagrant machine, loads new Vagrantfile configuration
- `vagrant reload --provision`  -- restart the virtual machine and force provisioning

# Getting into a VM
- `vagrant ssh`           -- connects to machine via SSH
- `vagrant ssh <boxname>` -- If you give your box a name in your Vagrantfile, you can ssh into it with boxname. Works from any directory.

# Stopping a VM
- `vagrant halt`        -- stops the vagrant machine
- `vagrant suspend`     -- suspends a virtual machine (remembers state)

# Cleaning Up a VM
- `vagrant destroy`     -- stops and deletes all traces of the vagrant machine
- `vagrant destroy -f`   -- same as above, without confirmation

# Boxes
- `vagrant box list`              -- see a list of all installed boxes on your computer
- `vagrant box add <name> <url>`  -- download a box image to your computer
- `vagrant box outdated`          -- check for updates vagrant box update
- `vagrant box remove <name>`   -- deletes a box from the machine
- `vagrant package`               -- packages a running virtualbox env in a reusable box

# Saving Progress
-`vagrant snapshot save [options] [vm-name] <name>` -- vm-name is often `default`. Allows us to save so that we can rollback at a later time

# Tips
- `vagrant -v`                    -- get the vagrant version
- `vagrant status`                -- outputs status of the vagrant machine
- `vagrant global-status`         -- outputs status of all vagrant machines
- `vagrant global-status --prune` -- same as above, but prunes invalid entries
- `vagrant provision --debug`     -- use the debug flag to increase the verbosity of the output
- `vagrant push`                  -- yes, vagrant can be configured to [deploy code](http://docs.vagrantup.com/v2/push/index.html)!
- `vagrant up --provision | tee provision.log`  -- Runs `vagrant up`, forces provisioning and logs all output to a file

# Plugins
- [vagrant-hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater) : `$ vagrant plugin install vagrant-hostsupdater` to update your `/etc/hosts` file automatically each time you start/stop your vagrant box.
- [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd) : `$ vagrant plugin install vagrant-winnfsd` the plugin adds NFS support to vagrant in Windows.
- [vagrant-vbguest](https://github.com/dotless-de/vagrant-vbguest) : `$ vagrant plugin install vagrant-vbguest` 

# Notes
- If you are using [VVV](https://github.com/varying-vagrant-vagrants/vvv/), you can enable xdebug by running `vagrant ssh` and then `xdebug_on` from the virtual machine's CLI.


# Build a custom Vagrant box 


## Configure the Virtual Hardware

Create a new Virtual Machine with the following settings:

	Name: focal_fossa
	Type: Linux
	Version: Ubuntu64
	Memory Size: 1024MB
	New Virtual Disk: [Type: VMDK, Size: 25 GB]

Modify the hardware settings of the virtual machine for performance and because SSH needs port-forwarding enabled for the vagrant user:

    Disable audio
    Disable USB
    Ensure Network Adapter 1 is set to NAT
    Add this port-forwarding rule: [Name: SSH, Protocol: TCP, Host IP: blank, Host Port: 2222, Guest IP: blank, Guest Port: 22]

Mount the Linux Distro ISO and boot up the server.

## Install The Operating System

Follow the on-screen prompts. And when prompted to setup a user, set the user to vagrant and the password to vagrant. 

## Setup the Super User

Vagrant must be able run sudo commands without a password prompt. Anything in the /etc/sudoers.d/* folder is included in the “sudoers” privileges when created by the root user.

```
sudo -s
visudo -f /etc/sudoers.d/vagrant

# add vagrant user
vagrant ALL=(ALL) NOPASSWD:ALL
```

## Updating The Operating System

```
sudo apt-get update
sudo apt-get upgrade -y 
sudo apt-get dist-upgrade -y
sudo update-grub 
sudo apt-get install linux-headers-$(uname -r) build-essential dkms
sudo apt-get autoremove 
sudo apt-get autoclean
```
## VirtualBox Guest Additions

VirtualBox Guest Additions must be installed so that things such as shared folders can function. Installing guest additions also usually improves performance since the guest OS can make some optimizations by knowing it is running within VirtualBox.

```
sudo mount /dev/cdrom /media/cdrom
sudo sh /media/cdrom/VBoxLinuxAdditions.run
```

## “Zero out” the drive

```
sudo dd if=/dev/zero of=/EMPTY bs=1M
sudo rm -f /EMPTY
```

## Vagrant public key

`ssh-keygen`
```
wget https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub \
-O /home/vagrant/.ssh/authorized_keys
```
`chmod 0600 /home/vagrant/.ssh/authorized_keys`
`chown -R vagrant /home/vagrant/.ssh`

`sudo update-alternatives --config editor`

## Packaging the box
```
vagrant package --base focal_fossa
vagrant box add focal_fossa package.box
vagrant init focal_fossa
```
