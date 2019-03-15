---
layout: post
title: ISIS 在 IPv6 网络中的配置
date: 2019-03-15 21:12:00
categories: 
- Networking
tags:
- Routing
- ISIS
- IPv6
- IGP
---

# 网络拓扑

组网拓扑图如下：

    +-------------------------------------------+
    |                                           |
    |      +---------+ 10:1::2/64               |
    |      | RouterA +----------+               |  +----------------------+
    |      +---------+          |               |  |                      |
    |           L1              |               |  |       20::1/64 +     |
    |                           | 10:1::1/64    |  |                |     |
    |           10:2::1/64 +----+----+          |  |          +-----+---+ |
    |            +---------+ RouterC +------------------------+ RouterD | |
    |            |         +---------+ 30::1/64 |  | 30::2/64 +---------+ |
    | 10:2::2/64 |            L1/2              |  |               L2     |
    |      +-----+---+                          |  |                      |
    |      | RouterB |                          |  |                      |
    |      +---------+                          |  |       Area 20        |
    |           L1                              |  |                      |
    |               Area 10                     |  +----------------------+
    |                                           |
    +-------------------------------------------+

要求这 4 台路由器上的网络互通，同时模拟 A 和 B 路由器较为低端，使其处理较少的数据信息。

# 配置

注意对应接口，没有画在拓扑图上。

路由器 A：

    sy
    sy RouterA
    ipv6
    isis 1
    is-level level-1
    net 10.0000.0000.0001.00
    ipv6 enable
    q
    int g0/0/0
    ipv6 enable
    ipv6 add 10:1::2/64
    isis ipv6 enable 1
    q

路由器 B：

    sy
    sy RouterB
    ipv6
    isis 1
    is-level level-1
    net 10.0000.0000.0002.00
    ipv6 enable
    q
    int g0/0/0
    ipv6 en
    ipv6 add 10:2::2/64
    isis ipv6 en 1
    q

路由器 C：

    sy
    sy RouterC
    ipv6
    isis 1
    net 10.0000.0000.0003.00
    ipv6 en
    q
    int g0/0/0
    ipv6 en
    ipv6 add 10:1::1/64
    isis ipv6 en 1
    q
    int g0/0/1
    ipv6 en
    ipv6 add 10:2::1/64
    isis ipv6 en 1
    q
    int g0/0/2
    ipv6 en
    ipv6 add 30::1/64
    isis ipv6 en 1
    isis circuit-level level-2
    q

路由器 D：

    sy
    sy RouterD
    ipv6
    isis 1
    is-level level-2
    net 20.0000.0000.0004.00
    ipv6 en
    q
    int g0/0/0
    ipv6 en
    ipv6 add 30::2/64
    isis ipv6 en 1
    q
    int lo0
    ipv6 en
    ipv6 add 20::1/64
    isis ipv6 en 1
    q

# 验证结果

## 从路由器 A 查看 ISIS 路由表

在路由器 A 上执行 `dis isis route`，结果如下：

    [RouterA]dis isis route

                            Route information for ISIS(1)
                            -----------------------------

                            ISIS(1) Level-1 Forwarding Table
                            --------------------------------

    IPV4 Destination     IntCost    ExtCost ExitInterface   NextHop         Flags
    -------------------------------------------------------------------------------
    0.0.0.0/0            10         NULL

    IPV6 Dest.      ExitInterface   NextHop                       Cost       Flags
    -------------------------------------------------------------------------------
    ::/0            GE0/0/0         FE80::2E0:FCFF:FEEC:6CE7      10         A/-/-
    10:1::/64       GE0/0/0         Direct                        10         D/L/-
    10:2::/64       GE0/0/0         FE80::2E0:FCFF:FEEC:6CE7      20         A/-/-

        Flags: D-Direct, A-Added to URT, L-Advertised in LSPs, S-IGP Shortcut,
                                U-Up/Down Bit Set

