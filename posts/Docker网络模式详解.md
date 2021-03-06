Docker网络模式介绍
==

当前的Docker提供了四种可用的网络模式，这四种网络功能还是比较简单，不能满足很多复杂的应用场景，大多数的应用场景下都没有直接用应这几种模试，而是通过一些工具，如pipework、weave配置自己的网络环境以满足更高的网络需求。要做到这些就要求使用者要深入了解Docker的网络知识，了解Docker容器是如何进行网络通信的。本文的目地是讲明白bridge网络模式下是如何进行网络通信的，理解了这些有利于我们自定义Docker使用的IP地址、DNS等信息，甚至使用自己定义的网桥，其工作方式都是一样的。

#1. Docker的4种网络模式

用docker run创建docker容器时，可以通过--net选项指定容器的网络模式，Docker有以下4种网络模式：

- bridge模式，使用--net=bridge指定，是docker默认设置。bridge是docker基础的网络模型，这里着重介绍此模式。
- host模式，使用--net=host指定。容器使用宿主机的IP和端口。带来的一个问题是若同一种类型的应用在docker服务器上跑多个实例容易发生端口冲突，管理应用的端口是个麻烦的事情。
- container模式，使用--net=container:NAME_or_ID指定。使用场景不多，两个容器必须跑在同一台docker服务器上。
- none模式，使用--net=none指定。在这种模式下，Docker容器拥有自己的Network，但没有网卡、IP、路由等信息。需要我们自己为Docker容器添加网卡、配置IP等。


#2. bridge模式的网络结构

当Docker server启动时，会在主机上创建一个名为docker0（可以在主机上使用ifconfig命令看到docker0）的虚拟网桥，虚拟网桥的工作方式和物理交换机类似，此主机上启动的Docker容器会连接到这个虚拟网桥上。这样主机上的所有容器就通过交换机连在了一个二层网络中。接下来就要为容器分配IP了，Docker会从[RFC1918](http://tools.ietf.org/html/rfc1918)所定义的私有IP网段中，选择一个和宿主机不同的IP地址和子网分配给docker0，连接到docker0的容器就从这个子网中选择一个未占用的IP使用。如一般Docker会使用172.17.0.0/16这个网段，并将172.17.42.1/16分配给docker0网桥。那么，Docker容器，docker0，宿主机组成了如下的网络结构：

![](https://raw.githubusercontent.com/wplatform/blog/master/assets/docker002/0001.png) 



容器与docker0网桥的网络配置大概是这样子的：在主机上创建一对虚拟网卡veth pair设备。veth设备总是成对出现的，它们组成了一个数据的通道，数据从一个设备进入，就会从另一个设备出来。因此，veth设备常用来连接两个网络设备。
Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0。另一端放在主机中，以veth39341b8这样类似的名字命名，并将这个网络设备加入到docker0网桥中，可以通过brctl show命令查看。

![](https://raw.githubusercontent.com/wplatform/blog/master/assets/docker002/0002.png)

从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关。

#3. bridge模式下容器的通信

在bridge模式下，连在同一网桥上的容器可以相互通信（若出于安全考虑，也可以禁止它们之间通信，方法是在DOCKER_OPTS变量中设置--icc=false，这样只有使用--link才能使两个容器通信）。

容器也可以与外部通信，我们看一下主机上的Iptable规则，可以看到这么一条

```Bash

-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE

```

这条规则会将源地址为172.17.0.0/16的包（也就是从Docker容器产生的包），并且不是从docker0网卡发出的，进行源地址转换，转换成主机网卡的地址。这么说可能不太好理解，举一个例子说明一下。假设主机有一块网卡为eth0，IP地址为10.10.101.105/24，网关为10.10.101.254。从主机上一个IP为172.17.0.1/16的容器中ping百度（180.76.3.151）。IP包首先从容器发往自己的默认网关docker0，包到达docker0后，也就到达了主机上。然后会查询主机的路由表，发现包应该从主机的eth0发往主机的网关10.10.105.254/24。接着包会转发给eth0，并从eth0发出去（主机的ip_forward转发应该已经打开）。这时候，上面的Iptable规则就会起作用，对包做SNAT转换，将源地址换为eth0的地址。这样，在外界看来，这个包就是从10.10.101.105上发出来的，Docker容器对外是不可见的。

那么，外面的机器是如何访问Docker容器的服务呢？我们首先用下面命令创建一个含有web应用的容器，将容器的80端口映射到主机的80端口。

```Bash

docker run -d --name web -p 80:80 fmzhen/simpleweb

```

然后查看Iptable规则的变化，发现多了这样一条规则：

```Bash

-A DOCKER ! -i docker0 -p tcp -m tcp --dport 80 -j DNAT --to-destination 172.17.0.5:80

```
此条规则就是对主机eth0收到的目的端口为80的tcp流量进行DNAT转换，将流量发往172.17.0.5:80，也就是我们上面创建的Docker容器。所以，外界只需访问10.10.101.105:80就可以访问到容器中得服务。


#4. 总结

默认的, Docker使用的是基于主机的私有网络，所以，位于同一台Docker服务器上的容器相互之间进行通信这没问题，如果要与其它Docker主机上的容间进行能信，他们必须在宿主机IP地址上分配端口（expose的方式），然后通过宿主机把数据包转发或代理的给容器。这种方式不好，因为为各式各式的应用协调端口是件很难的事情，这对云计算的平台太重要了。很多时候我们想让不同宿主机上的容器能相互通信，我们可以自定义Docker使用的IP地址、DNS等信息，甚至使用自己定义的网桥，工作方式还是一样的。
