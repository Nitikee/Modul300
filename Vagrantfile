#Setting BOX_IMAGE for all VMs"
BOX_IMAGE = "ubuntu/trusty64"

#Setting numbers of webserver to be created
WEB_COUNT = 2

#Provisioning script for loadbalancing on "lbalancer" server as variable
#A file load-balancer.conf is going to be created with the loadbalancing options
#The Webserver are 192.168.200.10 and 192.168.200.11
#You can add as many server as you want to
#After this the default site in /etc/nginx/sites-enabled/default will be removed
#OpenVPN is going to be installed
#For connection in and form the internet the "lbalancer" server is goint to be connected to "nitinankeel.ch"
#The configration file for OpenVPN is in your shared folder

$script = <<SCRIPT
  echo I am provisioning...
  cat <<EOF > /etc/nginx/conf.d/load-balancer.conf
upstream backend {
  least_conn;
  server 192.168.200.10;
  server 192.168.200.11;
}
server {
  listen 80;
  location / {
    proxy_pass http://backend;
 }
}
EOF
sudo rm /etc/nginx/sites-enabled/default
sudo service nginx restart
sudo apt-get -y install openvpn
sudo openvpn --config /home/vagrant/svmb01.ovpn --daemon
SCRIPT

#Vagrant configurations
Vagrant.configure("2") do |config|
  
  #Configurations for the loadbalancer
  config.vm.define "lbalancer" do |subconfig|
    #Get the Box-Image
    subconfig.vm.box = BOX_IMAGE
    #Configure name for VM in vagrant --> Not Virtualbox VM name!
    subconfig.vm.hostname = "lbalancer"

    #Creating VM Net "LAN" with the netaddress "192.168.200.0/24"
    #If you don't name this network, virtualbox will create a Hostonly adapter --> VM's cant communicate with each other (We dont wan't this)
    subconfig.vm.network :private_network, ip: "192.168.200.1",
    virtualbox__intnet: "LAN"

    #Creating shared folder for storing the .ovpn file
    #The synced folder in your VM is in /home/vagrant
    subconfig.vm.synced_folder "lbalancer", "/home/vagrant"
    
    #Run the provision script
    subconfig.vm.provision "shell", inline:$script
    
    #These are the settings for your virtualbox
    #Configure the virtualbox with VM Name und amount of memmory
    subconfig.vm.provider "virtualbox" do |v|
        v.name = "lbalancer"
        v.memory = 1024
      end
  end # End of "lbalancer" configuration
  
  #Starting configuration for webserver
  #Loop foreach webserver you defined
  (1..WEB_COUNT).each do |i|
    
    #Creating 'i' amount of webserver with the bellow configurations
    config.vm.define "web#{i}" do |subconfig|
      
      #Box-Image
      subconfig.vm.box = BOX_IMAGE

      #Hostname of vagrant vm
      #{i} is a integer -> eg. web1 or web2
      subconfig.vm.hostname = "web#{i}"

      #create Network and asssing IPs
      #{i + 9} --> eg. "i" is equal 1; add 9
      subconfig.vm.network :private_network, ip: "192.168.200.#{i + 9}",
      
      #The name of the network needs to be the same.
      virtualbox__intnet: "LAN"
      
      #Synced foler web1 and web2
      subconfig.vm.synced_folder "web#{i}", "/home/vagrant"

      #Copy the index.html file too the www/html folder from nginx
      #I couldn't make the synced folder in the www/html file because nginx will complain at the installation
      subconfig.vm.provision "shell", inline: <<-SHELL
      cp /home/vagrant/index.html /usr/share/nginx/html/
      SHELL

      #Virtualbox settings
      subconfig.vm.provider "virtualbox" do |v|
        v.name = "web#{i}"
        v.memory = 512
      end
    end
  end #End of Webserver configurations

  #Settings for all machines
  #Install updates
  #Add UFW Rule 80
  #Enable UFW
  #Install NGINX
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo ufw allow 80
    sudo ufw allow 22
    sudo ufw -f enable
    sudo apt-get -y install nginx
  SHELL
end