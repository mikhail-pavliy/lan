# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
    :inetRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-loc"},
                ]
  },
  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-loc"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "directors-loc"},
                   {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "ohw-loc"},
                   {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "wifi-loc"},
                ]
  },
  
  :office1Router => {
    	:box_name => "centos/7",
    	:net => [
               {ip: '192.168.254.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1Router"},
               {ip: '192.168.2.1', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "office1-dev-loc"},
               {ip: '192.168.2.65', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "office1-testservers-loc"},
               {ip: '192.168.2.129', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "office1-managers-loc"},
               {ip: '192.168.2.193', adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "office1-hardware-loc"}
            ]
  },
  
  :office2Router => {
   	 :box_name => "centos/7",
   	 :net => [
              {ip: '192.168.253.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office2Router"},
              {ip: '192.168.1.1', adapter: 3, netmask: "255.255.255.128", virtualbox__intnet: "office2-dev-loc"},
              {ip: '192.168.1.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "office2-testservers-loc"},
              {ip: '192.168.1.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "office2-hardware-loc"}
            ]
  }, 

   :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "directors-loc"}
                ]
  },
  
  
  :office1Server => {
    :box_name => "centos/7",
    :net => [
               {ip: '192.168.2.2', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "office1-dev-loc"}
            ]
  },

    
  :office2Server => {
    :box_name => "centos/7",
    :net => [
               {ip: '192.168.1.2', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "office2-dev-loc"}
            ]
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        
        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        
        case boxname.to_s
        when "inetRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sudo bash -c 'echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf'; sudo sysctl -p
            sudo yum install -y iptables-services; sudo systemctl enable iptables && sudo systemctl start iptables;
            sudo iptables -F; sudo iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE; sudo service iptables save
            sudo bash -c 'echo "192.168.0.0/16 via 192.168.255.2 dev eth1" > /etc/sysconfig/network-scripts/route-eth1'; sudo systemctl restart network
            sudo echo "192.168.255.1 inetRouter" >> /etc/hosts
			sudo echo "192.168.255.2 centralRouter" >> /etc/hosts
			sudo echo "192.168.254.2 office1Router" >> /etc/hosts
			sudo echo "192.168.253.2 office2Router" >> /etc/hosts
			sudo echo "192.168.0.2 centralServer" >> /etc/hosts
			sudo echo "192.168.2.2 office1Server" >> /etc/hosts
			sudo echo "192.168.1.2 office2Server" >> /etc/hosts
            sudo reboot
            SHELL
        when "centralRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sudo bash -c 'echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf'; sudo sysctl -p
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            sudo nmcli connection modify "System eth1" +ipv4.addresses "192.168.254.1/30"; sudo nmcli connection modify "System eth1" +ipv4.addresses "192.168.253.1/30"
            sudo bash -c 'echo "192.168.2.0/24 via 192.168.254.2 dev eth1" > /etc/sysconfig/network-scripts/route-eth1'
            sudo bash -c 'echo "192.168.1.0/24 via 192.168.253.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1'
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo echo "192.168.255.1 inetRouter" >> /etc/hosts
			sudo echo "192.168.255.2 centralRouter" >> /etc/hosts
			sudo echo "192.168.254.2 office1Router" >> /etc/hosts
			sudo echo "192.168.253.2 office2Router" >> /etc/hosts
			sudo echo "192.168.0.2 centralServer" >> /etc/hosts
			sudo echo "192.168.2.2 office1Server" >> /etc/hosts
			sudo echo "192.168.1.2 office2Server" >> /etc/hosts
            sudo reboot
            SHELL
        when "office1Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sudo bash -c 'echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf'; sudo sysctl -p
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.254.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo echo "192.168.255.1 inetRouter" >> /etc/hosts
			sudo echo "192.168.255.2 centralRouter" >> /etc/hosts
			sudo echo "192.168.254.2 office1Router" >> /etc/hosts
			sudo echo "192.168.253.2 office2Router" >> /etc/hosts
			sudo echo "192.168.0.2 centralServer" >> /etc/hosts
			sudo echo "192.168.2.2 office1Server" >> /etc/hosts
			sudo echo "192.168.1.2 office2Server" >> /etc/hosts
            sudo reboot
            SHELL
		when "office2Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sudo bash -c 'echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.conf'; sudo sysctl -p
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.253.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo echo "192.168.255.1 inetRouter" >> /etc/hosts
			sudo echo "192.168.255.2 centralRouter" >> /etc/hosts
			sudo echo "192.168.254.2 office1Router" >> /etc/hosts
			sudo echo "192.168.253.2 office2Router" >> /etc/hosts
			sudo echo "192.168.0.2 centralServer" >> /etc/hosts
			sudo echo "192.168.2.2 office1Server" >> /etc/hosts
			sudo echo "192.168.1.2 office2Server" >> /etc/hosts
            sudo reboot
            SHELL
	    when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
             sudo echo "192.168.255.1 inetRouter" >> /etc/hosts
			sudo echo "192.168.255.2 centralRouter" >> /etc/hosts
			sudo echo "192.168.254.2 office1Router" >> /etc/hosts
			sudo echo "192.168.253.2 office2Router" >> /etc/hosts
			sudo echo "192.168.0.2 centralServer" >> /etc/hosts
			sudo echo "192.168.2.2 office1Server" >> /etc/hosts
			sudo echo "192.168.1.2 office2Server" >> /etc/hosts
            sudo reboot
            SHELL
        when "office1Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.2.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo echo "192.168.255.1 inetRouter" >> /etc/hosts
			sudo echo "192.168.255.2 centralRouter" >> /etc/hosts
			sudo echo "192.168.254.2 office1Router" >> /etc/hosts
			sudo echo "192.168.253.2 office2Router" >> /etc/hosts
			sudo echo "192.168.0.2 centralServer" >> /etc/hosts
			sudo echo "192.168.2.2 office1Server" >> /etc/hosts
			sudo echo "192.168.1.2 office2Server" >> /etc/hosts
            sudo reboot
            SHELL
        
         when "office2Server"
           box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.1.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            sudo systemctl restart network
            sudo echo "192.168.255.1 inetRouter" >> /etc/hosts
			sudo echo "192.168.255.2 centralRouter" >> /etc/hosts
			sudo echo "192.168.254.2 office1Router" >> /etc/hosts
			sudo echo "192.168.253.2 office2Router" >> /etc/hosts
			sudo echo "192.168.0.2 centralServer" >> /etc/hosts
			sudo echo "192.168.2.2 office1Server" >> /etc/hosts
			sudo echo "192.168.1.2 office2Server" >> /etc/hosts
            sudo reboot
            SHELL
        end

      end

  end
  
  
end
