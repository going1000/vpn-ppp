将 www.baidu.com 转换为 ip ==》 通过 route table 决定出口 ==》 得到结果

在osx下面，我们使用nslookup来查看访问一个域名时候的使用的dns。用traceroute工具来查看详细路由信息，用netstat来查看本机的路由表。

nslookup 例子：

    going1000@osx ppp $ nslookup www.baidu.com
    Server: 192.168.1.1
    Address:    192.168.1.1#53

    Non-authoritative answer:
    www.baidu.com   canonical name = www.a.shifen.com.
    Name:   www.a.shifen.com
    Address: 115.239.211.110
    Name:   www.a.shifen.com
    Address: 115.239.210.27

traceroute 例子：

    going1000@osx ppp $ traceroute www.163.com
    traceroute: Warning: www.163.com has multiple addresses; using 101.227.66.158
    traceroute to 163.xdwscache.glb0.lxdns.com (101.227.66.158), 64 hops max, 52 byte packets
     1 192.168.0.1 (192.168.0.1) 303.570 ms 58.269 ms 161.238 ms
     2 192.168.1.1 (192.168.1.1) 28.019 ms 23.551 ms 33.749 ms
     3 1.244.79.218.broad.xw.sh.dynamic.163data.com.cn (218.79.244.1) 19.224 ms 15.921 ms 13.544 ms
     4 124.74.12.169 (124.74.12.169) 228.705 ms 244.191 ms 197.058 ms
     5 124.74.215.217 (124.74.215.217) 252.492 ms 237.622 ms 446.333 ms
     6 61.152.80.2 (61.152.80.2) 340.711 ms 284.042 ms 214.222 ms
     7 * 124.74.233.98 (124.74.233.98) 759.646 ms *
     8 101.227.66.2 (101.227.66.2) 761.930 ms 607.199 ms *
     9 101.227.66.6 (101.227.66.6) 201.392 ms
        101.227.66.158 (101.227.66.158) 115.748 ms
        101.227.66.6 (101.227.66.6) 1020.250 ms

netstat 例子：

    going1000@osx ppp $ netstat -nr
    Routing tables

    Internet:
    Destination Gateway Flags Refs Use Netif Expire
    default 192.168.0.1 UGSc 13 66 en0
    127 127.0.0.1 UCS 0 0 lo0
    127.0.0.1 127.0.0.1 UH 129 688863 lo0

##二、vpn的概念
通俗地讲，vpn就是重新定制了路由转发规则。将ip包通过vpn服务器作转发。这个概念和proxy是不是很类似？只是vpn的目的是提供访问内网的方式而已。

##三、怎么设置vpn？
通过上面的一些解释，我们可以了解到，设置vpn会有两个步骤：

0、添加vpn基本信息

![](http://going1000sblog-image.stor.sinaapp.com/add_vpn.png)

1、设置dns

![](http://going1000sblog-image.stor.sinaapp.com/set_vpn_dns.png)

比如我的公司的内部dns服务器是192.168.1.98

2、设置所有包通过vpn

![](http://going1000sblog-image.stor.sinaapp.com/set_vpn.png)

3、这时候，看看内网是不是可以用了

    going1000@osx ppp $ netstat -nr
    Routing tables

    Internet:
    Destination Gateway Flags Refs Use Netif Expire
    default 180.166.126.91 UGSc 186 3 ppp0
    default 192.168.0.1 UGScI 10 0 en0
    127 127.0.0.1 UCS 0 0 lo0
    192.168.189 ppp0 USc 0 0 ppp0

dns信息

    going1000@osx ppp $ nslookup db.kfs.dev.anjuke.com
    Server: 192.168.1.98
    Address:    192.168.1.98#53

    Name:   db.kfs.dev.anjuke.com
    Address: 192.168.1.167

4、但是问题来了，上不了外网

##四、优化设置，使得连接vpn同时能够上外网
我们依旧和上面一样，一步步来

0、添加vpn基本信息

如上

1、设置dns

如上

2、设置route table

![](http://going1000sblog-image.stor.sinaapp.com/set_vpn_2.png)

![](http://going1000sblog-image.stor.sinaapp.com/set_vpn_3.png)

添加一条路由转发规则

    sudo route add 192.0.0.0 -interface ppp0

这个时候，内网ip是不是可以ping通了？但是现在也有问题，你突然发现之前能解析的内网域名没法接解析了

3、再次设置dns

域名无法解析，一定是dns出了问题，用nslookup看看：

    going1000@osx ppp $ nslookup db.kfs.dev.anjuke.com
    Server: 192.168.0.1
    Address:    192.168.0.1#53

    Non-authoritative answer:
    Name:   db.kfs.dev.anjuke.com
    Address: 180.168.41.175

果然没有使用到dns服务器，原因很简单，因为上面1设置dns服务器只是针对于vpn连接，只有一种情况会生效：route table default 是vpn的时候。（这里多说一句，在mac里，有两种方式可以达到这种效果：1）vpn选项中选择
