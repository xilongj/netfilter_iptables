## 1. Netfilter Iptables

### 1.1. Netfilter and Iptables Introduction

```css
- Netfilter is a software firewall, a packet filtering framework inside the Linux Kernel;
- It enables packet filtering, NAT, PAT, Port Forwarding and packet managling;
- Netfilter framework is controlled by the iptables commands;
- Iptables is a tool that belongs to the user-space used to configure netfilter;
- Netfilter and Iptables are often combined into a single expression netfilter/iptables;
- Every Linux distribution uses netfiler/iptables, there is nothing extra that should be installed;
- Only root user can use or configure netfilter framework;
- Every packet is inspected by firewall rules. Firewall rules determine what traffic your firewall allows and what is blocked;
- The Iptables firewall uses tables to organize its rules;
- Within each iptables table, rules are further organized within sepatare CHAINS. 
  * Rules are placed within a specific chain of a specific   table.
- Within a chain, a packet starts at the top of the chain and is matched rule by rule.
- When a match is found the target is executed.
- A target is the action that is triggered when a packet meets the matching criteria of a rule. If the target is terminating no other       rule will evaluate the packet.
```

 #### 1.2 Netfilter CHAINS

```css
INPUT       - used for filtering incoming packets. Our host is the packet destination;
OUTPUT      - used for filtering outgoing packets. Our host is the source of packet;
FORWARD     - used for filtering routed packets.   Our host is router;
PREROUTING  - used for DNAT / Port Forwarding;
POSTROUTING - used for SNAT ( MASQUERATE );
```

```bash
# INPUT
C:\Users\xilongj>ping 192.168.2.10
Pinging 192.168.2.10 with 32 bytes of data:
Reply from 192.168.2.10: bytes=32 time<1ms TTL=64

root@cnos2-10:-# iptables -t filter -A INPUT -p icmp -j DROP

C:\Users\xilongj>ping 192.168.2.10
Pinging 192.168.2.10 with 32 bytes of data:
Request timed out.
```

```bash
# INPUT - used for filtering incoming packets. Our host is the packet destination;
root@cnos2-10:-# iptables -t filter -A INPUT -p icmp -j DROP
Note: We should always write table names in lowercase letters and chain names in uppercase letters. Targets like drop in our example are always written in uppercase letters.
```

```bash
# OUTPUT
root@cnos2-10:~# iptables -F
root@cnos2-10:~# ping -c 2 netfilter.org
PING netfilter.org (92.243.18.11) 56(84) bytes of data.
64 bytes from xvm-18-11.dc0.ghst.net (92.243.18.11): icmp_seq=1 ttl=128 time=233 ms
64 bytes from xvm-18-11.dc0.ghst.net (92.243.18.11): icmp_seq=2 ttl=128 time=233 ms

--- netfilter.org ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 233.111/233.205/233.299/0.094 ms

root@cnos2-10:~# iptables -t filter -A OUTPUT -d netfilter.org -j DROP
root@cnos2-10:~# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DROP       all  --  anywhere             xvm-18-11.dc0.ghst.net

root@cnos2-10:~# ping -c 2 netfilter.org
PING netfilter.org (92.243.18.11) 56(84) bytes of data.
ping: sendmsg: Operation not permitted
ping: sendmsg: Operation not permitted
^C
--- netfilter.org ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1027ms
```

#### 1.3 Netfilter TABLES

```css
#1. filter
filter is the default table for iptables;
iptables filter table has the following built-in chains: INPUT, OUTPUT and FORWARD
#2. nat
nat table is specialized for SNAT and DNAT (Port Forwarding)
iptables NAT table has the following built-in chains: PREROUTING, POSTROUTING and OUTPUT (for local generated packets)
#3. mangle
iptables mangle table is specialized for packet alteration;
mangle table has the following built-in chains: PREROUTING, INPUT, FORWARD, OUTPUT, POSTROUTING
#4. raw
The raw table is only used to set a mark on packets that should not be handled by the connection tracking system. This is done by using the NOTRACK target on the packet. 
Raw table has the following built-in chains: PREROUTING and OUTPUT
```

```bash
#1. filter
filter is the default table for iptables;
iptables filter table has the following built-in chains: INPUT, OUTPUT and FORWARD
root@cnos2-10:~# iptables -A INPUT -p tcp --dport 22 -s 192.168.2.20 -j ACCEPT
root@cnos2-10:~# iptables -A INPUT -p tcp --dport 22 -j DROP
root@cnos2-10:~# client_loop: send disconnect: Connection reset

root@cnos2-10:~# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  192.168.2.20         anywhere             tcp dpt:ssh
DROP       tcp  --  anywhere             anywhere             tcp dpt:ssh

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Note: 
- Now, I only can connect to 192.168.2.20 vm from Windows Host, then jump connect to 192.168.2.10. 
- Other trying connections will be dropped.
```

#### 1.4 Chain Traversal in a Nutshell

```css
Incoming traffic is filtered on the INPUT CHAIN of the filter table;
Outgoing traffic is filtered on the OUTPUT CHAIN of the filter table;
Routed traffic is filtered on the FORWARD CHAIN of the filter table;
SNAT/MASQUERADE is performed on the POSTROUTING CHAIN of the nat table;
DNAT/Port Forwarding is performed on the PREROUTING CHAIN of the nat table;
To modify values from the packet's headers add rules to the mangle table;
To skip the connection tracking add rules with NOTRACK target to the raw table;
```



