
Check vm.swappiness on all your nodes
```
[root@ip-172-31-4-207 ~]# cat /proc/sys/vm/swappiness
30
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]# sysctl vm.swappiness=1
vm.swappiness = 1
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]# cat /proc/sys/vm/swappiness
1
[root@ip-172-31-4-207 ~]#

** Setup permanent when machine startup
[root@ip-172-31-4-207 ~]# tail -2 /etc/sysctl.conf
####################
vm.swappiness=1
[root@ip-172-31-4-207 ~]#
```

Show mount attributed of all volumes
```
[root@ip-172-31-4-207 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda2      100G  984M  100G   1% /
devtmpfs        3.9G     0  3.9G   0% /dev
tmpfs           3.7G     0  3.7G   0% /dev/shm
tmpfs           3.7G   17M  3.7G   1% /run
tmpfs           3.7G     0  3.7G   0% /sys/fs/cgroup
tmpfs           757M     0  757M   0% /run/user/1000
tmpfs           757M     0  757M   0% /run/user/0
[root@ip-172-31-4-207 ~]#
```
Show the reserve space of any non-root, ext-based volumes
```
[root@ip-172-31-4-207 ~]# egrep "^/dev/*" /proc/mounts
/dev/xvda2 / xfs rw,seclabel,relatime,attr2,inode64,noquota 0 0
[root@ip-172-31-4-207 ~]#

** I'm configure only 1 volum to shared with all (root and non-root)
```
Disable transparent hugepages
```
[root@ip-172-31-4-207 ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
[always] madvise never
[root@ip-172-31-4-207 ~]# cat /sys/kernel/mm/transparent_hugepage/defrag
[always] madvise never
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]# echo never > /sys/kernel/mm/transparent_hugepage/enabled
[root@ip-172-31-4-207 ~]# echo never > /sys/kernel/mm/transparent_hugepage/defrag
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]# cat /sys/kernel/mm/transparent_hugepage/enabled
always madvise [never]
[root@ip-172-31-4-207 ~]# cat /sys/kernel/mm/transparent_hugepage/defrag
always madvise [never]
[root@ip-172-31-4-207 ~]#

** Setup permanent when machine startup
[root@ip-172-31-4-207 ~]# tail -2 /etc/rc.local
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
[root@ip-172-31-4-207 ~]#
```
List the network interface configuration
```
[root@ip-172-31-4-207 ~]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 172.31.4.207  netmask 255.255.240.0  broadcast 172.31.15.255
        inet6 fe80::12:d4ff:fecb:401b  prefixlen 64  scopeid 0x20<link>
        ether 02:12:d4:cb:40:1b  txqueuelen 1000  (Ethernet)
        RX packets 3648  bytes 320205 (312.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3403  bytes 534153 (521.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 68  bytes 6420 (6.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 68  bytes 6420 (6.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth* | egrep "^DEVICE|BOOTPROTO|ONBOOT|NETMASK|NETWORK|IPADDR|PEERDNS"
DEVICE="eth0"
BOOTPROTO="dhcp"
ONBOOT="yes"
PEERDNS="yes"
[root@ip-172-31-4-207 ~]#
```
List forward and reverse host lookups using getent or nslookup
```
[root@ip-172-31-4-207 ~]# getent hosts 172.31.4.207
172.31.4.207    ip-172-31-4-207.ap-southeast-1.compute.internal
[root@ip-172-31-4-207 ~]#
```
Show nscd service is running
```
** Check RHEL Repo and YUM service available
[root@ip-172-31-4-207 ~]# head /etc/yum.repos.d/redhat-rhui.repo
# The amazon ec2 region is now dynamically set via yum.  REGION can be replaced by the amazon ec2 region for debugging
[rhui-REGION-rhel-server-releases]
name=Red Hat Enterprise Linux Server 7 (RPMs)
mirrorlist=https://rhui2-cds01.REGION.aws.ce.redhat.com/pulp/mirror/content/dist/rhel/rhui/server/7/$releasever/$basearch/os
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
sslverify=1
sslclientkey=/etc/pki/rhui/content-rhel7.key
sslclientcert=/etc/pki/rhui/product/content-rhel7.crt
[root@ip-172-31-4-207 ~]#

[root@ip-172-31-4-207 ~]# yum list nscd
Loaded plugins: amazon-id, rhui-lb, search-disabled-repos
Available Packages
nscd.x86_64                                    2.17-157.el7_3.1                                    rhui-REGION-rhel-server-releases
[root@ip-172-31-4-207 ~]#
```
```
** Install nscd package
[root@ip-172-31-4-207 ~]# yum install nscd
Loaded plugins: amazon-id, rhui-lb, search-disabled-repos
Resolving Dependencies
--> Running transaction check
---> Package nscd.x86_64 0:2.17-157.el7_3.1 will be installed
--> Processing Dependency: glibc = 2.17-157.el7_3.1 for package: nscd-2.17-157.el7_3.1.x86_64
--> Running transaction check
---> Package glibc.x86_64 0:2.17-157.el7 will be updated
--> Processing Dependency: glibc = 2.17-157.el7 for package: glibc-common-2.17-157.el7.x86_64
---> Package glibc.x86_64 0:2.17-157.el7_3.1 will be an update
--> Running transaction check
---> Package glibc-common.x86_64 0:2.17-157.el7 will be updated
---> Package glibc-common.x86_64 0:2.17-157.el7_3.1 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

===================================================================================================================================
 Package                   Arch                Version                         Repository                                     Size
===================================================================================================================================
Installing:
 nscd                      x86_64              2.17-157.el7_3.1                rhui-REGION-rhel-server-releases              267 k
Updating for dependencies:
 glibc                     x86_64              2.17-157.el7_3.1                rhui-REGION-rhel-server-releases              3.6 M
 glibc-common              x86_64              2.17-157.el7_3.1                rhui-REGION-rhel-server-releases               11 M

Transaction Summary
===================================================================================================================================
Install  1 Package
Upgrade             ( 2 Dependent packages)

Total download size: 15 M
Is this ok [y/d/N]: y
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/3): glibc-2.17-157.el7_3.1.x86_64.rpm                                                                    | 3.6 MB  00:00:00
(2/3): nscd-2.17-157.el7_3.1.x86_64.rpm                                                                     | 267 kB  00:00:00
(3/3): glibc-common-2.17-157.el7_3.1.x86_64.rpm                                                             |  11 MB  00:00:00
-----------------------------------------------------------------------------------------------------------------------------------
Total                                                                                               15 MB/s |  15 MB  00:00:01
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : glibc-common-2.17-157.el7_3.1.x86_64                                                                            1/5
  Updating   : glibc-2.17-157.el7_3.1.x86_64                                                                                   2/5
warning: /etc/nsswitch.conf created as /etc/nsswitch.conf.rpmnew
  Installing : nscd-2.17-157.el7_3.1.x86_64                                                                                    3/5
  Cleanup    : glibc-common-2.17-157.el7.x86_64                                                                                4/5
  Cleanup    : glibc-2.17-157.el7.x86_64                                                                                       5/5
  Verifying  : glibc-2.17-157.el7_3.1.x86_64                                                                                   1/5
  Verifying  : nscd-2.17-157.el7_3.1.x86_64                                                                                    2/5
  Verifying  : glibc-common-2.17-157.el7_3.1.x86_64                                                                            3/5
  Verifying  : glibc-common-2.17-157.el7.x86_64                                                                                4/5
  Verifying  : glibc-2.17-157.el7.x86_64                                                                                       5/5

Installed:
  nscd.x86_64 0:2.17-157.el7_3.1

Dependency Updated:
  glibc.x86_64 0:2.17-157.el7_3.1                              glibc-common.x86_64 0:2.17-157.el7_3.1

Complete!
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]# service nscd status
Redirecting to /bin/systemctl status  nscd.service
● nscd.service - Name Service Cache Daemon
   Loaded: loaded (/usr/lib/systemd/system/nscd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
[root@ip-172-31-4-207 ~]#
```
```
** Start nscd service
[root@ip-172-31-4-207 ~]# service nscd start
Redirecting to /bin/systemctl start  nscd.service
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]# service nscd status
Redirecting to /bin/systemctl status  nscd.service
● nscd.service - Name Service Cache Daemon
   Loaded: loaded (/usr/lib/systemd/system/nscd.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2017-02-13 04:01:49 EST; 9s ago
  Process: 3259 ExecStart=/usr/sbin/nscd $NSCD_OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 3260 (nscd)
   CGroup: /system.slice/nscd.service
           └─3260 /usr/sbin/nscd

Feb 13 04:01:49 ip-172-31-4-207.ap-southeast-1.compute.internal nscd[3260]: 3260 monitoring file `/etc/hosts` (4)
Feb 13 04:01:49 ip-172-31-4-207.ap-southeast-1.compute.internal nscd[3260]: 3260 monitoring directory `/etc` (2)
Feb 13 04:01:49 ip-172-31-4-207.ap-southeast-1.compute.internal nscd[3260]: 3260 monitoring file `/etc/resolv.conf` (5)
Feb 13 04:01:49 ip-172-31-4-207.ap-southeast-1.compute.internal nscd[3260]: 3260 monitoring directory `/etc` (2)
Feb 13 04:01:49 ip-172-31-4-207.ap-southeast-1.compute.internal nscd[3260]: 3260 monitoring file `/etc/services` (6)
Feb 13 04:01:49 ip-172-31-4-207.ap-southeast-1.compute.internal nscd[3260]: 3260 monitoring directory `/etc` (2)
Feb 13 04:01:49 ip-172-31-4-207.ap-southeast-1.compute.internal nscd[3260]: 3260 disabled inotify-based monitoring for file `...ory
Feb 13 04:01:49 ip-172-31-4-207.ap-southeast-1.compute.internal nscd[3260]: 3260 stat failed for file `/etc/netgroup'; will t...ory
Feb 13 04:01:49 ip-172-31-4-207.ap-southeast-1.compute.internal nscd[3260]: 3260 Access Vector Cache (AVC) started
Feb 13 04:01:49 ip-172-31-4-207.ap-southeast-1.compute.internal systemd[1]: Started Name Service Cache Daemon.
Hint: Some lines were ellipsized, use -l to show in full.
[root@ip-172-31-4-207 ~]#
```
Show the ntpd service is running
```
** Install ntpd package
[root@ip-172-31-4-207 ~]# yum list ntp
Loaded plugins: amazon-id, rhui-lb, search-disabled-repos
Available Packages
ntp.x86_64                                   4.2.6p5-25.el7_3.1                                    rhui-REGION-rhel-server-releases
[root@ip-172-31-4-207 ~]#

[root@ip-172-31-4-207 ~]# yum install ntp
Loaded plugins: amazon-id, rhui-lb, search-disabled-repos
Resolving Dependencies
--> Running transaction check
---> Package ntp.x86_64 0:4.2.6p5-25.el7_3.1 will be installed
--> Processing Dependency: ntpdate = 4.2.6p5-25.el7_3.1 for package: ntp-4.2.6p5-25.el7_3.1.x86_64
--> Processing Dependency: libopts.so.25()(64bit) for package: ntp-4.2.6p5-25.el7_3.1.x86_64
--> Running transaction check
---> Package autogen-libopts.x86_64 0:5.18-5.el7 will be installed
---> Package ntpdate.x86_64 0:4.2.6p5-25.el7_3.1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===================================================================================================================================
 Package                     Arch               Version                         Repository                                    Size
===================================================================================================================================
Installing:
 ntp                         x86_64             4.2.6p5-25.el7_3.1              rhui-REGION-rhel-server-releases             547 k
Installing for dependencies:
 autogen-libopts             x86_64             5.18-5.el7                      rhui-REGION-rhel-server-releases              66 k
 ntpdate                     x86_64             4.2.6p5-25.el7_3.1              rhui-REGION-rhel-server-releases              85 k

Transaction Summary
===================================================================================================================================
Install  1 Package (+2 Dependent packages)

Total download size: 699 k
Installed size: 1.6 M
Is this ok [y/d/N]: y
Downloading packages:
(1/3): autogen-libopts-5.18-5.el7.x86_64.rpm                                                                |  66 kB  00:00:00
(2/3): ntp-4.2.6p5-25.el7_3.1.x86_64.rpm                                                                    | 547 kB  00:00:00
(3/3): ntpdate-4.2.6p5-25.el7_3.1.x86_64.rpm                                                                |  85 kB  00:00:00
-----------------------------------------------------------------------------------------------------------------------------------
Total                                                                                              793 kB/s | 699 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : autogen-libopts-5.18-5.el7.x86_64                                                                               1/3
  Installing : ntpdate-4.2.6p5-25.el7_3.1.x86_64                                                                               2/3
  Installing : ntp-4.2.6p5-25.el7_3.1.x86_64                                                                                   3/3
  Verifying  : ntpdate-4.2.6p5-25.el7_3.1.x86_64                                                                               1/3
  Verifying  : ntp-4.2.6p5-25.el7_3.1.x86_64                                                                                   2/3
  Verifying  : autogen-libopts-5.18-5.el7.x86_64                                                                               3/3

Installed:
  ntp.x86_64 0:4.2.6p5-25.el7_3.1

Dependency Installed:
  autogen-libopts.x86_64 0:5.18-5.el7                              ntpdate.x86_64 0:4.2.6p5-25.el7_3.1

Complete!
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]# service ntpd status
Redirecting to /bin/systemctl status  ntpd.service
● ntpd.service - Network Time Service
   Loaded: loaded (/usr/lib/systemd/system/ntpd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
[root@ip-172-31-4-207 ~]#
```
```
** Start ntpd service
[root@ip-172-31-4-207 ~]# service ntpd start
Redirecting to /bin/systemctl start  ntpd.service
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]# service ntpd status
Redirecting to /bin/systemctl status  ntpd.service
● ntpd.service - Network Time Service
   Loaded: loaded (/usr/lib/systemd/system/ntpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2017-02-13 04:08:40 EST; 2s ago
  Process: 3409 ExecStart=/usr/sbin/ntpd -u ntp:ntp $OPTIONS (code=exited, status=0/SUCCESS)
 Main PID: 3410 (ntpd)
   CGroup: /system.slice/ntpd.service
           └─3410 /usr/sbin/ntpd -u ntp:ntp -g

Feb 13 04:08:40 ip-172-31-4-207.ap-southeast-1.compute.internal ntpd[3410]: Listen and drop on 1 v6wildcard :: UDP 123
Feb 13 04:08:40 ip-172-31-4-207.ap-southeast-1.compute.internal ntpd[3410]: Listen normally on 2 lo 127.0.0.1 UDP 123
Feb 13 04:08:40 ip-172-31-4-207.ap-southeast-1.compute.internal ntpd[3410]: Listen normally on 3 eth0 172.31.4.207 UDP 123
Feb 13 04:08:40 ip-172-31-4-207.ap-southeast-1.compute.internal ntpd[3410]: Listen normally on 4 eth0 fe80::12:d4ff:fecb:401b...123
Feb 13 04:08:40 ip-172-31-4-207.ap-southeast-1.compute.internal ntpd[3410]: Listen normally on 5 lo ::1 UDP 123
Feb 13 04:08:40 ip-172-31-4-207.ap-southeast-1.compute.internal ntpd[3410]: Listening on routing socket on fd #22 for interfa...tes
Feb 13 04:08:40 ip-172-31-4-207.ap-southeast-1.compute.internal systemd[1]: Started Network Time Service.
Feb 13 04:08:41 ip-172-31-4-207.ap-southeast-1.compute.internal ntpd[3410]: 0.0.0.0 c016 06 restart
Feb 13 04:08:41 ip-172-31-4-207.ap-southeast-1.compute.internal ntpd[3410]: 0.0.0.0 c012 02 freq_set kernel 0.000 PPM
Feb 13 04:08:41 ip-172-31-4-207.ap-southeast-1.compute.internal ntpd[3410]: 0.0.0.0 c011 01 freq_not_set
Hint: Some lines were ellipsized, use -l to show in full.
[root@ip-172-31-4-207 ~]#
```
Setup SSH with passwordless
```
** Gen Key on CM server
[root@ip-172-31-4-207 ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
98:33:f7:c4:ec:02:89:36:5c:cf:84:a6:6a:e1:4c:17 root@ip-172-31-4-207.ap-southeast-1.compute.internal
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|       .         |
|    E + .        |
|   . * B o       |
|  o B B S +      |
| + = . = +       |
|  =     . o      |
| .       .       |
|                 |
+-----------------+
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]#

** Add Key in "authorized_keys" file (All Node)
[root@ip-172-31-4-207 ~]# cd .ssh
[root@ip-172-31-4-207 .ssh]# ls
authorized_keys  id_rsa  id_rsa.pub
[root@ip-172-31-4-207 .ssh]# cat id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDZPxtmeHx/9A/mAj1pVeWSMUFiLGHEM1OpOdcL1tLFYB38uz/scHZkoeKCWHNxSw5L7PHs7t4AufSdyu+FX2BYY+myAjMFsUY9v81kMgQ/HKRaA89m1Xmsd4Hxie9Bf+vYAlWNCltmDxwjliqIlYm9j1eUI/PYcYKMd+IX2wHk3A1O8rAYrEo7CFkA5vcHKwWJpcvcJZYqYq01lWkae9Y6VMbQH59ucAnw8xejd7RftbbOixE3ysiXTKP4jEPYihmuNbC4C7gOV5vgcJ22/vqXCAvesxn6XsJG5C/7GfC00XS/TDee1vMl1wDZI6HRH86GN1IX4t5re9aiE5 root@ip-172-31-4-207.ap-southe-1.compute.internal
[root@ip-172-31-4-207 .ssh]#
[root@ip-172-31-4-207 .ssh]# vi authorized_keys
[root@ip-172-31-4-207 .ssh]#

** Test ssh to all node
[root@ip-172-31-4-207 .ssh]# ssh ip-172-31-4-207.ap-southeast-1.compute.internal
The authenticity of host 'ip-172-31-4-207.ap-southeast-1.compute.internal (172.31.4.207)' can't be established.
ECDSA key fingerprint is 5c:42:30:43:4e:f3:8d:8d:33:84:2f:ba:66:ea:53:cc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ip-172-31-4-207.ap-southeast-1.compute.internal' (ECDSA) to the list of known hosts.
Last login: Mon Feb 13 11:25:00 2017 from ip-172-31-4-207.ap-southeast-1.compute.internal
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]# ssh ip-172-31-15-226.ap-southeast-1.compute.internal
The authenticity of host 'ip-172-31-15-226.ap-southeast-1.compute.internal (172.31.15.226)' can't be established.
ECDSA key fingerprint is 1d:ea:a1:a9:54:96:4e:ad:4a:1a:1c:df:6f:7c:ed:77.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ip-172-31-15-226.ap-southeast-1.compute.internal,172.31.15.226' (ECDSA) to the list of known hosts.
[root@ip-172-31-15-226 ~]#
[root@ip-172-31-15-226 ~]#
[root@ip-172-31-15-226 ~]# exit
logout
Connection to ip-172-31-15-226.ap-southeast-1.compute.internal closed.
[root@ip-172-31-4-207 ~]# ssh ip-172-31-3-223.ap-southeast-1.compute.internal
The authenticity of host 'ip-172-31-3-223.ap-southeast-1.compute.internal (172.31.3.223)' can't be established.
ECDSA key fingerprint is e5:f3:66:45:90:4c:0a:8a:05:53:33:cf:c2:fd:ec:04.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ip-172-31-3-223.ap-southeast-1.compute.internal,172.31.3.223' (ECDSA) to the list of known hosts.
[root@ip-172-31-3-223 ~]# exit
logout
Connection to ip-172-31-3-223.ap-southeast-1.compute.internal closed.
[root@ip-172-31-4-207 ~]# ssh ip-172-31-13-39.ap-southeast-1.compute.internal
The authenticity of host 'ip-172-31-13-39.ap-southeast-1.compute.internal (172.31.13.39)' can't be established.
ECDSA key fingerprint is b4:c0:ad:6e:12:b9:7f:42:69:e0:c3:84:36:8d:86:a8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ip-172-31-13-39.ap-southeast-1.compute.internal,172.31.13.39' (ECDSA) to the list of known hosts.
[root@ip-172-31-13-39 ~]# exit
logout
Connection to ip-172-31-13-39.ap-southeast-1.compute.internal closed.
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]#
[root@ip-172-31-4-207 ~]# ssh ip-172-31-14-77.ap-southeast-1.compute.internal
The authenticity of host 'ip-172-31-14-77.ap-southeast-1.compute.internal (172.31.14.77)' can't be established.
ECDSA key fingerprint is 8b:e0:86:f2:92:dc:55:55:d7:03:c8:aa:52:42:12:c3.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ip-172-31-14-77.ap-southeast-1.compute.internal,172.31.14.77' (ECDSA) to the list of known hosts.
Last login: Mon Feb 13 10:46:31 2017
[root@ip-172-31-14-77 ~]#
[root@ip-172-31-14-77 ~]# exit
logout
Connection to ip-172-31-14-77.ap-southeast-1.compute.internal closed.
[root@ip-172-31-4-207 ~]#
```
Configure ulimit
```
[root@ip-172-31-4-207 ~]# cat /etc/security/limits.conf
# /etc/security/limits.conf
#
#This file sets the resource limits for the users logged in via PAM.
#It does not affect resource limits of the system services.
#
#Also note that configuration files in /etc/security/limits.d directory,
#which are read in alphabetical order, override the settings in this
#file in case the domain is the same or more specific.
#That means for example that setting a limit for wildcard domain here
#can be overriden with a wildcard setting in a config file in the
#subdirectory, but a user specific setting here can be overriden only
#with a user specific setting in the subdirectory.
#
#Each line describes a limit for a user in the form:
#
#<domain>        <type>  <item>  <value>
#
#Where:
#<domain> can be:
#        - a user name
#        - a group name, with @group syntax
#        - the wildcard *, for default entry
#        - the wildcard %, can be also used with %group syntax,
#                 for maxlogin limit
#
#<type> can have the two values:
#        - "soft" for enforcing the soft limits
#        - "hard" for enforcing hard limits
#
#<item> can be one of the following:
#        - core - limits the core file size (KB)
#        - data - max data size (KB)
#        - fsize - maximum filesize (KB)
#        - memlock - max locked-in-memory address space (KB)
#        - nofile - max number of open file descriptors
#        - rss - max resident set size (KB)
#        - stack - max stack size (KB)
#        - cpu - max CPU time (MIN)
#        - nproc - max number of processes
#        - as - address space limit (KB)
#        - maxlogins - max number of logins for this user
#        - maxsyslogins - max number of logins on the system
#        - priority - the priority to run user process with
#        - locks - max number of file locks the user can hold
#        - sigpending - max number of pending signals
#        - msgqueue - max memory used by POSIX message queues (bytes)
#        - nice - max nice priority allowed to raise to values: [-20, 19]
#        - rtprio - max realtime priority
#
#<domain>      <type>  <item>         <value>
#

* - nofile 32768
* - nproc 65536

#*               soft    core            0
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#@student        -       maxlogins       4

# End of file
[root@ip-172-31-4-207 ~]#
```
Disable SELinux
```
[root@ip-172-31-4-207 hadoop-hdfs]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
[root@ip-172-31-4-207 hadoop-hdfs]# setenforce 0
[root@ip-172-31-4-207 hadoop-hdfs]# sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
[root@ip-172-31-4-207 hadoop-hdfs]#

[root@ip-172-31-4-207 hadoop-hdfs]# cat /etc/sysconfig/selinux

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted


[root@ip-172-31-4-207 hadoop-hdfs]#
```
