---
title: 服务器带宽测速iperf3工具使用
author: 要名俗气
type: post
date: 2024-05-03T05:52:37+00:00
url: /2024/how-to-use-iperf3
description: 网络测速相信很多人都使用过，想要看看自己家里的宽带网速怎么样，最简单的方式就是搜索[测速网](https://www.speedtest.cn/)，然后等结果出来就可以了，比较简单。今天要介绍的主角是：iperf3,由于经常要和vps打交到，想要测试本地到服务器的速度，使用iperf3会更准确一点，接下来就介绍一下[iperf3](https://iperf.fr/)，以及如何使用[iperf3](https://iperf.fr/)来进行测试速度。
featured_image: /wp-content/uploads/2024/05/iperf_logo.gif
categories:
  - 工具
tags:
  - iperf3
---
网络测速相信很多人都使用过，想要看看自己家里的宽带网速怎么样，最简单的方式就是搜索[测速网](https://www.speedtest.cn/)，然后等结果出来就可以了，比较简单。今天要介绍的主角是：iperf3,由于经常要和vps打交到，想要测试本地到服务器的速度，使用iperf3会更准确一点，接下来就介绍一下[iperf3](https://iperf.fr/)，以及如何使用[iperf3](https://iperf.fr/)来进行测试速度。

## iperf3是什么

[iPerf3](https://iperf.fr/) 是一种主动测量 IP 网络上可实现的最大带宽的工具。 它支持调整与计时、缓冲区和协议（TCP、UDP、SCTP 与 IPv4 和 IPv6）相关的各种参数。 对于每次测试，它都会报告带宽、损耗和其他参数。它是一个 C/S 类型的测速工具，在服务器上开放某个端口，然后在客户机上连接该服务器对应的端口，就可以开始进行 tcp 或 udp 的下载速度了

## iperf3安装

iperf3的官网也详细介绍了几种操作平台的安装，具体的地址：[安装iperf3](https://iperf.fr/iperf-download.php)。下边说明一下几种操作系统的安装。

### Windows

直接去官方下载地址：[budman.pw](https://files.budman.pw/)或者[github](https://github.com/ar51an/iperf3-win-builds/releases)去下载安装文件,下载最新的就可以了。

### Mac OS

mac可以使用homebrew来进行安装，如果还没有安装homebrew可以参考我的另一篇文章来进行安装：[Mac更换源更快速的安装Homebrew](https://www.iminling.com/2023/10/02/265.html "Mac更换源更快速的安装Homebrew").然后通过命令`brew install iperf3`。

### Ubuntu/Debian

使用命令：`sudo apt-get install iperf3`进行安装。如果当前用户是root用户，则可以省略`sudo`.

### Centos/Red hat

使用命令：`yum install iperf3`进行安装。

## iperf3使用

iperf3分为客户端和服务端，服务端开放端口，客户端则连上对应的端口进行发包，达到测速。下边来使用命令来快速的进行一轮测速：

### 服务端

服务端执行命令`iperf3 -s -p 1111`就启动了：

```
root@docker:~# iperf3 -s -p 1111
-----------------------------------------------------------
Server listening on 1111 (test #1)
-----------------------------------------------------------
```

服务端开启`1111`端口后就等待客户端连上来。

### 客户端

客户端执行命令<span class="s1"><code>iperf3 -c 192.168.1.222 -p 1111 -i 1 -t 10</code> ,其中<code>192.168.1.222</code>是你服务端所在服务器的ip，执行后客户端输出如下：</span>

```
$ iperf3 -c 192.168.1.222 -p 1111 -i 1 -t 10

Connecting to host 192.168.1.222, port 1111
Reverse mode, remote host 192.168.1.222 is sending
[  5] local 192.168.1.10 port 56431 connected to 192.168.1.222 port 1111
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec  52.1 MBytes   435 Mbits/sec
[  5]   1.00-2.00   sec  67.4 MBytes   567 Mbits/sec
[  5]   2.00-3.00   sec  58.6 MBytes   492 Mbits/sec
[  5]   3.00-4.00   sec  71.4 MBytes   600 Mbits/sec
[  5]   4.00-5.00   sec  69.2 MBytes   579 Mbits/sec
[  5]   5.00-6.00   sec  69.8 MBytes   586 Mbits/sec
[  5]   6.00-7.00   sec  67.5 MBytes   568 Mbits/sec
[  5]   7.00-8.00   sec  70.9 MBytes   593 Mbits/sec
[  5]   8.00-9.00   sec  73.1 MBytes   613 Mbits/sec
[  5]   9.00-10.00  sec  73.1 MBytes   613 Mbits/sec
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.03  sec   676 MBytes   565 Mbits/sec    2             sender
[  5]   0.00-10.00  sec   673 MBytes   564 Mbits/sec                  receiver

iperf Done.
```

一次测速就完成了。服务端也会有同样的日志输出。可见速度大概是500Mb/s。

## 参数详解

具体的参数详细解释可以参考[官网的文档](https://iperf.fr/iperf-doc.php);这里我也简单的介绍一下。

### 通用参数

通用参数就是客户端和服务端都可以使用的参数。

<table class="tableau">
  <tr>
    <th colspan="2">
      通用选项
    </th>
  </tr>

  <tr>
    <th>
      命令行选项
    </th>

    <th>
      说明
    </th>
  </tr>

  <tr>
    <td id="3port">
      **-p**, --port ***n***
    </td>

    <td>
      服务器侦听和客户端连接的服务器端口。 这在客户端和服务器中应该是相同的。 默认值为 5201。
    </td>
  </tr>

  <tr>
    <td id="3cport">
      **--cport** ***n***
    </td>

    <td>
      指定客户端端口的选项。 (iPerf 3.1新特性)
    </td>
  </tr>

  <tr>
    <td id="3format">
      **-f**, --format ***[kmKM]***
    </td>

    <td>
      指定打印带宽数字的格式的字母。支持的格式有</p>

      <pre>    'k' = Kbits/sec           'K' = KBytes/sec
    'm' = Mbits/sec           'M' = MBytes/sec</pre>

<p>
  自适应格式根据需要在千级和兆级之间进行选择。</td> </tr>

  <tr>
    <td id="3interval">
      **-i**, --interval ***n***
    </td>

    <td>
      设置周期性带宽、抖动和丢失报告之间的间隔时间（以秒为单位）。 如果非零，则自上次报告以来每隔带宽秒数就会生成一份报告。 如果为零，则不打印定期报告。 默认为零。
    </td>
  </tr>

  <tr>
    <td id="3filname">
      **-F**, --file name
    </td>

    <td>
      **客户端:** 从文件中读取并写入网络，而不是使用随机数据；<br /> **服务端:** 从网络读取并写入文件，而不是丢弃数据。
    </td>
  </tr>

  <tr>
    <td id="3affinity">
      **-A**, --affinity ***n/n,m-F***
    </td>

    <td>
      如果可能的话，设置 CPU 关联性（仅限 Linux 和 FreeBSD）。 在客户端和服务器上，您可以使用此参数的 n 形式（其中 n 是 CPU 编号）来设置本地关联性。 此外，在客户端，您可以使用 n,m 形式的参数覆盖服务器的关联性，仅用于该测试。 请注意，使用此功能时，进程将仅绑定到单个 CPU（而不是包含潜在多个 CPU 的集合）。
    </td>
  </tr>

  <tr>
    <td id="3bind">
      **-B**, --bind ***host***
    </td>

    <td>
      绑定到特定网卡，即该计算机的地址之一。 对于客户端，这设置出站网卡接口。 对于服务器，这设置传入网卡接口。 这仅在具有多个网卡接口的多宿主主机上有用。
    </td>
  </tr>

  <tr>
    <td id="3verbose">
      **-V**, --verbose
    </td>

    <td>
      给出更详细的输出
    </td>
  </tr>

  <tr>
    <td id="3json">
      **-J**, --json
    </td>

    <td>
      以 JSON 格式输出
    </td>
  </tr>

  <tr>
    <td id="3logfile">
      **--logfile** file
    </td>

    <td>
      将输出发送到日志文件。(iPerf 3.1新特性)
    </td>
  </tr>

  <tr>
    <td id="3debug">
      **--d**, --debug
    </td>

    <td>
      发出调试输出。 主要供开发人员使用。
    </td>
  </tr>

  <tr>
    <td id="3version">
      **-v**, --version
    </td>

    <td>
      显示版本信息并退出。
    </td>
  </tr>

  <tr>
    <td id="3help">
      **-h**, --help
    </td>

    <td>
      显示帮助概要并退出。
    </td>
  </tr></tbody> </table>

  <p>
    以上是通用的参数说明，可以根据需求在执行命令的时候添加。
  </p>

  <h3>
    服务端参数
  </h3>

  <p>
    下边对服务端特有参数进行说明：
  </p>

  <table class="tableau">
    <tr>
      <th colspan="2">
        服务端特定选项
      </th>
    </tr>

    <tr>
      <th>
        命令行选项
      </th>

      <th>
        说明
      </th>
    </tr>

    <tr>
      <td id="3server">
        **-s**, --server
      </td>

      <td>
        在服务器模式下运行 iPerf。 （这一次只允许一个 iperf 连接）
      </td>
    </tr>

    <tr>
      <td id="3daemon">
        **-D**, --daemon
      </td>

      <td>
        服务器作为守护进程在后台运行。
      </td>
    </tr>

    <tr>
      <td id="3pidfile">
        **-I**, --pidfile***file***
      </td>

      <td>
        写入一个包含进程 ID 的文件，这在作为守护进程运行时最有用。 (iPerf 3.1新特性)
      </td>
    </tr>
  </table>

  <p>
    服务端的命令选项相对来说少一些。
  </p>

  <h3>
    客户端参数
  </h3>

  <p>
    下边对客户端特有参数进行说明：
  </p>

  <table class="tableau">
    <tr>
      <th colspan="2">
        客户端特定选项
      </th>
    </tr>

    <tr>
      <th>
        命令行选项
      </th>

      <th>
        说明
      </th>
    </tr>

    <tr>
      <td id="3client">
        **-c**, --client ***host***
      </td>

      <td>
        在客户端模式下运行 iPerf，连接到主机上运行的 iPerf 服务器。
      </td>
    </tr>

    <tr>
      <td id="3sctp">
        **--sctp**
      </td>

      <td>
        使用 SCTP 而不是 TCP（Linux、FreeBSD 和 Solaris）。 (iPerf 3.1新特性)
      </td>
    </tr>

    <tr>
      <td id="3udp">
        **-u**, --udp
      </td>

      <td>
        使用 UDP 而不是 TCP。 另请参阅 -b 选项。
      </td>
    </tr>

    <tr>
      <td id="3bandwidth">
        **-b**, --bandwidth ***n[KM]***
      </td>

      <td>
        将目标带宽设置为 n 位/秒（UDP 默认为 1 Mbit/秒，TCP 无限制）。 如果有多个流（-P 标志），则带宽限制将单独应用于每个流。 您还可以向带宽说明符添加“/”和数字。 这称为“突发模式”。 它将发送给定数量的数据包而不会暂停，即使暂时超出了指定的带宽限制。
      </td>
    </tr>

    <tr>
      <td id="3time">
        **-t**, --time ***n***
      </td>

      <td>
        传输时间（以秒为单位）。 iPerf 通常通过在 time 秒内重复发送 len 个字节的数组来工作。 默认值为 10 秒。 另请参见 -l、-k 和 -n 选项。
      </td>
    </tr>

    <tr>
      <td id="3num">
        **-n**, --num ***n[KM]***
      </td>

      <td>
        要传输的缓冲区数量。 通常，iPerf 发送 10 秒。 -n 选项会覆盖此设置并发送 len 个字节的数组 num 次，无论需要多长时间。 另请参见 -l、-k 和 -t 选项。
      </td>
    </tr>

    <tr>
      <td id="3blockcount">
        **-k**, --blockcount ***n[KM]***
      </td>

      <td>
        要传输的块（数据包）的数量。 （而不是 -t 或 -n）另请参阅 -t、-l 和 -n 选项。
      </td>
    </tr>

    <tr>
      <td id="3len">
        **-l**, --length ***n[KM]***
      </td>

      <td>
        要读取或写入的缓冲区的长度。 iPerf 的工作原理是多次写入 len 个字节的数组。 TCP 默认为 128 KB，UDP 默认为 8 KB。 另请参见 -n、-k 和 -t 选项。
      </td>
    </tr>

    <tr>
      <td id="3parallel">
        **-P**, --parallel ***n***
      </td>

      <td>
        与服务器建立的同时连接数。 默认值为 1。
      </td>
    </tr>

    <tr>
      <td id="3reverse">
        **-R**, --reverse
      </td>

      <td>
        以反向模式运行（服务器发送，客户端接收）。
      </td>
    </tr>

    <tr>
      <td id="3window">
        **-w**, --window ***n[KM]***
      </td>

      <td>
        将套接字缓冲区大小设置为指定值。 对于 TCP，这设置 TCP 窗口大小。 （这会被发送到服务器并在该端使用）
      </td>
    </tr>

    <tr>
      <td id="3mss">
        **-M**, --set-mss ***n***
      </td>

      <td>
        尝试设置 TCP 最大分段大小 (MSS)。 MSS 通常是 TCP/IP 标头的 MTU - 40 字节。 对于以太网，MSS 为 1460 字节（1500 字节 MTU）。
      </td>
    </tr>

    <tr>
      <td id="3nodelay">
        **-N**, --no-delay
      </td>

      <td>
        设置 TCP 无延迟选项，禁用 Nagle 算法。 通常，只有交互式应用程序（如 telnet）才禁用此功能。
      </td>
    </tr>

    <tr>
      <td id="3version4">
        **-4**, --version4
      </td>

      <td>
        仅使用 IPv4。
      </td>
    </tr>

    <tr>
      <td id="3version6">
        **-6**, --version4
      </td>

      <td>
        仅使用 IPv6。
      </td>
    </tr>

    <tr>
      <td id="3tos">
        **-S**, --tos ***n***
      </td>

      <td>
        传出数据包的服务类型。 （许多路由器忽略 TOS 字段。）您可以指定带有“0x”前缀的十六进制值、带有“0”前缀的八进制值或十进制值。 例如，十六进制“0x10”= 八进制“020”= 十进制“16”。 RFC 1349 中指定的 TOS 编号为：</p>

        <pre>    IPTOS_LOWDELAY     minimize delay        0x10
    IPTOS_THROUGHPUT   maximize throughput   0x08
    IPTOS_RELIABILITY  maximize reliability  0x04
    IPTOS_LOWCOST      minimize cost         0x02</pre>
</td>
</tr>

<tr>
  <td id="3flowlabel">
    **-L**, --flowlabel ***n***
  </td>

  <td>
    设置 IPv6 流标签（目前仅在 Linux 上支持）。
  </td>
</tr>

<tr>
  <td id="3zerocopy">
    **-Z**, --zerocopy
  </td>

  <td>
    使用“零复制”方法发送数据，例如 sendfile(2)，而不是通常的 write(2)。 这会使用更少的 CPU。
  </td>
</tr>

<tr>
  <td id="3omit">
    **-O**, --omit ***n***
  </td>

  <td>
    省略测试的前 n 秒，以跳过 TCP TCP 慢启动期。
  </td>
</tr>

<tr>
  <td id="3title">
    **-T, --title *str***
  </td>

  <td>
    使用此字符串作为每个输出行的前缀。
  </td>
</tr>

<tr>
  <td id="3congestion">
    **-C**, --linux-congestion ***algo***
  </td>

  <td>
    设置拥塞控制算法（Linux 仅适用于 iPerf 3.0，Linux 和 FreeBSD 适用于 iPerf 3.1）。
  </td>
</tr>
</table>

<p>
  以上就是服务端和客户端的所有参数说明。
</p>

<p>

</p>
