# Deployment of Vagrant Ubuntu Cluster with LAMP Stack

## Overview
This guide explains the automated deployment of two Vagrant-based Ubuntu systems, designated as Master and Slave, with an integrated LAMP stack on both systems. The Master node acts as a control system, and the Slave node is managed by the Master node.

## Prerequisites

- [Vagrant](https://www.vagrantup.com/) 
- [VirtualBox](https://www.virtualbox.org/)

## Objective
Develop a bash script to orchestrate the automated deployment of two Vagrant-based Ubuntu systems, designated as 'Master' and 'Slave', with an integrated LAMP stack on both systems.

## Specifications:

### Infrastructure Configuration:

#### Vagrantfile configuration:

I used `vagrant init` to create a Vagrantfile in the directory. I added the following configurations to the Vagrantfile:

```ruby
Vagrant.configure("2") do |config|
  config.vm.define "master" do |master|
    master.vm.box = "generic/ubuntu2204"
    master.vm.network "private_network", ip: "192.168.33.10"
    master.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
    end
  end
  
  config.vm.define "slave" do |slave|
    slave.vm.box = "generic/ubuntu2204"
    slave.vm.network "private_network", ip: "192.168.33.11"
    slave.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
    end
  end
  ```

#### Deploy two Ubuntu systems:

I added the following commads to the bash script to automate the deployment of two Vagrant-based Ubuntu systems (Master and Slave) with an integrated LAMP stack on both. These commands will create two VMs on Ubuntu 22.04, it will install the LAMP stack (Apache, MySQL, PHP) on both VMs, and it will configure the Master to act as a control system by creating a simple PHP script:

```bash
#!/bin/bash

MASTER_VM="master"
SLAVE_VM="slave"
MASTER_IP="192.168.33.10"
SLAVE_IP="192.168.33.11"
vagrant init generic/ubuntu2204
vagrant up $MASTER_VM --provider virtualbox
vagrant up $SLAVE_VM --provider virtualbox
echo "Installing LAMP stack on Master..."
vagrant ssh $MASTER_VM -c "sudo apt update"
vagrant ssh $MASTER_VM -c "sudo apt install -y apache2 mysql-server php libapache2-mod-php php-mysql"
echo "Configuring Master as control system..."
vagrant ssh $MASTER_VM -c "echo '<?php phpinfo(); ?>' | sudo tee /var/www/html/index.php"
vagrant ssh $MASTER_VM -c "sudo systemctl enable apache2"
vagrant ssh $MASTER_VM -c "sudo systemctl start apache2"
echo "Installing LAMP stack on Slave..."
vagrant ssh $SLAVE_VM -c "sudo apt update"
vagrant ssh $SLAVE_VM -c "sudo apt install -y apache2 mysql-server php libapache2-mod-php php-mysql"
echo "Connecting Slave to Master for management..."
vagrant ssh $SLAVE_VM -c "sudo sed -i 's/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf"
vagrant ssh $SLAVE_VM -c "sudo systemctl enable apache2"
vagrant ssh $SLAVE_VM -c "sudo systemctl start apache2"
echo "Deployment completed successfully."
echo "Master IP: $MASTER_IP"
echo "Slave IP: $SLAVE_IP"
```
### User Management:

I created a user named AltSchool and granted AltSchool user root (superuser) privileges on the Master node by adding the following commands to the `bash.sh` file:

```bash
echo "Creating user AltSchool on Master..."
vagrant ssh $MASTER_VM -c "sudo adduser $NEW_USER"
echo "Granting superuser privileges to AltSchool..."
vagrant ssh $MASTER_VM -c "sudo usermod -aG sudo $NEW_USER"
```
### Inter-node Communication:
To ensure that the Master node (AltSchool user) can seamlessly SSH into the Slave node without requiring a password, I set up the SSH key-based authentication between the two nodes by adding the following commands to the `bash.sh` file:

```bash 
echo "Configuring SSH key-based authentication from Master to Slave..."
vagrant ssh $MASTER_VM -c "sudo su - $NEW_USER -c 'ssh-keygen -t rsa -b 2048 -N \"\" -f ~/.ssh/id_rsa'"
vagrant ssh $MASTER_VM -c "sudo su - $NEW_USER -c 'ssh-copy-id $NEW_USER@$SLAVE_IP'"
```
### Data Management and Transfer:

#### On initiation:

I used the following bash commands to copy the contents of /mnt/AltSchool directory from the Master node to /mnt/AltSchool/slave on the Slave node: 

```bash 
echo "Copying contents from Master to Slave..."
vagrant ssh $MASTER_VM -c "sudo su - $NEW_USER -c 'rsync -avz /mnt/AltSchool/ $NEW_USER@$SLAVE_IP:/mnt/AltSchool/slave/'"
```
#### Process Monitoring:

I added the following commands to the `bash.sh` file for the Master node to display an overview of the Linux process management, showcasing currently running processes:

```bash
echo "Overview of currently running processes on Master:"
vagrant ssh $MASTER_VM -c "ps aux"
```
## LAMP Stack Deployment:
I added the following commands to the `bash.sh` to install a LAMP (Linux, Apache, MySQL, PHP) stack on both nodes, to ensure Apache is running and set to start on boot, to secure the MySQL installation and initialize it with a default user and password, and to validate PHP functionality with Apache:

```bash
MYSQL_ALT_PASSWORD="Alt%sch00luser"
MYSQL_ROOT_PASSWORD="Alt%sch00l"
echo "Installing LAMP stack on Master..."
vagrant ssh $MASTER_VM -c "sudo apt update"
vagrant ssh $MASTER_VM -c "sudo apt install -y apache2 mysql-server php libapache2-mod-php php-mysql"
echo "Installing LAMP stack on Slave..."
vagrant ssh $SLAVE_VM -c "sudo apt update"
vagrant ssh $SLAVE_VM -c "sudo apt install -y apache2 mysql-server php libapache2-mod-php php-mysql"
echo "Configuring Apache on Master..."
vagrant ssh $MASTER_VM -c "sudo systemctl enable apache2"
vagrant ssh $MASTER_VM -c "sudo systemctl start apache2"
echo "Configuring Apache on Slave..."
vagrant ssh $SLAVE_VM -c "sudo systemctl enable apache2"
vagrant ssh $SLAVE_VM -c "sudo systemctl start apache2"
echo "Securing MySQL installation on Master..."
vagrant ssh $MASTER_VM -c "sudo mysql_secure_installation"
echo "Initializing MySQL on Master with default user and password..."
vagrant ssh $MASTER_VM -c "sudo mysql -uroot -p$MYSQL_ROOT_PASSWORD -e \"CREATE USER '$MYSQL_ALT_USER'@'localhost' IDENTIFIED BY '$MYSQL_ALT_PASSWORD';\""
vagrant ssh $MASTER_VM -c "sudo mysql -uroot -p$MYSQL_ROOT_PASSWORD -e \"GRANT ALL PRIVILEGES ON *.* TO '$MYSQL_ALT_USER'@'localhost' WITH GRANT OPTION;\""
vagrant ssh $MASTER_VM -c "sudo mysql -uroot -p$MYSQL_ROOT_PASSWORD -e \"FLUSH PRIVILEGES;\""
echo "Securing MySQL installation on Slave..."
vagrant ssh $SLAVE_VM -c "sudo mysql_secure_installation"
echo "Initializing MySQL on Slave with default user and password..."
vagrant ssh $SLAVE_VM -c "sudo mysql -uroot -p$MYSQL_ROOT_PASSWORD -e \"CREATE USER '$MYSQL_ALT_USER'@'localhost' IDENTIFIED BY '$MYSQL_ALT_PASSWORD';\""
vagrant ssh $SLAVE_VM -c "sudo mysql -uroot -p$MYSQL_ROOT_PASSWORD -e \"GRANT ALL PRIVILEGES ON *.* TO '$MYSQL_ALT_USER'@'localhost' WITH GRANT OPTION;\""
vagrant ssh $SLAVE_VM -c "sudo mysql -uroot -p$MYSQL_ROOT_PASSWORD -e \"FLUSH PRIVILEGES;\""
echo "Creating PHP test script on Master..."
vagrant ssh $MASTER_VM -c "echo '<?php phpinfo(); ?>' | sudo tee /var/www/html/phpinfo.php"
echo "Creating PHP test script on Slave..."
vagrant ssh $SLAVE_VM -c "echo '<?php phpinfo(); ?>' | sudo tee /var/www/html/phpinfo.php"
```
## Conclusion
In order to make the `bash.sh` file executable, I used the following command:
```bash
chmod +x bash.sh
```
After making `bash.sh` executable, I used `vagrant up` to run the 'master' and the 'slave' VMs.

After running the VMs, I used the `./bash.sh` command to execute the script and to begin the automated deployment of Vagrant Ubuntu Cluster with LAMP Stack. The deployment was successful and the following result showed:
```bash
Overview of currently running processes on Master:
Connecting Slave to Master for management...
Deployment completed successfully.
Master IP: 192.168.33.10
Slave IP: 192.168.33.11
```
 




























