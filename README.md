# FID_dhcp_config
Includes copies of various files from the root directory that allow the DHCP server to work. They are meant to be perserved here as working copies of the actual files of the system, as well as reused if needed.

__The files `dhcpd_dhcp.conf` and `dhcpd_dhcp3.conf` both need to be changed to `dhcpd.conf` in order to work. All file locations are specified on the top line of each file.__

## Equipment Used:
* Apple IMAC (~2011)
* Linux Mint w/ Cinnammon Desktop
* Dlinkgreen Switch

## Instructions on setting up the DHCP server for the HP35900E Mulitchannel Interface from scratch:
In Terminal run:

`$ ip address`

Look for something in the output like 'enp....' or 'eth0' (aka interface name), and within that paragraph look for 'inet'
which will be the ip address assigned to your computer by your router. Write down both the interface name, and inet as
you'll need them later.

`$ sudo apt-get update`

`$ sudo apt-get install isc-dhcp-server`

Open the dhcp config file, and add the following:

`$ sudo nano /etc/dhcp/dhcpd.conf`

Anywhere it says 192.168.1.some_number should be replaced with the first three sets of numbers from the 'inet' you 
wrote down earlier.
Hence if my inet was '10.0.0.5', the subnet below would become 10.0.0.0
```
subnet 192.168.1.0 netmask 255.255.255.0 {
    authoritative;
    range 192.168.1.20 192.168.1.100;
    default-lease-time 3600;
    max-lease-time 7200;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.1.255;
    option routers 192.168.1.1;
    option domain-name-servers 192.168.1.1, 8.8.8.8;
    option domain-name "example.com";
}

option space hp35900e;
option buffer-packing code 146 = Boolean;
group {
        option buffer-packing on;
        host hp35900e.1 { hardware Ethernet 08:00:09:99:31:E5; fixed-address
        192.168.1.2;}
}
```

FYI: The 'authoritative' decleration above is meant to be used when you want this dhcp server to be the offical one for
your local network.

__I have ommitted the dhcp3, dhcpd.conf script as I believe it is not needed.__

Open the isc-dhcp-server file, and add the following:

`$ nano /etc/default/isc-dhcp-server`

```
INTERFACES = "<your interface's name>"
```

Now run the following commands:

`$ sudo systemctl restart isc-dhcp-server`

`$ sudo systemctl status isc-dhcp-server`

Within the output of the previous command you should some green text that says 'active (running)', this is exactly what we wanted.

Now execute a ping on the static ip we set earlier (should be the string of numbers following 'fixed address' in the 'dhcp.conf' file.

`$ ping 192.168.1.2`

Should see a stream of returned packets indicating the device at that ip address and your current device can communicate.
