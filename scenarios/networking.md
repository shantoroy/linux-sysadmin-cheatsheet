# Networking Scenarios

## Network Configuration and Troubleshooting

### Quickly check interface status
```bash
# Show all interfaces and their status
ip -br addr show

# Show specific interface details
ip addr show dev eth0

# Check interface statistics
ip -s link show dev eth0
```

### Configure a static IP address
```bash
# Temporary static IP (until reboot)
ip addr add 192.168.1.100/24 dev eth0
ip route add default via 192.168.1.1

# Using NetworkManager (permanent)
nmcli con mod "Wired connection 1" ipv4.addresses 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.dns "8.8.8.8,8.8.4.4" ipv4.method manual
nmcli con up "Wired connection 1"

# For Debian/Ubuntu (permanent via interfaces file)
cat > /etc/network/interfaces.d/eth0 << EOF
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
EOF
systemctl restart networking

# For RHEL/CentOS (permanent via ifcfg file)
cat > /etc/sysconfig/network-scripts/ifcfg-eth0 << EOF
DEVICE=eth0
BOOTPROTO=static
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
ONBOOT=yes
TYPE=Ethernet
EOF
systemctl restart network
```

### Set up network bonding (link aggregation)
```bash
# Create a bond interface (RHEL/CentOS)
cat > /etc/sysconfig/network-scripts/ifcfg-bond0 << EOF
DEVICE=bond0
TYPE=Bond
NAME=bond0
BONDING_MASTER=yes
BOOTPROTO=static
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
BONDING_OPTS="mode=802.3ad miimon=100"
ONBOOT=yes
EOF

# Configure slave interfaces
cat > /etc/sysconfig/network-scripts/ifcfg-eth0 << EOF
DEVICE=eth0
TYPE=Ethernet
MASTER=bond0
SLAVE=yes
BOOTPROTO=none
ONBOOT=yes
EOF

cat > /etc/sysconfig/network-scripts/ifcfg-eth1 << EOF
DEVICE=eth1
TYPE=Ethernet
MASTER=bond0
SLAVE=yes
BOOTPROTO=none
ONBOOT=yes
EOF

# Ensure bonding module is loaded
echo "bonding" > /etc/modules-load.d/bonding.conf

# Restart networking
systemctl restart network
```

### Configure a bridge interface
```bash
# Create a bridge (Debian/Ubuntu)
apt-get install bridge-utils
cat > /etc/network/interfaces.d/br0 << EOF
auto br0
iface br0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bridge_ports eth0
    bridge_stp on
    bridge_fd 0
EOF
systemctl restart networking

# Using NetworkManager
nmcli con add type bridge ifname br0
nmcli con mod bridge-br0 ipv4.addresses 192.168.1.100/24 ipv4.method manual ipv4.gateway 192.168.1.1
nmcli con add type ethernet slave-type bridge con-name br0-port1 ifname eth0 master br0
nmcli con up bridge-br0
```

### Network troubleshooting sequence
```bash
# Check interface status
ip addr

# Check routing table
ip route
route -n

# Check DNS resolution
nslookup google.com
dig google.com

# Check default gateway connectivity
ping -c4 $(ip route | grep default | awk '{print $3}')

# Check external connectivity
ping -c4 8.8.8.8

# Check ports in use
ss -tuln

# Check for listening services
netstat -tulnp

# Trace route to destination
traceroute google.com
mtr google.com

# Check for packet loss
ping -c 100 google.com | grep -E 'transmitted|loss'

# Check if firewall is blocking
iptables -L -n -v
firewall-cmd --list-all

# Test specific port connectivity
nc -zv host.example.com 22
telnet host.example.com 80
```

### Configure network traffic shaping
```bash
# Limit bandwidth on an interface to 1Mbit
tc qdisc add dev eth0 root tbf rate 1mbit burst 32kbit latency 400ms

# Clear traffic control settings
tc qdisc del dev eth0 root

# Prioritize SSH traffic
tc qdisc add dev eth0 root handle 1: prio
tc filter add dev eth0 protocol ip parent 1: prio 1 u32 match ip dport 22 0xffff flowid 1:1
```

### Port forwarding with iptables
```bash
# Forward external port 8080 to internal 80
iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80

# Forward port 80 to internal server
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:80
iptables -t nat -A POSTROUTING -j MASQUERADE

# Save rules (Debian/Ubuntu)
iptables-save > /etc/iptables/rules.v4

# Save rules (RHEL/CentOS)
service iptables save
```

### Set up IP masquerading (NAT)
```bash
# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# Set up NAT rules
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth1 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT

# Save the rules
iptables-save > /etc/iptables/rules.v4
```

### Capture and analyze network traffic
```bash
# Basic packet capture
tcpdump -i eth0

# Save output to a file
tcpdump -i eth0 -w capture.pcap

# Filter for specific traffic (example: HTTP)
tcpdump -i eth0 port 80

# Filter by source/destination
tcpdump -i eth0 src 192.168.1.100 and dst 192.168.1.200

# Analyze a specific protocol
tcpdump -i eth0 icmp

# Read a saved capture file
tcpdump -r capture.pcap

# Display verbose output and packet payload
tcpdump -i eth0 -vvv -X port 80
```

## DNS Configuration

### Configure local DNS resolution
```bash
# Add hosts to /etc/hosts
echo "192.168.1.10 server1.example.com server1" >> /etc/hosts

# Flush DNS cache (systemd-resolved)
systemd-resolve --flush-caches

# Flush DNS cache (nscd)
service nscd restart
```

### Set up a local DNS server (using dnsmasq)
```bash
# Install dnsmasq
apt-get install dnsmasq

# Configure dnsmasq
cat > /etc/dnsmasq.conf << EOF
# Listen on this interface
interface=eth0
# Don't forward short names
domain-needed
# Don't forward addresses in non-routed address spaces
bogus-priv
# Use Google DNS
server=8.8.8.8
server=8.8.4.4
# Local domain
local=/example.local/
domain=example.local
# DHCP range
dhcp-range=192.168.1.50,192.168.1.150,12h
# Add some fixed hosts
dhcp-host=00:11:22:33:44:55,server1,192.168.1.10
# Set gateway
dhcp-option=3,192.168.1.1
# Set DNS server
dhcp-option=6,192.168.1.1
EOF

# Start dnsmasq
systemctl enable dnsmasq
systemctl restart dnsmasq
```

### DNS lookup tools
```bash
# DNS lookup using different tools
dig example.com
host example.com
nslookup example.com

# Query specific record types
dig MX example.com
dig TXT example.com
dig NS example.com

# Query specific DNS server
dig @8.8.8.8 example.com

# Reverse DNS lookup
dig -x 8.8.8.8
```