shorewall-lite on openwrt

##### ��װ

```
root@lab:~ # echo "" >> /etc/apt/sources.list
root@lab:~ # aptitude update && aptitude install shorewall ulogd
```

##### ����

/etc/default/shorewall

```
startup=1
...
```

shorewall.conf

```bash
# �޸����²�����
LOGFILE=/var/log/ulog/syslogemu.log
LOGFORMAT=""
# ������info�滻��$LOG

...
```

/etc/shorewall/params

```
INT_IF=eth0
VPN_IF=tun0
LOG=ULOG
```

/etc/shorewall/zones

```
fw      firewall
net     ipv4
vpn     ipv4
```

/etc/shorewall/interfaces

```
net     $INT_IF            dhcp,tcpflags,logmartians,nosmurfs,sourceroute=0
vpn     $VPN_IF            dhcp,tcpflags,logmartians,nosmurfs,sourceroute=0
```

/etc/shorewall/policy

```
$FW             net             ACCEPT
$FW             vpn             ACCEPT
vpn             $FW             ACCEPT
net             all             DROP            $LOG
all             all             REJECT          $LOG
```

/etc/shorewall/rules

```
###############
# net2fw

# allow ssh,http,https,655,80,2003 from net to $FW
ACCEPT          net             $FW             tcp             ssh,http,https,2003 
ACCEPT          net             $FW             udp,tcp         655

###############
# vpn2net

# allow ssh,http from USER1 to LAB 
ACCEPT     vpn:10.8.0.32/27     net:192.168.33.0/24,192.168.66.0/24  tcp     ssh,http

# allow all from CORP to LAB
ACCEPT          vpn:10.8.0.64/27        net:192.168.33.0/24     all     
ACCEPT          vpn:10.8.0.64/27        net:192.168.55.0/24     all     
ACCEPT          vpn:10.8.0.64/27        net:192.168.66.0/24     all     
ACCEPT          vpn:10.8.0.64/27        net:192.168.88.0/24     all     
ACCEPT          vpn:10.8.0.64/27        net:192.168.100.0/24    all     
ACCEPT          vpn:10.8.0.64/27        net:172.16.33.0/24      all     

###############
# net2vpn

ACCEPT  net:192.168.33.0/24     vpn     all
ACCEPT  net:192.168.66.0/24     vpn     all
ACCEPT  net:10.5.1.0/24         vpn     all
ACCEPT  net:172.16.33.0/24      vpn     all

###############
# all2all

Ping(ACCEPT)   all             all
```

/etc/shorewall/nat

```
10.8.0.2       tun0            192.168.33.231     No               No
10.8.0.3       tun0            192.168.33.232     No               No
10.8.0.4       tun0            192.168.33.233     No               No
10.8.0.5       tun0            192.168.33.234     No               No
10.8.0.6       tun0            192.168.66.21      No               No
10.8.0.7       tun0            192.168.66.22      No               No
10.8.0.8       tun0            192.168.66.23      No               No
10.8.0.9       tun0            192.168.55.120     No               No
10.8.0.10      tun0            192.168.88.120     No               No
```

```
root@lab:~ # shorewall check
root@lab:~ # shorewall restart
root@lab:~ # /etc/init.d/ulogd start
```

> **NOTE** ����Լ��������棬ϣ���ҵ����ѻ�������̫��

���ˣ�a mesh vpn network����ˡ�

####��shorewall-lite on OpenWRT

shorewall������perl������OpenWRT��˵̫�Ӵ��ˡ����⣬�����ж������ǽ������Ҫһ�׻��ƽ���ͳһ�������ǵ�����shorewall-lite��

���죬���Ǿ���OpenWRT������һ��shorewall-lite��ħ����
ͼƬ

admin(administrative system)��װ��shorewall��firewall�������ļ�����admin����ɣ����ͨ��shorewall compile�����ɽű�������ͨ��scp���ýű�������firewall��/etc/shorewall-lite/stateĿ¼��Ȼ��ʹ��sshԶ��ִ��firewall��shorewall-lite�����ű�ת����iptables rules��

�����shorewall-lite������ԭ��

���ԣ����Ȱ�װadmin��shorewall

