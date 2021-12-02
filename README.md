# xdp-redirect-lab
xdp redirect veth pair conn lab




### XDP REDIRECT 

# iperf1

```
172.17.0.2
vethd32424b
02:42:ac:11:00:02

./xdp_loader --dev vethd32424b --force --progsec xdp_redirect_map

./xdp_prog_user -d vethd32424b -r  veth70d1189  --src-mac 02:42:ac:11:00:02 --dest-mac 02:42:ac:11:00:03

```

# iperf3

```
172.17.0.3
veth70d1189
02:42:ac:11:00:03





./xdp_loader --dev veth70d1189 --force --progsec xdp_redirect_map

./xdp_prog_user -d veth70d1189 -r vethd32424b --src-mac 02:42:ac:11:00:03 --dest-mac 02:42:ac:11:00:02

```



### Lab




### 没使用xdp加速

```
/ $ iperf3 -c 172.17.0.3
Connecting to host 172.17.0.3, port 5201
[  5] local 172.17.0.2 port 40596 connected to 172.17.0.3 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec  4.05 GBytes  34.8 Gbits/sec  388    930 KBytes
[  5]   1.00-2.00   sec  3.98 GBytes  34.2 Gbits/sec  449    940 KBytes
[  5]   2.00-3.00   sec  4.05 GBytes  34.8 Gbits/sec  493    963 KBytes
[  5]   3.00-4.00   sec  3.71 GBytes  31.8 Gbits/sec    3    994 KBytes
[  5]   4.00-5.00   sec  4.02 GBytes  34.5 Gbits/sec    3   1010 KBytes
[  5]   5.00-6.00   sec  4.08 GBytes  35.1 Gbits/sec    5   1020 KBytes
[  5]   6.00-7.00   sec  3.95 GBytes  34.0 Gbits/sec    2   1.01 MBytes
[  5]   7.00-8.00   sec  4.01 GBytes  34.5 Gbits/sec    2   1.01 MBytes
[  5]   8.00-9.00   sec  3.67 GBytes  31.5 Gbits/sec    5   1.01 MBytes
[  5]   9.00-10.00  sec  3.91 GBytes  33.5 Gbits/sec    0   1.03 MBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  39.4 GBytes  33.9 Gbits/sec  1350             sender
[  5]   0.00-10.00  sec  39.4 GBytes  33.9 Gbits/sec                  receiver

```

#### 使用XDP加速后

```
iperf Done.
/ $ iperf3 -c 172.17.0.3
Connecting to host 172.17.0.3, port 5201
[  5] local 172.17.0.2 port 40722 connected to 172.17.0.3 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   782 MBytes  6.56 Gbits/sec    0    269 KBytes
[  5]   1.00-2.00   sec   772 MBytes  6.48 Gbits/sec    0    426 KBytes
[  5]   2.00-3.00   sec   796 MBytes  6.68 Gbits/sec    0    426 KBytes
[  5]   3.00-4.00   sec   784 MBytes  6.58 Gbits/sec  104    199 KBytes
[  5]   4.00-5.00   sec   719 MBytes  6.02 Gbits/sec    6    197 KBytes
[  5]   5.00-6.00   sec   775 MBytes  6.50 Gbits/sec    0    225 KBytes
[  5]   6.00-7.00   sec   573 MBytes  4.81 Gbits/sec   53    269 KBytes
[  5]   7.00-8.00   sec   821 MBytes  6.89 Gbits/sec    0    448 KBytes
[  5]   8.00-9.00   sec   650 MBytes  5.44 Gbits/sec  376    218 KBytes
[  5]   9.00-10.00  sec   815 MBytes  6.83 Gbits/sec    0    249 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  7.31 GBytes  6.28 Gbits/sec  539             sender
[  5]   0.00-10.00  sec  7.31 GBytes  6.28 Gbits/sec                  receiver

```


#### XDP status
```
XDP-action  
XDP_ABORTED            0 pkts (         0 pps)           0 Kbytes (     0 Mbits/s) period:2.000435
XDP_DROP               0 pkts (         0 pps)           0 Kbytes (     0 Mbits/s) period:2.000441
XDP_PASS          659425 pkts (         0 pps)       43524 Kbytes (     0 Mbits/s) period:2.000438
XDP_TX                 0 pkts (         0 pps)           0 Kbytes (     0 Mbits/s) period:2.000434
XDP_REDIRECT           0 pkts (         0 pps)           0 Kbytes (     0 Mbits/s) period:2.000429
```


### 尝试关掉一些一些参数
```
root@ubuntu-hirsute:~/withlin/xdp-tutorial/packet03-redirecting# nsenter --net=/var/run/docker/netns/ff0a5996e2d2  ethtool  -K eth0 tso offroot@ubuntu-hirsute:~/withlin/xdp-tutorial/packet03-redirecting# nsenter --net=/var/run/docker/netns/ff0a5996e2d2  ethtool  -K eth0 lro  offroot@ubuntu-hirsute:~/withlin/xdp-tutorial/packet03-redirecting# nsenter --net=/var/run/docker/netns/1eebaa5a17b4 ethtool -K eth0 tso offroot@ubuntu-hirsute:~/withlin/xdp-tutorial/packet03-redirecting# nsenter --net=/var/run/docker/netns/1eebaa5a17b4 ethtool -K eth0 lro  off
root@ubuntu-hirsute:~/withlin/xdp-tutorial/packet03-redirecting# nsenter --net=/var/run/docker/netns/1eebaa5a17b4 ethtool -K eth0 gso  off
root@ubuntu-hirsute:~/withlin/xdp-tutorial/packet03-redirecting# nsenter --net=/var/run/docker/netns/1eebaa5a17b4 ethtool -K eth0 gro  off
root@ubuntu-hirsute:~/withlin/xdp-tutorial/packet03-redirecting# nsenter --net=/var/run/docker/netns/ff0a5996e2d2  ethtool  -K eth0 gso  offroot@ubuntu-hirsute:~/withlin/xdp-tutorial/packet03-redirecting# nsenter --net=/var/run/docker/netns/ff0a5996e2d2  ethtool  -K eth0 gro  offroot@ubuntu-hirsute:~/withlin/xdp-tutorial/packet03-redirecting# 

```

#### 总结

加速 加了个寂寞