可以看到，下一条都是使用的链路本地地址。明细路由只有 `10:1::/64` 和 `10:2::/64` 两个网段（其中一个是直连），到路由器 D 的 loopback 口是通过默认路由来走的。可以 ping 测试：

    [RouterA]ping ipv6 20::1
      PING 20::1 : 56  data bytes, press CTRL_C to break
        Request time out
        Reply from 20::1
        bytes=56 Sequence=2 hop limit=63  time = 30 ms
        Reply from 20::1
        bytes=56 Sequence=3 hop limit=63  time = 50 ms
        Reply from 20::1
        bytes=56 Sequence=4 hop limit=63  time = 30 ms
        Reply from 20::1
        bytes=56 Sequence=5 hop limit=63  time = 40 ms

      --- 20::1 ping statistics ---
        5 packet(s) transmitted
        4 packet(s) received
        20.00% packet loss
        round-trip min/avg/max = 30/37/50 ms

## 查看路由器 C 的邻居信息

在路由器 C 上执行 `dis isis peer`，可以查看邻居信息。如果要查看详细信息，则在命令后面加上 `verbose`。结果如下：

    [RouterC]dis isis peer verbose

                            Peer information for ISIS(1)

    System Id     Interface          Circuit Id       State HoldTime Type     PRI
    -------------------------------------------------------------------------------
    0000.0000.0001  GE0/0/0            0000.0000.0003.01 Up   30s      L1       64

    MT IDs supported     : 0(UP)
    Local MT IDs         : 0
    Area Address(es)     : 10
    Peer IPv6 Address(es): FE80::2E0:FCFF:FE4A:3D72
    Uptime               : 00:13:22
    Adj Protocol         : IPV6
    Restart Capable      : YES
    Suppressed Adj       : NO
    Peer System Id       : 0000.0000.0001  

    0000.0000.0002  GE0/0/1            0000.0000.0003.02 Up   26s      L1       64

    MT IDs supported     : 0(UP)
    Local MT IDs         : 0
    Area Address(es)     : 10
    Peer IPv6 Address(es): FE80::2E0:FCFF:FE08:27DD
    Uptime               : 00:12:55
    Adj Protocol         : IPV6
    Restart Capable      : YES
    Suppressed Adj       : NO
    Peer System Id       : 0000.0000.0002  

    0000.0000.0004  GE0/0/2            0000.0000.0003.03 Up   22s      L2       64

    MT IDs supported     : 0(UP)
    Local MT IDs         : 0
    Area Address(es)     : 20
    Peer IPv6 Address(es): FE80::2E0:FCFF:FED1:6D7B
    Uptime               : 00:08:00
    Adj Protocol         : IPV6
    Restart Capable      : YES
    Suppressed Adj       : NO
    Peer System Id       : 0000.0000.0004  


    Total Peer(s): 3

在这张表中，能够看到路由器 C 的三个 ISIS 邻居，它们的 System ID（也就是 net 地址除了最后 `.00` 之外的后面三段）、level（两个 level-1 邻居和一个 level-2 邻居），以及对应的接口。

## 查看路由器 C 的 LSDB 信息

通过命令 `dis isis lsdb` 查看 LSDB 信息。同样可以加上 `verbose` 以显示详细信息。

    [RouterC]dis isis lsdb

                            Database information for ISIS(1)
                            --------------------------------

                            Level-1 Link State Database

    LSPID                 Seq Num      Checksum      Holdtime      Length  ATT/P/OL
    -------------------------------------------------------------------------------
    0000.0000.0001.00-00  0x00000005   0xfcee        475           86      0/0/0
    0000.0000.0002.00-00  0x00000006   0x6581        730           86      0/0/0
    0000.0000.0003.00-00* 0x0000000b   0x5a16        832           143     1/0/0
    0000.0000.0003.01-00* 0x00000002   0xb213        832           55      0/0/0
    0000.0000.0003.02-00* 0x00000002   0xc7fb        832           55      0/0/0

    Total LSP(s): 5
        *(In TLV)-Leaking Route, *(By LSPID)-Self LSP, +-Self LSP(Extended),
            ATT-Attached, P-Partition, OL-Overload


                            Level-2 Link State Database

    LSPID                 Seq Num      Checksum      Holdtime      Length  ATT/P/OL
    -------------------------------------------------------------------------------
    0000.0000.0003.00-00* 0x0000000c   0x90f7        832           146     0/0/0
    0000.0000.0003.03-00* 0x00000002   0xf8c7        832           55      0/0/0
    0000.0000.0004.00-00  0x00000006   0xad48        404           116     0/0/0

    Total LSP(s): 3
        *(In TLV)-Leaking Route, *(By LSPID)-Self LSP, +-Self LSP(Extended),
            ATT-Attached, P-Partition, OL-Overload