root@shorewall-centre-d6:/ # apt-get update && apt-get install shorewall

    * Ϊÿ��firewall����һ��exportĿ¼

root@shorewall-centre-d6:/ # make -p export/rb450g && cd export/rb450g

    * ׼��firewall�����ļ�

����debianϵ��������tarball����ѹ��/usr/share/shorewall/configfiles�е��ļ�������exportĿ¼��

    * ����firewall�����ļ�

���������������ļ����ǿ��ļ�����Ҫ���е������á�

/etc/shorewall/export/rb450g/params

```
WAN_IF=eth0
LAN_IF=br-lan
OA_IF=br-oa
VPN_IF=tun0
LOG=ULOG
```

/etc/shorewall/export/rb450g/zones

```
fw      firewall
oa      ipv4
lan     ipv4
wan     ipv4
vpn     ipv4
```

/etc/shorewall/export/rb450g/interfaces

```
wan             $WAN_IF                 dhcp,tcpflags,logmartians,nosmurfs,sourceroute=0
lan             $LAN_IF                 tcpflags,logmartians,nosmurfs,sourceroute=0
vpn             $VPN_IF                 tcpflags,logmartians,nosmurfs,sourceroute=0
oa              $OA_IF                  tcpflags,logmartians,nosmurfs,sourceroute=0
```

/etc/shorewall/export/rb450g/policy

```
$FW     all     ACCEPT
lan     all     ACCEPT
wan     all     DROP            $LOG    10/sec:40
all     all     REJECT
```

/etc/shorewall/export/rb450g/rules

```
SECTION NEW
Invalid(DROP)   wan             all

###############
# vpn2fw

Ping(ACCEPT)    vpn             $FW
SSH(ACCEPT)     vpn             $FW
HTTP(ACCEPT)    vpn             $FW

###############
# wan2fw

ACCEPT          wan             $FW     tcp     655
ACCEPT          wan             $FW     udp     655
SSH(ACCEPT)     wan             $FW
```

/etc/shorewall/export/rb450g/masq

```
$OA_IF          192.168.44.0/24 10.199.27.17
$VPN_IF         192.168.44.0/24 10.8.0.65
$WAN_IF         192.168.44.0/24 192.168.7.21
```

OpenWRT��׼������

����`state`

root@RB450G:/ # mkdir /etc/shorewall-lite/state

����firewall��

```bash
root@RB450G:/ # /etc/init.d/firewall disable
root@RB450G:/ # /etc/init.d/firewall stop
```

����shorewall-lite

```bash
root@RB450G:/# /etc/init.d/shorewall-lite enable
```

    * ����firewall�ű�

��Ȼ����`shorewall compile`������firewall�ű���Ȼ��Thomas M. Eastep�Լ�д�˸�Makefile��Ȼ��ͨ��`make`��`make install`������linux����Ա���������ָ��������Ͳ���

root@shorewall-centre-d6:/etc/shorewall/export/rb450g# wget http://www1.shorewall.net/pub/shorewall/contrib/Shorewall-lite/

Ȼ�����Makefile�е�HOST��������IP��ַ���ɡ���������������Ҫȷ�����Խ�����

