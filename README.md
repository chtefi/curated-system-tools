A list of tools to help debugging issues or simply check what's going on in the system.

## Classic

> Linux is assumed. On OSX, options can be way different.

- `top cw` : something's taking up all the cpu or mem ?
- `htop` : a colorful top, easy to play with
- `ps fauxww` : list of all processes with command line + hierarchy
- `free -h` : memory and swap
- `df -h` : mount points
- `iptables -L -v` : firewall rules
- `dmesg -T`: kernel messages. Can be fulfilled of iptables denied message :-) or other useful stuff to check in case of problems
- `env`: list the environment variables
- `uptime`: checkout 1min/5min/15min load average
- `strace`: trace system calls and signals a program does (file open, read, stat, mmap, ...). `strace -e open uptime 2>&1`
- `lsof`: list opened files (and sockets): `lsof -i -n -p`: sockets, `lsof /var` which processes is opening files in `/var`
- `lsblk`: list info about block devices, useful to see disk not mount but handled differently
- `ping -c 1 $(ifconfig | grep broadcast | cut -d' ' -f6) && arp -a`: ping the broadcast address to list network connected devices

## System resources

List of tools used to look after system performances (mem, cpu, disks, network, processes, files..) :

- [sysdig](http://www.sysdig.org/) : a console ui to monitor (live and snapshots) several aspects of the system `sudo sysdig 'proc.name=java' -w ~/sysdig.scap`
- [iostat](http://linuxcommand.org/man_pages/iostat1.html) : i/o accesses `iostat -m -x -d 2`
- [vmstat](http://linuxcommand.org/man_pages/vmstat8.html) : mem/swap/cpu `vmstat 1`
- [mpstat](http://linuxcommand.org/man_pages/mpstat1.html) : check the stats for each cores, useful to spot single-threaded apps (if unbalanced) `mpstat -P ALL 1`
- [ifstat](https://linux.die.net/man/1/ifstat) : like iostat, vmstat, but for network interfaces
- [netstat](https://linux.die.net/man/8/netstat) : details about all the network connections of the system `netstat -putel`. `netstat -anr`
- ss : an easier netstat? `ss -nlts src :10010` or more explicit `ss -n4lt '(sport = :5000 or dport = :5000)'` (*n*umeric, *l*isten, *t*cp, *s*ummary ipv*4*) Useful to look at send/receive tcp/udp queues (can indicate congestion)
- [ss](https://linux.die.net/man/8/netstat) : a bit like netstat, list all sockets (tcp/udp), their state `ss -ta` (TCP, all)
- [dstat](http://dag.wiee.rs/home-made/dstat/) :  *stat all-in-one
- [sar](http://linuxcommand.org/man_pages/sar1.html) : monitor network, devices `sar -n DEV 2` All commands in a nice pic: http://www.brendangregg.com/Perf/linux_observability_sar.png
- [iotop](http://guichaz.free.fr/iotop/) : top, with i/o !
- [iperf](https://iperf.fr/) : test maximum bandwidth (tcp/udp) `iperf -c server -f m -d`
- [ulimit](http://ss64.com/bash/ulimit.html): memory, open files, and misc size limits for the user (often, the open file limit must be raised if the server contains hot apps) `ulimit -n 2000000` (open file descriptors)

Another repo with great scripts using ftrace under the hood: https://github.com/brendangregg/perf-tools

## Network

- [dig](https://linux.die.net/man/1/dig): query dns servers `dig +short github.com` `dig +nocmd github.com any +multiline +noall +answer`
- [traceroute](https://linux.die.net/man/8/traceroute): find the way to any host. This website is nice to test from multiple locations around the world: http://mtr.guru/
- [host](https://linux.die.net/man/1/host): resolve dns/ip `host -t ANY github.com`
- [lnstat](https://linux.die.net/man/8/lnstat): network stats (arp cache, route cache, nf and ip conntrack entries..): `lnstat -j`
- nmap: The famous tool to know which ports are opened: `nmap -sT -vv -p 1-65535 [ip]`
- tcpdump: listen to what's going on on the network interfaces: `tcpdump -i lo -A dst port 8080` (`-A` for ascii, eg: for HTTP)
- ngrep: a simpler? tcpdump with grep features! can listen to specific or all interfaces, given port, and match patterns.
```
$ ngrep -d any "Value" port 2003
interface: any
filter: (ip or ip6) and ( port 2003 )
match: Value
####
T 172.17.0.1:54820 -> 172.17.0.2:2003 [AP]
  com.ctheu.test.Value 42 1486331086.
```
For HTTP requests, it's better to use:
```ngrep -d any -q port 8081 -W byline```

To monitor multicast:
```ngrep -q -W byline '' multicast```

## System devices

- [hdparm](https://linux.die.net/man/8/hdparm) : check drives settings `hdparm -Tt /dev/sda8`
- [ethtool](http://linuxcommand.org/man_pages/ethtool8.html) : check the ethernet cards settings (speed, duplex etc. if you have a doubt) `ethtool eth0`

## Topology

- [lstopo](https://linux.die.net/man/1/lstopo): a wonderful tool to draw the topology of the server (show cpus, their caches, the physical sockets, the memory) into a nice big picture `lstopo --output-format txt -v`

## Performance

A tons of good links and presentations here: http://www.brendangregg.com/linuxperf.html.

## Java specifics

- [jstat](http://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr017.html) : like iostat, vmstat, for java processes `jstat -gc -t -h30 [vmid] 1s : monitor Java GC`
- [jvisualvm](https://visualvm.github.io/) : packaged with java, ultra useful
- [jmc](https://docs.oracle.com/javacomponents/jmc-5-5/jmc-user-guide/jmc.htm) : Java Mission Control. A better jvisualvm

## System tuning

- `/proc/sys/vm/vfs_cache_pressure`
- `/proc/sys/vm/swappiness`
- `/proc/sys/vm/zone_reclaim_mode` (Disable NUMA)

## Misc info

- `cat /proc/cpuinfo` : list of cpus of the system with details (type, MHz, cache size..)
- `lscpu` : shorter
- `/proc/sys/fs/nr_open`: hard limit of the current number of file handles the kernel can handle
- `/proc/sys/fs/file-max`: current number of file handles the kernel can handle
- `/proc/sys/fs/file-nr`: file handles currently opened/used file handles/the max (= file-max)
- `/proc/sys/vm/nr_hugepages`: map huge memory pages (if using Java with a big heap, set also +UseLargePages)

`sysctl` can be used to change the values: `sysctl -w fs.file-max=786046`. Or `/etc/sysctl.conf`.

## Network tuning

Flags I grab here and there, not optimal or anything, just to know they exist.

- `net.ipv4.tcp_slow_start_after_idle = 0`
- `net.core.netdev_max_backlog = 5000`
- `net.ipv4.tcp_no_metrics_save = 1`
- `net.ipv4.tcp_sack = 1`
- `net.ipv4.tcp_timestamps = 1`
- `net.ipv4.tcp_window_scaling = 1`
- `net.core.wmem_max = 12582912`
- `net.core.rmem_max = 12582912`
- `net.ipv4.tcp_rmem = 10240 87380 12582912` (tcp receive buffer thresholds)
- `net.ipv4.tcp_wmem = 10240 87380 12582912` (tcp sendbuffer buffer thresholds)
- `net.ipv4.tcp_mem = 10000000 10000000 10000000` (tcp memory autotuning, define low/middle/max thresholds)

https://wwwx.cs.unc.edu/~sparkst/howto/network_tuning.php

- nf_conntrack can be very important too

```
sysctl -w fs.file-max="9999999"
sysctl -w fs.nr_open="9999999"
sysctl -w net.core.netdev_max_backlog="4096"
sysctl -w net.core.rmem_max="16777216"
sysctl -w net.core.somaxconn="65535"
sysctl -w net.core.wmem_max="16777216"
sysctl -w net.ipv4.ip_local_port_range="1025       65535"
sysctl -w net.ipv4.tcp_fin_timeout="30"
sysctl -w net.ipv4.tcp_keepalive_time="30"
sysctl -w net.ipv4.tcp_max_syn_backlog="20480"
sysctl -w net.ipv4.tcp_max_tw_buckets="400000"
sysctl -w net.ipv4.tcp_no_metrics_save="1"
sysctl -w net.ipv4.tcp_syn_retries="2"
sysctl -w net.ipv4.tcp_synack_retries="2"
sysctl -w net.ipv4.tcp_tw_recycle="1"
sysctl -w net.ipv4.tcp_tw_reuse="1"
sysctl -w vm.min_free_kbytes="65536"
sysctl -w vm.overcommit_memory="1"
sysctl -w net.ipv4.tcp_slow_start_after_idle="0"
ulimit -n 9999999
```

```
net.ipv4.ip_local_port_range = 18000    65535
net.ipv4.netfilter.ip_conntrack_tcp_timeout_time_wait = 1
```
http://www.lognormal.com/blog/2012/09/27/linux-tcpip-tuning/

Size of the conntrack table:`sysctl net.netfilter.nf_conntrack_count` (the limit being `sysctl net.nf_conntrack_max`)
See `lnstat -j`

To do some testing, it's possible to alter the quality of the network traffic:
- `tc qdisc add dev wlan0 root netem loss 10%`
- `tc qdisc add dev eth0 root netem delay 80ms 15ms distribution normal`

# Resources

- http://techblog.netflix.com/2015/11/linux-performance-analysis-in-60s.html
- http://www.brendangregg.com/blog/2016-12-27/linux-tracing-in-15-minutes.html
