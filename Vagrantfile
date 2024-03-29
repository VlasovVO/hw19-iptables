# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:inetRouter => {
        :box_name => "centos/6",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                ]
  },
  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.255.5', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.255.9', adapter: 4, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.0.1', adapter: 5, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.0.33', adapter: 6, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.0.65', adapter: 7, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},
                ]
  },

  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                ]
  },
  :inetRouter2 => {
    :box_name => "centos/7",
    :net => [
      {ip: '192.168.255.6', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
      {adapter: 3, auto_config: true, virtualbox__intnet: true},
      {adapter: 4, auto_config: false, virtualbox__intnet: true},

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
            echo "192.168.0.0/16 via 192.168.255.2" >  /etc/sysconfig/network-scripts/route-eth1
            echo -e 'net.ipv4.conf.all.^Crwarding=1\nnet.ipv4.ip_forward=1' >> /etc/sysctl.d/99-override.conf
            sysctl net.ipv4.conf.all.forwarding=1
            sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
            sed -i 's/PermitRootLogin no/PermitRootLogin yes/g' /etc/ssh/sshd_config
            service sshd restart
            cp /vagrant/files/iptables /etc/sysconfig/
            service iptables restart


            service network restart
            SHELL
      when "centralRouter"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo -e 'net.ipv4.conf.all.^Crwarding=1\nnet.ipv4.ip_forward=1' >> /etc/sysctl.d/99-override.conf
            sysctl net.ipv4.conf.all.forwarding=1
            echo -e '192.168.1.0/24 via 192.168.255.10' > /etc/sysconfig/network-scripts/route-eth3
            echo -e '192.168.2.0/24 via 192.168.255.6' > /etc/sysconfig/network-scripts/route-eth2
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            SHELL
      when "centralServer"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            yum install epel-release -y
            yum install nginx -y
            systemctl start nginx

            systemctl restart network
            SHELL

      when "inetRouter2"
        config.vm.network "forwarded_port", guest: 8080, host: 8080
        box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            sysctl net.ipv4.conf.all.forwarding=1
            echo -e 'net.ipv4.conf.all.forwarding=1\nnet.ipv4.ip_forward=1' >> /etc/sysctl.d/99-override.conf
            echo "GATEWAY=192.168.255.5" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            yum install iptables-services -y
            cp /vagrant/files/iptables-inetrouter2 /etc/sysconfig/iptables
            systemctl restart iptables
            systemctl restart network
            SHELL
      end
    end
  end
end
