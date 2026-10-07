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