```bash
root@shorewall-centre-d6:/etc/shorewall/export/rb450g# make
shorewall compile -e . firewall
Compiling...
Processing /etc/shorewall/export/rb450g/params ...
Processing /etc/shorewall/export/rb450g/shorewall.conf...
   WARNING: Your capabilities file is out of date -- it does not contain all of the capabilities defined by Shorewall version 4.5.5.3
Compiling /etc/shorewall/export/rb450g/zones...
Compiling /etc/shorewall/export/rb450g/interfaces...
Determining Hosts in Zones...
Locating Action Files...
Compiling /usr/share/shorewall/action.Drop for chain Drop...
Compiling /usr/share/shorewall/action.Broadcast for chain Broadcast...
Compiling /usr/share/shorewall/action.Invalid for chain Invalid...
Compiling /usr/share/shorewall/action.NotSyn for chain NotSyn...
Compiling /usr/share/shorewall/action.Reject for chain Reject...
Compiling /etc/shorewall/export/rb450g/policy...
Compiling /etc/shorewall/export/rb450g/notrack...
Running /etc/shorewall/export/rb450g/initdone...
Adding Anti-smurf Rules
Adding rules for DHCP
Compiling TCP Flags filtering...
Compiling Kernel Route Filtering...
Compiling Martian Logging...
Compiling Accept Source Routing...
Compiling /etc/shorewall/export/rb450g/tcrules...
Compiling /etc/shorewall/export/rb450g/masq...
Compiling MAC Filtration -- Phase 1...
Compiling /etc/shorewall/export/rb450g/rules...
Compiling /usr/share/shorewall/action.Invalid for chain %Invalid...
Compiling MAC Filtration -- Phase 2...
Applying Policies...
Generating Rule Matrix...
Creating iptables-restore input...
Shorewall configuration compiled to /etc/shorewall/export/rb450g/firewall
�����ڵ�ǰĿ¼����firewall�ű���Ȼ�����make install������firewall��
root@shorewall-centre-d6:/etc/shorewall/export/rb450g# make install
scp firewall firewall.conf root@192.168.44.1:/etc/shorewall-lite/state
root@192.168.44.1's password:
firewall                                                                                                                             100%   79KB  79.3KB/s   00:00   
firewall.conf                                                                                                                        100%  862     0.8KB/s   00:00   
ssh root@192.168.44.1 "/sbin/shorewall-lite restart"
root@192.168.44.1's password:Restarting Shorewall Lite....
Initializing...
Processing init user exit ...
Processing tcclear user exit ...
Setting up Route Filtering...
Setting up Martian Logging...
Setting up Accept Source Routing...
Setting up Proxy ARP...
Setting up Traffic Control...
Preparing iptables-restore input...
Running /usr/sbin/iptables-restore...
IPv4 Forwarding EnabledProcessing start user exit ...
Processing started user exit ...
done.
touch: /var/lock/subsys/shorewall: No such file or directory
```

ִ��`make install`ʱ��admin�Ὣfirewall��firewall.confͨ��scp������firewall(rb450g)��/etc/shorewall-lite/stateĿ¼�£���firewall(rb450g)��ִ��/etc/init.d/shorewall-lite stop|start|restart�����Ŀ¼�µ�firewall�ű��򽻵���

������make installִ�гɹ���/etc/shorewall-lite/state���ļ��б�

```bash
root@RB450G:/etc/shorewall-lite/state# ls -alh
drwxr-xr-x    1 root     root        2.0K Sep  3 13:49 .
drwxr-xr-x    1 root     root        2.0K Sep  3 11:05 ..
-rw-------    1 root     root           0 Sep  3 13:49 .dynamic
-rw-------    1 root     root        9.8K Sep  3 13:49 .iptables-restor
-rw-------    1 root     root        3.4K Sep  3 13:49 .modules
-rw-------    1 root     root          12 Sep  3 13:49 .modulesdir
-rw-r--r--    1 root     root        1.0K Sep  3 11:09 capabilities
-rwx------    1 root     root       79.3K Sep  3 13:43 firewall
-rw-------    1 root     root         862 Sep  3 13:43 firewall.conf
-rw-------    1 root     root         162 Sep  3 13:49 marks
-rw-------    1 root     root           0 Sep  3 13:49 nat
-rw-------    1 root     root         740 Sep  3 13:49 policies
-rw-------    1 root     root           0 Sep  3 13:49 proxyarp
-rw-------    1 root     root          29 Sep  3 13:49 restarted
-rw-------    1 root     root          74 Sep  3 13:49 state
-rw-------    1 root     root         110 Sep  3 13:49 zones
```

### tricks

**����iptables -F**

���������֮ǰ������ȷ��Chain INPUT��Ĭ��poliy�������ǣ�`Chain INPUT (policy DROP 0 packets, 0 bytes)`������Ҫ��`iptables -P INPUT ACCEPT`��Ȼ��`iptables -F`���������Լ�����ϵͳ֮�⡣

**�漰������������**

�ڲ��ԵĹ����У�����tinc��û����������shorewall make installʧ�ܡ�
��tinc���úú���make install�ͳɹ��ˡ�

**log**

