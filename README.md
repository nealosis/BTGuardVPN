# BTGuardVPN
Execute this script to establish a VPN split tunnel to the BTGuard VPN service. 

A split tunnel is a VPN connection that allows for connecting to remote VPN resources 
as well as Local Area Network (LAN) resources.

For example, you may need to connect to your employers corporate network via VPN but you also need
to be able to access a file server on your home network. ([^What is a split tunnel](https://en.wikipedia.org/wiki/Split_tunneling))


### Prerequisites

Before you can establish a BTGuard VPN tunnel for the first time you need to perform some 
setup tasks. The following steps pertain to Ubuntu Linux.

```
# Install OpenVPN (and unzip)
sudo apt-get install openvpn unzip
 
# Download BTGuard certificate (CA) and config
cd /etc/openvpn
sudo wget -O /etc/openvpn/btguard.ca.crt http://btguard.com/btguard.ca.crt
sudo wget -O /etc/openvpn/btguard.conf http://btguard.com/btguard.conf
```

Next you need to create a credentials file with your BTGuard username and password.
The username is on the first line and the password is on the second line. 

```
# Create login file (username and password on separate lines, as shown below)
sudo nano /etc/openvpn/login.txt

BTG_USERNAME
BTG_PASSWORD
```

### Installing

Download the script anywhere you like and make it executable

```
chmod +x btguard.sh
```

Update the script with your specific network information

```
sudo nano btguard.sh

# Change these variables so they match your network configuration

MAINIP=192.168.2.3          # Local IP address
GATEWAYIP=192.168.2.254     # Local Gateway (router) IP address
SUBNET=192.168.2.0/24       # Local Subnet Mask
```

You can see the available options using the -h option

```
./btguard.sh -h

btguard.sh - Establishes a split-tunnel VPN connection.
Usage: btguard.sh [-s|-t|-l|-o|-h|-v]
  -s:   Start openVPN tunnel and configure transmission
  -t:   Test run (commands are logged but not run)
  -l:   New log file (existing log is clobbered)
  -o:   Log to console & file (default is file only)
  -h:   Print help (this screen)
  -v:   Print version info

Examples:
  ./btguard.sh
  ./btguard.sh -so
```

**NOTE:** You must be an administrator to execute the script!

### Creating the split tunnel

```
sudo ./btguard -so

  Started executing script tasks.
  Configuring split tunnel.
  Stopping service transmission-daemon.
  Exec: service transmission-daemon stop 2>&1
  Adding public DNS host references.
   Exec: echo "nameserver 8.8.8.8" >> /etc/resolv.conf 2>&1
   Exec: echo "nameserver 209.222.18.218" >> /etc/resolv.conf 2>&1
  Adding local routes to ip table.
   Exec: ip rule add from 192.168.2.3 table 128 2>&1
   Exec: ip route add table 128 to 192.168.2.0/24 dev eth0 2>&1
   Exec: ip route add table 128 default via 192.168.2.254 2>&1
  Establishing OpenVPN tunnel.
   Exec: openvpn --config /etc/openvpn/btguard.conf --auth-user-pass /etc/openvpn/login.txt --script-security 2 2>&1 &
  Starting service transmission-daemon
  Exec: service transmission-daemon start 2>&1
Script tasks completed successfully.
```

## License

This project is licensed under the GPLV3 License - see the [LICENSE.md](LICENSE.md) file for details

## Resources

* [BTGuard](https://btguard.com/vpn_openvpn_linux.php) - Official manual setup steps
