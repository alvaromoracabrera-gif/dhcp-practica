# dhcp-practica
Github repository for practice with DHCP

## Creacion de Vagrantfile
Inside the Vagrantfile, we add the following. Here we must create the MAC address for the machines that need it, then in "virtualbox_intnet" we must put **intnet**, and the IP address is the one from the image.



```
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"


  config.vm.define "srv" do |srv|

    srv.vm.network "public_network", bridge: "enp64s0"

    # Red interna
    srv.vm.network "private_network",
      ip: "192.168.57.10",
      virtualbox__intnet: "intnet"


    srv.vm.provision "shell", inline: <<-SHELL
      apt update
      apt install -y isc-dhcp-server

      cp /vagrant/dhcpd.conf /etc/dhcp/dhcpd.conf

      sed -i 's/^INTERFACESv4=.*/INTERFACESv4="eth2"/' /etc/default/isc-dhcp-server

      dhcpd -t
      systemctl enable isc-dhcp-server
      systemctl restart isc-dhcp-server
    SHELL
    
  end

  config.vm.define "client" do |client|


    client.vm.network "private_network",
      type: "dhcp",
      virtualbox__intnet: "intnet"

  end


  config.vm.define "printer" do |printer|


    printer.vm.network "private_network",
      type: "dhcp",
      mac: "080027AABBCC",
      virtualbox__intnet: "intnet"

  end

end
```
## SERVER OPERATIONS:
After making the file, type the command vagrant up for starting up the VMs. Check if the machines are working with vagrant status:

```
usuario@Acer-Nitro-ANV15-51:~/dhcp-def$ vagrant status
Current machine states:

srv                       running (virtualbox)
client                    running (virtualbox)
printer                   running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.

```
After that, we connect to the VM through SSH:

```
usuario@Acer-Nitro-ANV15-51:~/dhcp-def$ vagrant ssh srv
Linux bookworm 6.1.0-29-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.123-1 (2025-01-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
```
Inside the srv VM, apply these two commands: ip route , to see the assignments that were made. ip route for checking that we have a public and a private IP address.

```
vagrant@bookworm:~$ ip route
default via 10.0.2.2 dev eth0 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 
192.168.1.0/24 dev eth1 proto kernel scope link src 192.168.1.87 
192.168.57.0/24 dev eth2 proto kernel scope link src 192.168.57.10 
vagrant@bookworm:~$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:8d:c0:4d brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic eth0
       valid_lft 82241sec preferred_lft 82241sec
    inet6 fd17:625c:f037:2:a00:27ff:fe8d:c04d/64 scope global dynamic mngtmpaddr 
       valid_lft 86353sec preferred_lft 14353sec
    inet6 fe80::a00:27ff:fe8d:c04d/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d7:26:54 brd ff:ff:ff:ff:ff:ff
    altname enp0s8
    inet 192.168.1.87/24 brd 192.168.1.255 scope global dynamic eth1
       valid_lft 82256sec preferred_lft 82256sec
    inet6 fe80::a00:27ff:fed7:2654/64 scope link 
       valid_lft forever preferred_lft forever
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d7:c6:21 brd ff:ff:ff:ff:ff:ff
    altname enp0s9
    inet 192.168.57.10/24 brd 192.168.57.255 scope global eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed7:c621/64 scope link 
       valid_lft forever preferred_lft forever
```
We update the system (sudo apt update && sudo apt upgrade) and we install the dhcp service:
```
vagrant@bookworm:~$ sudo apt install isc-dhcp-server
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
isc-dhcp-server is already the newest version (4.4.3-P1-2).
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
```
In my case it told me I had it previously installed

Next, we should go editing the file /etc/default/isc-dhcp-server with nano (sudo nano /etc/default/isc-dhcp-server)

```
# Defaults for isc-dhcp-server (sourced by /etc/init.d/isc-dhcp-server)

# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
#DHCPDv4_CONF=/etc/dhcp/dhcpd.conf
#DHCPDv6_CONF=/etc/dhcp/dhcpd6.conf

# Path to dhcpd's PID file (default: /var/run/dhcpd.pid).
#DHCPDv4_PID=/var/run/dhcpd.pid
#DHCPDv6_PID=/var/run/dhcpd6.pid

# Additional options to start dhcpd with.
#       Don't use options -cf or -pf here; use DHCPD_CONF/ DHCPD_PID instead
#OPTIONS=""

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
#       Separate multiple interfaces with spaces, e.g. "eth0 eth1".
INTERFACESv4="eth2"
INTERFACESv6=""
```
Now, after leaving the file as shown, type the command: sudo nano /etc/dhcpd/dhcpd.conf . We are gonna edit this file adding to the end the following lines:
```
subnet 192.168.57.0 netmask 255.255.255.0 {
    range 192.168.57.25 192.168.57.50;

    default-lease-time 86400;
    max-lease-time 691200;

    option broadcast-address 192.168.57.255;
    option routers 192.168.57.2;
    option domain-name-servers 192.168.57.3, 4.4.4.4;
    option domain-name "amc.test";
}
host printer {
    hardware ethernet 08:00:27:aa:bb:cc;
    fixed-address 192.168.57.100;
}
```
After editing the file, we should input: sudo systemctl restart isc-dhcp-server . Then: sudo systemctl status isc-dhcp-server
```subnet 192.168.57.0 netmask 255.255.255.0 {
    range 192.168.57.25 192.168.57.50;

    default-lease-time 86400;
    max-lease-time 691200;

    option broadcast-address 192.168.57.255;
    option routers 192.168.57.2;
    option domain-name-servers 192.168.57.3, 4.4.4.4;
    option domain-name "amc.test";
}

host printer {
    hardware ethernet 08:00:27:aa:bb:cc;
    fixed-address 192.168.57.100;
    option host-name "printer";
    option domain-name "amc.test";
}
```
And thats how we have ssh service active and working in our server.
## CLIENT OPERATIONS:
