
---
title: 网络基础知识
---


## 分层模型 OSI model

**Open Systems Interconnection model** (**OSI model**) 开放系统互联模型是一个概念性的参考模型, 不指定任何协议的具体实现. 核心思想在于不同网络层级间的解耦、接口独立, 任何一层协议可以升级替换后不影响整个网络功能, 在网络通信协议设计时起到一个指导作用.  
<img src="osi7.png" alt="image-osi" style="zoom:50%;" />

### Definitions:

+ 通信协议(Communication protocols): 让处于同层的不同host实体可以相互交互.

+ Protocol data unit(PDU): 在每一层, 两个实体通过交换该层协议的PDU来进行交互, PDU包含payload(也称为[service data unit](https://en.wikipedia.org/wiki/Service_data_unit) ,SDU) 和 protocol-related headers or footers . 

+  数据通信流程: 从Layer highest开始, PDU向下层传递, 并作为下层的SDU; 重复向更下层传递直至Layer lowest; 接着,底层负责将PDU传递给对端底层; 然后,逐层向上传递,拆包解析出各层SDU.

<img src="./osi_pdu_pass.png" alt="image-20220119105145560" style="zoom:40%;" />

<img src="osi_transfer_example.png" alt="image-20220123111825642" style="zoom:43%;" />

### Layer architecture

#### Layer 1: Physical layer

物理层负责非结构化原始数据的传输和接收, between device和物理传输介质。它将raw digital bits转换成electrical电子,optical光学 或radio无线信号等. 物理层规范规格包含蓝牙、以太网和USB标准等.

Examples: Ethernet family 802.3 PHY(千兆宽带, 百兆宽带), 802.11 LAN & wireless(2.4Ghz, 5Ghz), BLE(Bluetooth Low Energy) PHY,   E1.

#### Layer 2: Data link layer

物理层的线路有传输介质与通信设备组成，比特流在传输介质上传输时肯定会存在误差错误的。这样就引入了数据链路层在物理层之上，采用差错检测、差错控制和流量控制等方法，向网络层提供高质量的数据传输服务。

数据链路层从网络层接受数据包, 并在将其发送到物理层之前, 将这些数据包封装成帧frame. 之后在两个相邻节点间(node to node)的链路上传送帧，每一帧包括**数据**和必要的**头尾信息**（如同步信息、地址信息、差错信息等）. 如果帧太大(超过MTU)的话，数据链路层通常会将大帧拆分为一个个的小帧，小帧能够使传输控制和错误检测更加高效。

#### Layer 3: Network layer

网络层的任务就是提供能在不同的网络间transfer packets的程序和方法. 网络中会有非常多的nodes, 每个node会拥有自己的访问地址, 网络层就是要提供一种节点间路由寻址方式能让packet准确的传递给destination. 网络层还会在packet在某些MTU较小链路层传输时, 对原包进行分片, 独立传输, 并在下一链路node重新装配.  网络层可以做数据错误校验, 可靠性保障的功能, 但并不necessary.

#### Layer 4: Transport layer

传输层负责将变长数据(variable-length data sequences)从源传输到目的地, 并提供保障QoS的方式. 

#### Layer 5: Session layer

会话层的主要功能是负责维护两个节点之间的传输联接，确保点到点传输不中断，以及管理数据交换等功能。会话层在应用进程中建立、管理和终止会话。

#### Layer 6: Presentation layer

表示层的主要功能是处理在两个通信系统中交换信息的表示方式，主要包括数据格式变化、[数据加密](https://baike.baidu.com/item/数据加密/11048982)与解密、[数据压缩](https://baike.baidu.com/item/数据压缩/5198909)与解压等

#### Layer 7: Application layer

该层直接面向用户，是OSI中的最高层。它的主要任务是为用户提供应用的接口，即提供不同计算机间的文件传送、访问与管理，电子邮件的内容处理，不同计算机通过网络交互访问的虚拟终端功能等.

### Protocol stack examples

#### BLE protocol stack

<img src="./ble_osi.png" alt="ble_osi" style="zoom:80%;" />

## 分层模型——TCP/IP模型及协议族

TCP/IP 模型将网络分为四层(算上物理层就是五层)，它不关注底层物理介质，主要关注终端之间的逻辑数据流转发。
TCP/IP 模型的核心是**网络层**和**传输层**，网络层解决网络之间的逻辑转发问题，传输层保证源端到目的端之间的可靠传输。最上层的**应用层**通过各种协议向终端用户提供业务应用。

### 物理层

任何协议栈都逃不脱最终帮其传输数据的物理层. 物理层传递的都是bit flow, 而以太网使用的都是曼彻斯特编码信号. 曼彻斯特(Manchester)编码的编码方法是把每个码元分成两个相等的间隔。 如图,码元1是前一个间隔为高电压后一个间隔为低电压，即位周期中心向下跳变表示1；码元0正好相反，位周期中心向上跳变表示0。这样保证了在每一个 码元中间出现一次电压的转换，接收端就利用这种电压的转换方便地把位同步信号提取出来.

<img src="./manchester.png" alt="image-20220125225252662" style="zoom:45%;" />

在链路层从物理层收到的bit流中提取数据时, 一个问题就是如何区分帧的开始.  Manchester + Preamble.

<img src="./preamble.png" alt="image-20220125230003591" style="zoom:40%;" />

### 数据链路层

数据链路层位于网络层之下, 这意味着它与网络层协议比如常用的IP协议是解耦的, 以**以太网数据链路层帧**为例, 在通信时, 其完全不关心payload中是IP协议报文或是其他协议报文, 而是完全通过帧首的源MAC、目标MAC地址进行寻址通信.

IEEE 802 规范下数据链路层向下还可细分为 logical link control (LLC) 层和 medium access control (MAC) 层。

<img src="./datalinklayer_sublayers.png" alt="dll_sublayes" style="zoom:40%;" />

LLC 层又叫做`逻辑控制链路`层，它主要用于数据传输，它充当网络层和数据链路层中的`媒体访问控制（MAC）`子层之间的接口。LLC 层的主要功能如下

- LLC 的主要功能是发送时在 MAC 层上多路复用协议，并在接收时同样地多路分解协议。
- LLC 提供跳到跳的流的差错控制，像是路由器和路由器之间这种相邻节点的数据传输称为 `一跳`。
- 它允许通过计算机网络进行多点通信。

MAC 层负责传输介质的流控制和多路复用，它的主要功能如下

- MAC 层为 LLC 和 OSI 网络的上层提供了物理层的抽象。
- MAC 层负责封装帧，以便通过物理介质进行传输。
- MAC 层负责解析源和目标地址。
- MAC 层还负责在冲突的情况下执行冲突解决并启动重传。
- MAC 层负责生成帧校验序列，从而有助于防止传输错误。

在 MAC 层中，有一个关键概念是 `MAC 地址`。MAC 地址主要用于识别数据链路中互联的节点. MAC 地址长 6 bytes (48 bit)，在使用`网卡(NIC)` 的情况下，MAC 地址一般都会烧入 ROM 中。因此，任何一个网卡的 MAC 地址都是唯一的。

<img src="/Users/talentxiet/Desktop/我的学习文档/网络基础/mac_addr.png" alt="mac_addr" style="zoom:28%;" />

链路层的差错控制通过增加差错编码来实现, 又可分为**检错编码**和**纠错编码**: 检错编码常见有奇偶校验码和循环冗余码; 纠错编码常见有海明码等.

流量(阻塞)控制是控制链路上的帧的发送速率，使接收方有足够的缓冲空间来接收帧，主要方法有两种，停止-等待协议和滑动窗口协议. 停止-等待协议是一种特殊的滑动窗口协议，相当于发送窗口和接收窗口大小为1的滑动窗口协议。

<img src="./datalink_timewindow.png" alt="image-20220121112137195" style="zoom:40%;" />

以太网上传输的链路层数据帧有两种格式：**Ethernet_II 和 IEEE802.3**。选择哪种格式由 TCP/IP 协议簇中的网络层决定。大多数的数据帧使用的是 Ethernet_II 格式. IEEE802.3 格式：Length字段值小于等于0x05DC. Ethernet_II 格式：Type 字段值大于等于0x0600. 以此来供链路层判断数据是哪种规范.

<img src="./data_link_layer_frame.png" alt="image-20220119143457079" style="zoom:45%;" />

FCS: frame check sequence, 错误检查的.
LLC：Logical Link Control，逻辑链路控制由目的访问点 D.SAP （Destination Service Access Point）、源服务访问点 S.SAP（Source Service Access Point）和 Control 字段组成.
SNAP：Sub-network Access Protocol，子网访问协议由机构代码（Org Code）和类型（Type）字段组成。Org Code 三个字节都为 0 。Type 字段的含义与 Ethernet_II 帧中的 Type 字段相同.

### 网络层

#### **IP协议**

- 对于网络层，**IP协议**是其中一个非常重要的协议
- IP协议使得复杂的实际网络变为一个**虚拟互联的网络**（也就是我们只需要将终端设备连接到这个网络中去即可，并不需要关心里边实际的复杂网络）
- IP协议使得网络层可以**屏蔽底层细节**而专注网络层的**数据转发**（如果单从网络层去看，我们是不用关心**数据包**是通过海底电缆还是通过无线WiFi传输到目的计算机的）
- IP协议**解决了在虚拟网络中数据报传输路径的问题**

<img src="./ip_nets.png" alt="image-20220125231347009" style="zoom:40%;" />

**IP协议特点:**

- 无状态, 是指IP通信双方不同步传输数据的状态信息，因此所有IP数据报的发送，传输，和接收都是相互独立的，没有山下文关系的。发送、传输和接收都是相互独立、没有上下文关系的。这种服务最大的缺点是无法处理乱序和重复的IP数据报。比如发送端发送出的第N个IP数据报可能比第N+1个IP数据报后到达接收端，而同一个IP数据报也可能经过不同的路径多次到达接收端。在这两种情况下，接收端的IP模块无法检测到乱序和重复，因为这些IP数据报之间没有任何上下文关系。接收端的IP模块只要收到了完整的IP数据报(如果是IP分片的话，IP 模块将先执行重组)，就将其数据部分(TCP报文段、UDP数据报或者ICMP报文)，上交给上层协议。那么从上层协议来看，这些数据就可能是乱序的、重复的。面向连接的协议，比如TCP协议，则能够自己处理乱序的、重复的报文段，它递交给上层协议的内容绝对是有序的、正确的。虽然IP数据报头部提供了一个标识字段(见后文)用以唯-标识-一个IP数据报，但它是被用来处理IP分片和重组的，而不是用来指示接收顺序的。无状态服务的优点也很明显:简单、高效。我们无须为保持通信的状态而分配一些内核资源，也无须每次传输数据时都携带状态信息。在网络协议中，无状态是很常见的，比如UDP协议和HTTP协议都是无状态协议。以HTTP协议为例，一个浏览器的连续两次网页请求之间没有任何关联，它们将被Web服务器独立地处理。
- 无连接(connectionless), 是指IP通信双方都不长久地维持对方的任何信息。这样，上层协议每次发送数据的时候，都必须明确指定对方的IP地址。
- 不可靠, 是指IP协议不能保证IP数据报准确地到达接收端，它只是承诺尽最大努力(best effort)。 很多种情况都能导致IP数据报发送失败。比如，某个中转路由器发现IP数据报在网络上存活的时间太长(根据IP数据报头部字段TTL判断，见后文)，那么它将丢弃之，并返回一个ICMP错误消息(超时错误)给发送端。又比如，接收端发现收到的IP数据报不正确(通过校验机制)，它也将丢弃之，并返回一个ICMP错误消息(IP 头部参数错误)给发送端。无论哪种情况，发送端的IP模块一旦检测到IP数据报发送失败，就通知上层协议发送失败，而不会试图重传。因此，使用IP服务的上层协议(比如TCP协议)需要自己实现数据确认、超时重传等机制以达到可靠传输的目的。

**IP协议报文格式 :**

<img src="./ip_packet.png" alt="image-20220125234059378" style="zoom:40%;" />

- **版本**：占4位，指的是IP协议的版本，通信双方的版本必须一致，即IPv4，也有IPv6

- **首部长度**：占4位，最大数值为15，表示的是IP首部的长度，单位是“32位字”（4个字节），也就是**IP首部**最大长度为60字节

- **服务类型**：服务登记, 比如VIP专线, 这个一般是不需要关心的

- **总长度**：占16位，最大数值为65535，表示的是**IP数据报总长度**（IP首部+IP数据） （在前边介绍数据链路层的时候，也提到过一个长度。对于数据链路层的长度，称之为**MTU**，一般为1500字节。而IP数据报的最大长度有65535个字节，比MTU要大。如果真正传输的时候，如果出现这种情况，数据链路层会对IP数据报进行**分片**，也就是将一个较长的IP数据报拆分成多个数据帧来进行传输）

- **标识**：占16位，计数器标示, 分片后的同组数据具备相同标示. 同一组的包根据标示重组.

- **标记**： 占3位，但目前只有2位有意义。
   标志字段中的最低位记为MF(More Fragment)。MF=1即表示后面“还有分片”的数据报,MF=0表示这已是若干数据报片中的最后一个
   标志字段中间的一位记为DF(Don’t Fragment)，意思是“不能分片”。只有当DF=0时才允许分片。
   
- **片偏移**：前边有提到，如果IP数据报的长度过长，会进行IP报文的分片，把一个IP报文拆分成多个数据帧进行数据链路层的传输(以**8 Bytes**的倍数分包)。因此，如果拆分的话，就需要使用**片偏移**来记录当前的数据帧，保存的**第几个偏移的IP数据**

   <img src="./ip_offset.png" alt="image-20220129104929487" style="zoom:45%;" />

- **TTL**：占8位，表明IP数据报文在网络中的**寿命**，每经过一个设备（不管是路由器还是计算机），TTL减一，当TTL=0时，网络设备必须**丢弃**该报文（它解决的就是，当网络报文找不到终点的时候，避免网络报文在网络中无限的传输，以消耗带宽）

- **协议**：占8位，表明IP数据所携带的具体数据是什么**协议**的（如TCP、UDP等，一些协议对应的值，可参考下图）

- **校验和**：占16位，校验IP首部是否有出错（接收方接收到IP首部之后也会进行校验，如果有错，则直接丢弃）

- **源IP地址**：**发送**IP数据报的网络设备的IP地址

- **目的IP地址**：IP数据报要**到达**的目的网络设备的IP地址

#### **ICMP协议**

<img src="./icmp_kinds.png" alt="image-20220126215905234" style="zoom:50%;" />

#### **路由选择协议**

**自治系统**：autonomous system。在互联网中，一个自治系统(AS)是一个有权自主地决定在本系统中应采用各种[路由协议](https://baike.baidu.com/item/路由协议/202634)的小型单位。这个网络单位可以是一个简单的网络也可以是一个由一个或多个普通的[网络管理员](https://baike.baidu.com/item/网络管理员/595848)来控制的网络群体，它是一个单独的可管理的[网络单元](https://baike.baidu.com/item/网络单元/9928606)（例如一所大学，一个企业或者一个公司个体）。

<img src="./ip_routing_protocol.png" alt="image-20220126220513526" style="zoom:50%;" />

内部网关协议:

+ RIP, Routing Information Protocol: 优点, 算法简单; 缺点, 路由故障链路改变远端感知慢, 距离有限 max16, 大规模网络路由表可能过大.

<img src="./rip.png" alt="image-20220126220918042" style="zoom:50%;" />

+ OSPF, Open Shortest Path First 开放的最短路径优先算法: 

  与RIP三个区别:（1）向本[自治系统](https://baike.baidu.com/item/自治系统)中所有路由器发送信息。（2）发送的信息就是与本[路由器](https://baike.baidu.com/item/路由器)相邻的所有路由器的链路状态，但这只是路由器所知道的部分信息。（3）只有在链路状态发生变化时，[路由器](https://baike.baidu.com/item/路由器)才向所有路由器用[洪泛](https://baike.baidu.com/item/洪泛)法发送此信息。

外部(边界)网关协议:

BGP用于在不同的自治系统（AS）之间交换路由信息。当两个AS需要交换路由信息时，每个AS都必须指定一个运行BGP的节点，来代表AS与其他的AS交换路由信息。两个AS中利用BGP交换信息的路由器也被称为边界网关（Border Gateway）或边界路由器（Border Router）. 

<img src="./BGP_speaker.png" alt="image-20220126223533819" style="zoom:50%;" />

<img src="./BGP_example.png" alt="image-20220126223429235" style="zoom:40%;" />



### 传输层

端口作用, 实现进程到进程的通信, 复用ip线路, 提供标准的的端口->进程的映射方式.

<img src="./transport_process.png" alt="image-20220126230009234" style="zoom:50%;" />

#### UDP

UDP, User Datagram Protocol用户数据报协议, 只在IP基础上多了复用分用和差错校验的功能.

##### 特点

+ 无连接
+ 尽努力交付
+ 面向报文的
+ 无拥塞控制
+ 支持一对一、一对多、多对一、多对多
+ 首部开销小

##### 协议格式

<img src="./udp_protocol.png" alt="image-20220126233302042" style="zoom:45%;" />

#### TCP

##### 特点

+ 面向连接

+ 每条点对点(socket1, socket2)

+ 可靠的

+ 全双工

+ 面向字节流的

+ 为应用程序间提供逻辑通信链路

  <img src="./tcp_bytes_stream.png" alt="image-20220129001752910" style="zoom:50%;" />

##### 协议格式

<img src="./tcp_protocol.png" alt="image-20220127225910379" style="zoom:50%;" />

**16位源端口号**：指发送端的端口号
**16位目的端口号**：指目的端的端口号
**4位头部长度**：同IP头部，表示TCP头部的大小，以4字节为单位。
**32位序列号**：TCP通信过程中，通过序列号来保证传输过程中数据的有序性
**32位确认号**：用以对接受到的报文进行确认
**保留6位**
**URG**：表示紧急指针
**ACK**：表示确认号
**PSH**：通知对端立即从缓冲区取走数据
**RST**: 通知当前连接出现问题, 要释放资源, 有需要则重新建立连接. 也可用于拒绝非法连接. 
**SYN**：表示请求建立连接
**FIN**：标志要通知对端本端的数据发送要关闭
**16位窗口大小**：TCP流量控制的手段，告诉发送方本端的接收端缓冲区还能接受多少数据, 单位字节.
**16位校验和**：由发送方填充，接收端用以检查TCP报文在传输过程中是否有损坏

##### 可靠性

###### 滑动窗口

A根据B TCP ack报文中的窗口大小发送滑动窗口大小以内的若干数据给B, 给回复按序抵达的最大包序号, 即31. B不应过度拖延回复ack的时间.

<img src="./tcp_window.png" alt="image-20220128000255497" style="zoom:50%;" />

<img src="./tcp_buffer.png" alt="image-20220128000641845" style="zoom:50%;" />

需注意, tcp是全双工通信, 每端都有自己的发送和接受窗口及缓冲. 

###### 超时重传时间 Retransmision Time out

<img src="./tcp_rto.png" alt="image-20220128001439268" style="zoom:40%;" />

###### 选择确认ACK Selective ACK

SACK用于接收端向发送端报告部分未按序已收到的字节块, 从而避免重复传输. 需要tcp连接时 option增加SACK头, 双方确认支持SACK. 但是描述哪些数据已接受其实是比较复杂的, 尽管RFC2018有详细说明, 但使用还是较少, 大部分还是发送端重发所有未ack字节.

<img src="./tcp_sack.png" alt="image-20220128002718253" style="zoom:50%;" />



##### TCP流量控制

###### 可利用滑动窗口控制发送方流量



###### TCP传输效率

应用程序把发送数据给到TCP缓存就不管了, 由TCP负责控制发送时机将数据正确的发送出去. 这是一个比较复杂的问题.

+ **Nagle算法**

  TCP/IP协议中，无论发送多少数据，总是要在数据前面加上协议头，同时，对方接收到数据，也需要发送ACK表示确认。为了尽可能的利用网络带宽，TCP总是希望尽可能的发送足够大的数据。（一个连接会设置MSS参数，因此，TCP/IP希望每次都能够以MSS尺寸的数据块来发送数据）。Nagle算法就是为了尽可能发送大块数据，避免网络中充斥着许多小数据块。

  Nagle算法的基本定义是**任意时刻，最多只能有一个未被确认的小段**。 所谓“小段”，指的是小于MSS尺寸的数据块，所谓“未被确认”，是指一个数据块发送出去后，没有收到对方发送的ACK确认该数据已收到。

  Nagle算法的规则（可参考tcp_output.c文件里tcp_nagle_check函数注释）：

  （1）如果包长度达到MSS，则允许发送；

  （2）如果该包含有FIN，则允许发送；

  （3）设置了TCP_NODELAY选项，则允许发送；

  （4）未设置TCP_CORK选项时，若所有发出去的小数据包（包长度小于MSS）均被确认，则允许发送；

  （5）上述条件都未满足，但发生了超时（一般为200ms），则立即发送。

  ```bash
      if there is new data to send #有数据要发送
          # 发送窗口缓冲区和队列数据 >=mss，队列数据（available data）为原有的队列数据加上新到来的数据
          # 也就是说缓冲区数据超过mss大小，nagle算法尽可能发送足够大的数据包
          if the window size >= MSS and available data is >= MSS 
              send complete MSS segment now # 立即发送
          else
              if there is unconfirmed data still in the pipe # 前一次发送的包没有收到ack
                  # 将该包数据放入队列中，直到收到一个ack再发送缓冲区数据
                  enqueue data in the buffer until an acknowledge is received 
              else
                  send data immediately # 立即发送
              end if
          end if
      end if　
  ```

  Nagle算法只允许一个未被ACK的包存在于网络，它并不管包的大小，因此它事实上就是一个扩展的停-等协议，只不过它是基于包停-等的，而不是基于字节停-等的。

+ **TCP确认延迟机制**

  当Server端收到数据之后，它并不会马上向client端发送ACK，而是会将ACK的发送延迟一段时间（假设为t），它希望在t时间内server端会向client端发送应答数据，这样ACK就能够和应答数据一起发送，就像是应答数据**捎带着ACK**过去。初始化是40ms。

+ **TCP_NODELAY 选项**

  默认情况下，发送数据采用Nagle 算法。这样虽然提高了[网络吞吐量](https://baike.baidu.com/item/网络吞吐量/646450)，但是实时性却降低了，在一些交互性很强的应用程序来说是不允许的，使用TCP_NODELAY选项可以禁止Nagle 算法。

  此时，应用程序向内核递交的每个数据包都会立即发送出去。需要注意的是，虽然禁止了Nagle 算法，但网络的传输仍然受到TCP确认延迟机制的影响。

+ **TCP_CORK 选项**

  所谓的CORK就是塞子的意思，形象地理解就是用CORK将连接塞住，使得数据先不发出去，等到拔去塞子后再发出去。设置该选项后，内核会尽力把小数据包拼接成一个大的数据包（一个MTU）再发送出去，当然若一定时间后（一般为200ms，该值尚待确认），内核仍然没有组合成一个MTU时也必须发送现有的数据（不可能让数据一直等待吧）。

##### TCP拥塞控制

+ 拥塞控制：防止过多的数据注入到网络中，这样可以使网络中的路由器或链路不致过载。拥塞控制所要做的都有一个前提：网络能够承受现有的网络负荷。拥塞控制是一个全局性的过程，涉及到所有的主机、路由器，以及与降低网络传输性能有关的所有因素。

+ 流量控制：指点对点通信量的控制，是端到端的问题。流量控制所要做的就是抑制发送端发送的速率，以便使接收端来得及接收。

解决拥塞问题一般是以下步骤: 1) 监测网络, 探测拥塞的位置与时间;  2) 将拥塞信息发送到能采取控制的地方; 3) 调整网络参数.

常用方法: 慢开始( slow-start )、拥塞避免( congestion avoidance )、快重传( fast retransmit )和快恢复( fast recovery )。

###### 慢开始

发送方维持一个拥塞窗口 cwnd ( congestion window )的状态变量。拥塞窗口的大小取决于网络的拥塞程度，并且动态地在变化。发送方让自己的发送窗口等于拥塞窗口。发送方控制拥塞窗口的原则是：只要网络没有出现拥塞，拥塞窗口就再增大一些，以便把更多的分组发送出去。但只要网络出现拥塞，拥塞窗口就减小一些，以减少注入到网络中的分组数。

慢开始算法：当主机开始发送数据时，如果立即所大量数据字节注入到网络，那么就有可能引起网络拥塞，因为现在并不清楚网络的负荷情况。因此，较好的方法是先探测一下，即由小到大逐渐增大发送窗口，也就是说，由小到大逐渐增大拥塞窗口数值。通常在刚刚开始发送报文段时，先把拥塞窗口 cwnd 设置为一个最大报文段MSS的数值。而在每收到一个对新的报文段的确认后，把拥塞窗口增加至多一个MSS的数值。用这样的方法逐步增大发送方的拥塞窗口 cwnd ，可以使分组注入到网络的速率更加合理。 cwnd = 1, 2, 4, 8 ...

为了防止拥塞窗口cwnd增长过大引起网络拥塞，还需要设置一个慢开始门限ssthresh状态变量。慢开始门限ssthresh的用法如下：

　　当 cwnd < ssthresh 时，使用上述的慢开始算法
　　当 cwnd > ssthresh 时，停止使用慢开始算法而改用拥塞避免算法
　　当 cwnd = ssthresh 时，既可使用慢开始算法，也可使用拥塞控制避免算法

###### 拥塞避免

让拥塞窗口cwnd缓慢地增大，即每经过一个往返时间RTT就把发送方的拥塞窗口cwnd加1，而不是加倍。这样拥塞窗口cwnd按线性规律缓慢增长，比慢开始算法的拥塞窗口增长速率缓慢得多。

无论在慢开始阶段还是在拥塞避免阶段，只要发送方判断网络出现拥塞（其根据就是没有收到确认），就要把慢开始门限ssthresh设置为出现拥塞时的发送方窗口值的一半（但不能小于2）。然后把拥塞窗口cwnd重新设置为1，执行慢开始算法。这样做的目的就是要迅速减少主机发送到网络中的分组数，使得发生拥塞的路由器有足够时间把队列中积压的分组处理完毕。

<img src="./tcp_slow_start.png" alt="image-20220128190952845" style="zoom:40%;" />

###### 快重传和快恢复

快重传算法首先要求接收方每收到一个报文段后(即便是后序)就立即发出重复确认（为的是使发送方及早知道有报文段没有到达对方），而不要等到自己发送数据时才进行捎带确认。当发送方连续收到三个重复确认，就执行“乘法减小”算法，把慢开始门限ssthresh减半。在采用快恢复算法时，慢开始算法只是在TCP连接建立时和网络出现超时时才使用。采用这样的拥塞控制方法使得TCP的性能有明显的改进。

<img src="./tcp_congest.png" alt="image-20220128190034654" style="zoom:50%;" />

##### TCP连接管理

###### 三次握手

三次握手是为了建立连接, 双方确认对方的存在,并协商一些参数,如窗口大小 , 初始化一下缓冲队列资源等. 为了确认双方存在, 头两次sync, sync+ack是 省不了的, 那第三次ack是为了做什么呢? 是为了避免已失效的sync请求又抵达B, 又消耗B的资源. 有了第三次ack, 当无效的syn被server sync+ack后, client不会回复ack, 超时后server也就会再销毁掉对应资源.

<img src="./tcp_handshake.png" alt="image-20220129000032602" style="zoom:50%;" />

###### 四次挥手

<img src="./tcp_bye_handshake.png" alt="image-20220129002627713" style="zoom:50%;" />

**MSL**: Max Segment lifetime, 报文端最大寿命. 

为什么A收到 FIN 后要等两个MSL呢? 

1. 确保能回复FIN的ack, 如果B没收到ack, 会重新发送FIN+ACK. 这时如果A已经断了, B就只能自己特殊处理到CLOSED状态.
2. 尽量让本次连接 网络中的网络包都传输完, 不会在新连接中收到旧连接报文.

##### TCP有限状态机

<img src="./tcp_state.png" alt="image-20220129152318656" style="zoom:50%;" />



#### 应用层

## 应用层常用协议

### DNS

基于UDP, **53**端口:  query请求(src port: 12868 , dst port: 53 ) + response响应(src port: 53, dst port: 12868 )

迭代查询 or 递归查询 + 缓存. 

**解析顺序**: 浏览器缓存, 系统缓存, 路由器DNS缓存 , ISP DNS 缓存, 根域名服务器, 顶级域名服务器, 主域名服务器, 保存结果至缓存.

<img src="./dns.png" alt="image-20220129165409193" style="zoom:40%;" />

### HTTP

HTTP协议发展历程:

![image-20220127112311050](/Users/talentxiet/Library/Application Support/typora-user-images/image-20220127112311050.png)

协议格式:

<img src="./http_protocol.png" alt="image-20220130002354946" style="zoom:50%;" />

### SMTP

<img src="./smtp.png" alt="image-20220130002634963" style="zoom:45%;" />

### DHCP

<img src="./dhcp.png" alt="image-20220130221121647" style="zoom:50%;" />

## 应用程序间通信过程

### 系统调用&socket套接字

<img src="./syscall.png" alt="image-20220130221429374" style="zoom:50%;" />

<img src="./socket2.png" alt="image-20220130224749539" style="zoom:50%;" />



## 网络安全

四个目标:

+ 保密性
+ 端点鉴别
+ 信息完整性
+ 运行安全性 访问控制

<img src="./net_sec_key.png" alt="image-20220205185347484" style="zoom:50%;" />

### DES AES

data encryption standard,  advanced encrytion standard, 对称加密、

### 公钥密码体制

任何加解密算法的安全性都取决于密钥长度,以及攻破密文的计算量

### 数字签名

目标:

+ 接收者能核实发送者对报文的数字签名, 即是发送者发的
+ 收到的数据是完整的
+ 发送者事后不可抵赖数字签名

<img src="./signature.png" alt="image-20220205191312308" style="zoom:50%;" />

### 报文鉴别

加解密算法可以实现报文鉴别,但是消耗cpu, 所以常用密码散列算法保护数据完整性. MD5 (message digest 第五代)和 SHA-1,2,3(secure hash algorithm).

<img src="./sec_mac.png" alt="image-20220205193055077" style="zoom:50%;" />

MAC: message authentication code,

### 密钥分配

#### 对称密钥分配

对称密钥是一对一的, 任意双方通信都要有唯一的key, 通常由KDC动态随机分配

<img src="./对称密钥分配.png" alt="image-20220205195510847" style="zoom:50%;" />

#### 公钥分配

由受信任的机构维护公钥和实体的关系, 通常是政府机构, 称之为CA (certification authority), 每个实体会被颁布 证书(certificate), 证书中包含拥有者标示, 公钥, 且此证书被CA进行了数字签名. 公众可以从机构公开信息中获取到机构的公钥, 对签名和 证书信息进行验证.

为了使CA拥有统一格式, IETF接受了ITU制定的X.509规范.

### 互联网中的安全协议

#### 网络层

##### IPsec

<img src="./ipsec.png" alt="image-20220206000051358" style="zoom:50%;" />

#### 传输层

##### SSL

Secure Socket Layer, Netscape开发, 95年交给IETF

<img src="./ssl.png" alt="image-20220206001829432" style="zoom:50%;" />

<img src="./ssl_procedure.png" alt="image-20220206002408083" style="zoom:50%;" />

##### TLS

Transport Layer Security, 基于SSL3.0设计. 

### 防火墙

是一种访问控制技术, 通常是特殊编程的路由器, 进行访问策略控制. 

+ 分组过滤路由器: 根据 源/目的 IP/端口 协议类型进行分组过滤控制.
+ 应用网关: 应用层网关, 根据应用层报文鉴权转发

## 实时音视频相关协议

<img src="./av_protocols.png" alt="image-20220206165809797" style="zoom:50%;" />

### RTP

<img src="./rtp_protocol.png" alt="image-20220206165710831" style="zoom:50%;" />

### RTCP

<img src="./rtcp.png" alt="image-20220206172316751" style="zoom:50%;" />

## 常见网络设备

### Repeater中继器

中继器是一个将输入信号增强放大的设备, 把信号送得更远，以延展网络长度. 与具体信号形式无关,比如卫星雷达, 以太网集线器; 属于物理层

### Hub集线器 

集线器（Ethernet hub）是指将多条以太网双绞线或光纤集合连接在同一段物理介质下的设备。集线器运作在OSI模型中的物理层，可以让其链接的设备工作在同一网段。集线器上有多个I/O端口，信号从任意一个端口进入后，会从其他端口出现. 

由于集线器会把收到的任何数字信号，从集线器的所有端口提交，这会造成信号之间碰撞的机会很大，而且信号也可能被窃听，并且这代表所有连到集线器的设备，都是属于同一个冲突网域以及广播网域(hub间组网所有hub都属于同一广播域), 冲突和泛洪严重，因此大部分Hub已被Switch代替.

<img src="./hub_pic.png" alt="image-20220121154139634" style="zoom:35%;" />

### Bridge网桥

网桥工作在第二层数据链路层, 可以解析出链路层数据包, 并在Bridge内维护每个端口和连接着的设备MAC地址映射关系, 从而可以动态判断是否要转发收到的物理帧, 从而在一定程度上解决hub间的泛洪问题.

<img src="/Users/talentxiet/Desktop/我的学习文档/网络基础/bridge_pic.png" alt="image-20220121155228284" style="zoom:50%;" />

### Switch交换机

二层交换机: 二层交换机工作在数据链路层, 相比Hub, 它可以解析物理帧中的链路层frame信息,并基于frame中的mac地址与交换机端口进行缓存映射, 从而高效的解决Hub的冲突, 泛洪等问题. 此外交换机通常还支持划分VLAN的功能, VLAN可以创建相互隔离的子LAN, 从而缩小广播域大小. 

<img src="./switch_pic.png" alt="image-20220121160921808" style="zoom:55%;" />

交换机组网示例:

<img src="./switch_net.png" alt="image-20220122201304858" style="zoom:50%;" />

三层交换机: 三层交换机在交换机的基础之上加了网络层协议的解析能力, 以及一些简单的三层路由功能. 不过它的主要功能还是交换机的，同时它的价格比路由器价格偏低，实用性更高。

一个真实的交换机图片如下所示:

<img src="./real_switch.png" alt="image-20220122202630798" style="zoom:38%;" />

### Router路由器 (网关)

讲到路由器,我们要先讲一下IP子网和网关(Gateway), IP地址+掩码标示了IP网段, 不同网段间的网络相互隔离. 而网关, 顾名思义, 就是不同网络间连接的关口, 在网络层上实现网络连通. 网关概念也可延伸到应用服务网关, 这里我们讨论的均指TCP/IP协议下网关.

在TCP/IP协议的系统内核层, 会在网络层向链路层封包时, 检测目标IP与源IP是否在同一子网, 如果在同一子网, 封包时的对端MAC地址通过ARP协议获取; 而当不在同一子网时, 会使用网关的MAC作为目标MAC地址. 可以用下图思考一下, 不同子网间通信时的隔离性到底是如何实现的? 

<img src="./ip_group_question.png" alt="image-20220122220139210" style="zoom:40%;" />

不同的IP网段可以设置不同的网关地址, 从而细粒度的控制流量出口. 一个默认macos下的路由表配置如下, 该机器ip为192.168.0.108, default网关为192.168.0.1, 访问127.0.0.1时会loopback到自己网卡, 访问192.168.0/24网段时会在en0网卡所处的进行链路层进行二层数据通信. 

<img src="./routing_table.png" alt="image-20220122215006525" style="zoom:50%;" />

路由器实现了网关的功能, 它工作的三层网络层, 可以实现三层协议的解包路由等功能. 当然路由器还有很多其他功能, 比如NAT等. 

路由器上会有很多网口, 可以按用处分为WAN口和LAN口, WAN口指的是接入其他Wide Area Network广域网的网口, LAN口指的是接入路由器内部Local Area Network局域网的网口. 往往内部LAN还支持Wifi的接入, 成为Wireless LAN (WLAN).  每个WAN口均可以提供三层能力, 即可配置独立的MAC及IP地址(独立网卡); LAN口则往往只具备二层传输能力, 提供交换机的功能. 具备三层能力的LAN口可以通过路由器配置作为WAN口服务.

<img src="/Users/talentxiet/Desktop/我的学习文档/网络基础/router_wan_lan.png" alt="router_wan_lan" style="zoom:40%;" />

当路由器要串联多组子网时, 若想要子网间按期望进行通信的话, 就需要正确配置Router的路由表了. 路由表比较核心的三个配置属性是Destination, Gateway 和 Interface. 

Destination可以是网段, Host或者default (均不匹配时), 路由器网卡收到frame并解出目标IP后, 会逐一进行匹配, 匹配到对应记录后, 会按路由表配置, 将frame的目标MAC地址转换为所配网关的MAC地址, 并通过指定配置的Interface发送出去. 当没配置网关时, 会在Interface广播域对目标IP进行广播寻址.

<img src="/Users/talentxiet/Desktop/我的学习文档/网络基础/router_trans.png" alt="router_transfer" style="zoom:36%;" />

<img src="./router_trans2.png" alt="image-20220124222512944" style="zoom:50%;" />

一个真实企业级路由器的图片如下: 

<img src="/Users/talentxiet/Desktop/我的学习文档/网络基础/real_router.png" alt="image-20220122203856676" style="zoom:45%;" />

### F5

F5是一家美国公司, 卖的F5设备可提供4-7层的解包和路由能力, 相比工作于网络层的Router其功能就更丰富了. 包括负载均衡，应用状态监控，高可用性保障，广域流量管理，链路控制；内容转换，高速缓存，SSL加速和卸载等能力等等.

<img src="./real_f5.png" alt="image-20220122205509626" style="zoom:60%;" />

## 真实组网案例

### 公共网络



### 家庭网络



## Glossary:

### Ethernet 以太网

以太网(Ethernet)是一种计算机局域网技术, 提出于1980s。IEEE组织的IEEE 802.3标准制定了以太网的技术标准, 后续延伸出了802.3~11标准，它规定了包括物理层的连线、电子信号和介质访问层协议的内容。以太网是当前应用最普遍的局域网技术，取代了其他局域网标准如令牌环、FDDI 和 ARCNET。

### 广播域 broadcast domain

广播域（Broadcast domain）是计算机网络的一个逻辑划分。广播域中的任意一个节点可以在数据链路层通过广播的方式到达任意一个节点。广播域可以处于同一个局域网内,也可以被桥接到其他的局域网。

根据目前的流行技术，任意连接到同一 repeater 或者 switch 的电脑属于同一个广播域，并且任意连接到同一个inter-connected的 repeater 或 switch 的集合的电脑也是属于同一个广播域的。而 Router 和其他high-level devices会在广播域间形成隔离。

### 冲突域 collision domain

与广播域相对的是冲突域。冲突域中所有节点都链接到同一个被集线器、交换机和学习型网桥划分的相互连接的中继器集合。冲突域一般来说小于或者包含在广播域中。 冲突域下的碰撞问题通过CSMA/CD (Carrier-sense multiple access)解决.



## 常用命令

### traceroute实现

向目标ip发送ICMP echo包或随机30000+大端口发送udp包,  通过TTL和ICMP结果返回报文判断路由点地址和通信时间.

