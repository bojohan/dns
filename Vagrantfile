
$gateway_setup = <<SCRIPT
iptables -A FORWARD -o eth0 -i eth1 -s 192.168.0.0/24 -m conntrack --ctstate NEW -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -t nat -F POSTROUTING
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE


iptables -I FORWARD 1 -j LOG --log-prefix='[gateway] '



iptables-save | sudo tee /etc/iptables.sav
awk  '/^exit 0/{print "iptables-restore < /etc/iptables.sav"}1' /etc/rc.local > tmp && mv tmp /etc/rc.local

sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
sed -i '/net.ipv4.ip_forward=1/s/^#//g' /etc/sysctl.conf

apt-get install dnsmasq -y
echo "dhcp-range=192.168.0.50,192.168.0.150,1h" >> /etc/dnsmasq.conf
echo "expand-hosts" >> /etc/dnsmasq.conf
echo " local=/itg.lan.test/" >> /etc/dnsmasq.conf
echo " domain=itg.lan.test" >> /etc/dnsmasq.conf
service dnsmasq restart

SCRIPT

$client_setup = <<SCRIPT
sed -e '/nameserver 10.0.2.3/s/^/#/g'  -i  /etc/resolv.conf
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: "echo Hello"

  config.vm.define "gateway" do |gateway|
    gateway.vm.hostname = "gateway"
    gateway.vm.box = "ubuntu/trusty64"
    gateway.vm.network "private_network", ip: "192.168.0.1",
        virtualbox__intnet: "happynetwork_dhcp"
    gateway.vm.provision "shell", inline: $gateway_setup
  end

  config.vm.define "client1" do |client1|
    client1.vm.hostname = "client1"
    client1.vm.box = "ubuntu/trusty64"
    client1.vm.network "private_network", type: "dhcp",
      virtualbox__intnet: "happynetwork_dhcp"
    client1.vm.provision "shell", inline: $client_setup
  end

  config.vm.define "client2" do |client2|
    client2.vm.hostname = "client2"
    client2.vm.box = "ubuntu/trusty64"
    client2.vm.network "private_network", type: "dhcp",
      virtualbox__intnet: "happynetwork_dhcp"
    client2.vm.provision "shell", inline: $client_setup
  end


end
