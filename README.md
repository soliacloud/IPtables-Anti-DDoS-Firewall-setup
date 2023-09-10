# Protect-Your-Server-Against-DDoS-Protection
Server DDoS Protection using IPtables Anti-DDoS Rules
```
Install iptables if you haven't already
#Alternatively use packet manager of your choice
apt-get install iptables

#Accept established traffic
iptables -t mangle -A PREROUTING -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -t mangle -A PREROUTING -i lo -j ACCEPT
iptables -t mangle -A PREROUTING -i tun+ -j ACCEPT

```

Rule #1
#Block Packets From Private Subnets (Spoofing)
```
iptables -t mangle -A PREROUTING -s 224.0.0.0/3 -j DROP 
iptables -t mangle -A PREROUTING -s 169.254.0.0/16 -j DROP 
iptables -t mangle -A PREROUTING -s 172.16.0.0/12 -j DROP 
iptables -t mangle -A PREROUTING -s 192.0.2.0/24 -j DROP 
iptables -t mangle -A PREROUTING -s 192.168.0.0/16 -j DROP 
iptables -t mangle -A PREROUTING -s 10.0.0.0/8 -j DROP 
iptables -t mangle -A PREROUTING -s 0.0.0.0/8 -j DROP 
iptables -t mangle -A PREROUTING -s 240.0.0.0/5 -j DROP 
```

Rule #2
#Block Invalid Packets
```
iptables -t mangle -A PREROUTING -m conntrack --ctstate INVALID -j DROP
```


Rule #3
#Custom Packets per second check, which applies tcp-option based rules. "1000" is default you can raise or lower it.
```
iptables -t mangle -N SYN
iptables -t mangle -A PREROUTING -p tcp -m tcp -j SYN
iptables -t mangle -A SYN -j RATEEST --rateest-name synrate --rateest-interval 150ms --rateest-ewma 0.5s
iptables -t mangle -A SYN -m rateest --rateest-lt --rateest-pps 1000 --rateest synrate -j RETURN
iptables -t mangle -A SYN -p tcp -m tcp --tcp-option 5 -j DROP
iptables -t mangle -A SYN -p tcp -m tcp --tcp-option 4 -j DROP
iptables -t mangle -A SYN -p tcp -m tcp --tcp-option 3 -j DROP
iptables -t mangle -A SYN -p tcp -m tcp --tcp-option 1 -j DROP
iptables -t mangle -A SYN -j RETURN
```

Rule #4
#Disable Website traffic on a port (Helps against Layer 7 Socket floods into you service)
```
iptables -t mangle -A PREROUTING -p tcp --dport 22 -m string --algo bm --string 'HTTP' -j DROP
```

Optional Rules

Optional #1
Simple ratelimit which ratelimits over than 4 established connections by 1 ip "Replace Destinaton-Port(--dport) to you needings"
```
iptables -t mangle -A PREROUTING -p tcp -m tcp --dport 22 --tcp-flags FIN,SYN,RST,ACK SYN -m connlimit --connlimit-above 4 --connlimit-mask 32 --connlimit-saddr -j DROP
```
