#Linux
```
find . -type d -exec chmod 755 {} +
find . -type f -exec chmod 644 {} +
```
##Ansible
```
ansible-playbook --ask-su-pass -i stage site.yml
```

source:
* http://www.ansible.com
* http://docs.ansible.com/ansible/playbooks_best_practices.html


##Gateway
eth0: local network  
eth1: external network

`nano /etc/network/if-up.d/00-firewall`

```
#!/bin/sh

PATH=/usr/sbin:/sbin:/bin:/usr/bin

#
# delete all existing rules.
#
iptables -F
iptables -t nat -F
iptables -t mangle -F
iptables -X

# Always accept loopback traffic
iptables -A INPUT -i lo -j ACCEPT

# Allow established connections, and those not coming from the outside
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -m state --state NEW -i ! eth1 -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow outgoing connections from the LAN side.
iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT

# Masquerade.
iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE

# Don't forward from the outside to the inside.
iptables -A FORWARD -i eth1 -o eth1 -j REJECT

# Enable routing.
echo 1 > /proc/sys/net/ipv4/ip_forward
```

`chmod +x /etc/network/if-up.d/00-firewall`

source: 
* https://www.debian-administration.org/article/23/Setting_up_a_simple_Debian_gateway

##Mysql Cluster
###Management
```
sudo ndb_mgmd --initial --configdir=/var/lib/mysql-cluster -f /var/lib/mysql-cluster/config.ini
ndb_mgm -e shutdown
```
###Data
```
sudo ndbmtd
```
###Sql
```
sudo /usr/local/mysql/bin/mysql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;

sudo /usr/local/mysql/bin/mysqld --user=mysql &
```