ѡ��shorewall/shorewall-lite��һ����Ҫԭ����shorewall log�ɸ���`Դzone+Ŀ��zones`���飬ʹ�ù���Ա����Ѹ�ٶ�λ����Ĺ��򡣾ٸ����ӣ�

��lab��telnet corp��tinc vpn��ַʧ�ܣ����ǲ鿴˫����log��־��lab����־��������corp���ҵ���˿����

```bash
# tail -f /var/log/ulogd.syslogemu | grep 10.8.0.65
Sep  3 13:27:14 corp Shorewall:vpn2fw:REJECT: IN=tun0 OUT= MAC= SRC=10.8.0.1 DST=10.8.0.65 LEN=60 TOS=10 PREC=0x00 TTL=64 ID=19084 DF PROTO=TCP SPT=41748 DPT=23 SEQ=1825983957 ACK=0 WINDOW=5840 SYN URGP=0 
```

��log������10.8.0.1 telnet 10.8.0.65������vpn2fw��policy��rules�����ǣ�����ֻҪ��`/etc/shorewall/export/rb450g/rules`�е�vpn2fw�������һ��`TELNET(ACCEPT)    vpn    $FW`���ɡ�

������shorewall��log����

debian

```bash
# sudo apt-get update
# sudo apt-get install iptables-mod-ulog kmod-ipt-ulog ulogd ulogd-mod-extra
# /etc/init.d/ulogd start
```

OpenWRT

```bash
# opkg update
# opkg install iptables-mod-ulog kmod-ipt-ulog ulogd ulogd-mod-extra
# /etc/init.d/ulogd start
```

�Ŵ�ʱ�ٿ���ulogd Ĭ������£��Ὣlogд��/var/log/ulogd.syslogemu��

shorewall�������iptables�Ĺ���Ȼ����pf��Ȼ�����ѷһ���Ҫ��shorewall�������ļ�̫���ˡ�pfֻ��Ҫһ�������ļ���Ҫ�򵥡��ɰ��öࡣ����Ϊ���������������ŵķ���ǽ����labΪ����

/etc/pf.conf

```
wan_if = eth0
vpn_if = tun0
lab_vpn_net = "10.8.0.0/24"
User1_vpn_net = "10.8.0.32/27"
corp_vpn_net = "10.8.0.64/27"

table <lab_net> { "192.168.33.0/24", "192.168.55.0/24"
                  "192.168.66.0/24", "192.168.88.0/24"
                  "192.168.100.0/24", "172.16.33.0/24"
}

###############
# 1:1 nat

pass on tun0 from 192.168.33.231 to any binat-to 10.8.0.2
pass on tun0 from 192.168.33.232 to any binat-to 10.8.0.3
pass on tun0 from 192.168.33.233 to any binat-to 10.8.0.4
pass on tun0 from 192.168.33.234 to any binat-to 10.8.0.5
pass on tun0 from 192.168.66.21 to any binat-to 10.8.0.6
pass on tun0 from 192.168.66.22 to any binat-to 10.8.0.7
pass on tun0 from 192.168.66.23 to any binat-to 10.8.0.8
pass on tun0 from 192.168.55.120 to any binat-to 10.8.0.9
pass on tun0 from 192.168.88.120 to any binat-to 10.8.0.10

###############
# masq

# lab�в���Ҫ�õ�masq�����������ñ�ע�͵�
# match out on $ext_if from !($vpn_if) to any nat-to ($vpn_if)

block all

###############
# rules

# wan2fw
permit proto { udp, tcp } from any to $wan_if port 655

# vpn2net
permit proto tcp from $User1_vpn_net to <lab_net> port { 22,80,443 }
permit from $corp_vpn_net to <lab_net>

# all2all
permit proto icmp all
```

pf.conf֧��table��list������Ƕ�ף���������rules������������ά����

��ɷ���ǽ���ú󣬻�Ҫ����tinc���á�


## openbsd&pfsense

�迪��openvpn����tinc vpn��Ȩ��
pass log inet proto { tcp, udp } from <vpn_net> to <tinc_net>

�ڡ�nat���ġ�openvpn��tab�У��½�һ��Ŀ���ַ��Ϊtinc_net��������ʲ���

[^footnote1]: **��ע1**


