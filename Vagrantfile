Vagrant.configure("2") do |config|

  config.vm.define "ipa" do |i|
    i.vm.box = "centos/7"
    i.vm.box_version = "1905.1"
    i.vm.hostname = "ipa.netrunner.lan"
    i.vm.network "private_network", ip: "192.168.16.1"
    i.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "4096"]
    end
    i.vm.provision "shell", inline: <<-SHELL
      mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
      yum update -y
      systemctl start firewalld
      systemctl enable firewalld
      firewall-cmd --add-service=freeipa-ldap --add-service=freeipa-ldaps --add-service=dns --zone=public --permanent
      firewall-cmd --reload
      yum install ipa-server -y
      yum install ipa-server-dns -y
      sed -i '1s/127.0.0.1.*/192.168.16.1 ipa.netrunner.lan/' /etc/hosts      
#      ipa-server-install -U -r NETRUNNER.LAN -n netrunner.lan -p DM123456 -a AD123456 --mkhomedir --hostname=ipa.netrunner.lan --setup-dns --auto-forwarders --forward-policy=only --no-reverse
    SHELL
  end

  config.vm.define "client" do |c|
    c.vm.box = "centos/7"
    c.vm.box_version = "1905.1"
    c.vm.hostname = "client.netrunner.lan"
    c.vm.network "private_network", ip: "192.168.16.2"
    c.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "512"]
    end
    c.vm.provision "shell", inline: <<-SHELL
      mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
      sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
      systemctl restart sshd
#      yum update -y
#      systemctl start firewalld
#      systemctl enable firewalld
#      firewall-cmd --add-service=freeipa-ldap --add-service=freeipa-ldaps --add-service=dns --zone=public --permanent
#      firewall-cmd --reload
#      yum install ipa-client -y
#      nmcli con mod 'System eth1' ipv4.dns "192.168.16.1"
#      nmcli con mod 'System eth0' ipv4.ignore-auto-dns yes
#      nmcli networking off && nmcli networking on
#      ipa-client-install -U --mkhomedir --domain=netrunner.lan --server=ipa.netrunner.lan --realm=NETRUNNER.LAN --hostname=client.netrunner.lan --force-ntpd -p admin --password=AD123456
    SHELL
  end
end

