# OSPF协议详解

## OSPF理论篇

### </br>

#### 写在前面

OSPF是一个动态路由协议，作为链路状态路由协议被广泛应用于网络中。

+ **为什么需要动态路由协议**
静态路由无法适应规模较大的网络，随着设备数量增加，配置量急剧增加。无法动态响应
网络变化。网络发生变化，无法自动收敛网络，需要工程师手动修改。

+ **为什么不使用距离矢量路由协议**
路由器并不清楚网络的拓扑，只是简单的知道要去往某个目的方向在哪里，距离有多远。
这就是距离矢量算法的本质。

#### OSPF-链路状态路由协议

+ **链路状态路由协议-LSA泛洪**
运行链路状态协议的路由器之间时首先会建立一个协议的邻居关系，然后彼此开
始交互LSA（Link State Advertisement，链路状态通告）。一类LSA将会被发送
给区域内每一台路由器。LSA描述了路由器接口的状态信息，例如接口的开销，连
接的对象等。

![图片1](/源库/OSPF/LSA泛洪.png)

LSA类型
Router-LSA(Type-1)
    每个设备都会产生，描述了设备的链路状态和开销，在所属的区域内传播。
Network-LSA（Type-2）
    由DR（Designated Router）产生，描述本网段的链路状态，在所属的区域内传播。
Network-summary-LSA(Type-3)
    由ABR产生，描述区域内某个网段的路由，并通告给发布或接收此LSA的非Totally Stub区
    或NSSA区域。例如：ABR同时属于Area0和Area1，Area0内存在网段10.1.1.0，Area1内
    存在网段11.1.1.0，ABR为Area0生成到网段11.1.1.0的Type3 LSA；ABR为Area1生成到网
    段10.1.1.0的Type3 LSA，并通告给发布或接收此LSA的非Totally Stub区或NSSA区域。
ASBR-summary-LSA（Type-4）
    由ABR产生，描述到ASBR的路由，通告给除ASBR所在区域的其他相关区域。
AS-external-LSA（Type-5）
     由ASBR产生，描述到AS外部的路由，通告到所有的区域（除了STUB区域和NSSA区域）
NSSA-LSA（Type-6）
     由ASBR产生，描述到AS外部的路由，仅在NSSA区域内传播。

+ **链路状态路由协议-LSDB组建**
路由器将LSA存放在LSDB（Link State Database，链路状态数据库）中，LSDB汇总
了网络中路由器对自己接口的描述，同时包含全网拓扑的描述。路由器基于此掌握全
网拓扑。

+ **链路状态路由协议-SPF计算**

![图片2](/源库/OSPF/SPF计算.png)

+ **链路状态路由协议-路由表生成**
+ 每台路由器基于LSDB，使用SPF（Shortest Path First，最短路径优先）算法进行计算。
在当前最新网路环境中，使用ISPF（增量型最短路径优先算法）+PRC（部分路由计算）
算法。前者用于解决拓扑结构发生改变，后者用于解决路由信息发生改变。每台路由器
计算出以自己为根的，无环的，拥有最短路径的”树“，了解到达网络各个角落的优选路径。
最后，将计算出来的最优路径，加载进自己的路由表。

#### OSPF-基础术语

+ **基础术语-区域**
OSPFArea 用于标识一个OSPF的区域。
区域是从逻辑上将设备划分成不同的组，每个组用区域号（Area ID）来标识。

+ **基础术语-router-id**
用于在一个OSPF域中唯一地标识一台路由器，可以通过手工配置的方式，或使
用系统自动配置的方式。

+ **基础术语-度量值**
OSPF使用cost(开销)作为路由的度量值，每一个激活了OSPF的接口都会维护一个
接口的Cost值缺省时接口Cost值=100Mbit/s/接口带宽。该值是可配置的。
一条OSPF路由的Cost值可以理解为是从目的网段到本路由器沿途所有入接口的Cost值累加。

#### OSPF-报文类型

![图片3](/源库/OSPF/报文类型.png)

#### OSPF-三大表项

+ **三大表项-邻居表**
OSPF在传递链路状态信息前，需先建立OSPF邻居关系，OSPF的邻居关系通过交互hello
报文建立。OSPF邻居表显示了OSPF路由器之间的邻居状态。
如果路由器之间只交互了hello报文，称为邻居状态，显示为2-way。
交互完所有报文之后，称为邻接状态，显示为Full。

+ **三大表项-LSDB表**
LSDB会保存自己产生的及从邻居收到的LSA信息
Type标识LSA的类型，AdvRouter标识发送LSA的路由器
使用命令行dispaly ospf lsdb查看LSDB表(华为命令)。

+ **三大表项-路由表**
OSPF路由表和路由器路由表是两张不同的表项。
OSPF路由表包含Destination、Cost、NextHop等指导转发的信息。

#### OSPF-邻接建立

+ **OSPF路由器之间的关系**
关于OSPF有两个重要的概念，邻居关系和邻接关系。
在两台路由器激活OSPF,通过Hello报文发现彼此之后，这两台路由器便形成了邻居关系。
当着两台路由器LSDB同步完成，并开始独立计算路由时，这两台路由器形成了邻接关系。

+ **OSPF邻接关系建立过程**
  + 接收到hello报文，邻接关系为Init.
  + 发送hello报文，并收到其它路由器的hello报文，邻接关系为2-way.
  + 交互三个DBD，完成主从的选举，邻接关系为Ex-start
  + 主路由器和从路由器进行LSDB数据库交互，邻接关系为Exchange.
  + 进行LSDB数据库的同步，邻接关系为loading.
  + 当完成LSDB数据库的同步，邻接关系变为Full.

#### OSPF网络类型

OSPF网络类型是一个非常重要的接口变量，这个变量将影响OSPF在接口上的操作，例如采用什么方式发送OSPF协议报文，以及是否需要选举DR、BDR等。

OSPF有四种网络类型：

1. Broadcast:广播式多路访问，默认接口为以太网接口.
2. NBMA:非广播多路访问，默认接口为帧中继.
3. P2P:点到点，默认接口为serial.
4. P2MP:点到多点，没有默认接口.一般用于vpn

两台路由器是否选举DR/BDR和接口没有关系，由网络类型决定。
在NBMA和Broadcast中,DRother之间不会邻居关系停留在2-way.
在P2P中，并不会进行DR/BDR选举.

#### DR与BDR

+ **为什么需要DR/BDR**
在MA网络中，如果每台OSPf路由器都与其他所有路由器建立OSPF邻接关系，便会导致网络中存在过多的OSPF邻接关系，增加设备负担，也增加网络中泛洪的OSPF报文数量。

+ **DR/BDR**
OSPF指定了三种路由器身份:DR(指定路由器),BDR(备用指定路由器)和DRother路由器.
只允许DR、BDR与其它OSPF路由器建立邻接关系.DRother之间不会建立全毗邻的邻接关系，双方停滞在2-way状态.
BDR会监控DR，并在他故障时取代他的角色.

#### OSPF多区域

如果OSPF仅有一个域，随着网络规模越来越大，OSPF路由器的数量越来越多，整个网络将不堪重负.
在单区域中，如果出现链路翻动(flapping),整个区域所有路由器都会针对链路每一次状态的变化去发生改变，网络的稳定性会比较差。
在多区域中，可以在ABR上进行区域间的路由汇总，一个区域内的链路翻动，对于被汇总的路由没有影响。
OSPF多区域的设计减小了LSA泛洪的范围，有效的把拓扑变化的影响控制在区域内，达到网络优化的目的.在区域边界进行路由汇总，可以减小路由表规模.多区域提高了网络拓展性，有利于组建大规模的网络.

#### OSPF路由器类型

   1.区域内路由器（Internal Router）
该类设备的所有接口都属于同一个ospf区域
   2.区域边界路由器（Area Border Router）
该类设备可以同时属于两个以上的区域，但其中一个必须是骨干区域。
ABR用来连接骨干区域和非骨干区域，它与骨干区域之间既可以是物理连接也可以是逻辑上的链接
   3.骨干路由器（Backbone Router）
该类设备至少有一个接口属于骨干区域
所有的ABR和位于Area0的内部设备都是骨干路由器
   4.自治系统边界路由器（AS Boundary Router）
与其他AS交换路由信息的设备称为ASBR。
ASBR不一定位于AS的边界，它可能是区域内设备，也可能是ABR。只要一台ospf设备引入了为外部路由的信息，他就成为ASBR。

在OSPF中，所有非骨干区域必须和骨干区域挂靠,否则那个区域将无法学习到其它区域的任何路由.这是出于星型拓扑的防环路设计。如果将一个区域看作一个路由器，那么区域之间的连接是一个星型拓扑，物理层面无环路.
那么为什么链路状态路由协议保证无环路还需要星型拓扑呢
链路状态路由协议的无环建立在区域内传递路由.一个区域内路由器所发送的一类LSA只能在那个区域内传播，而不同区域之间想要交互路由信息必须通过ABR，ABR可以同时收到两个区域的一类和二类LSA,根据两个区域生成不同的LSDB，并且发送三类LSA使两个区域路由完成交互.一个区域内的路由器想访问去往另一个区域的路由，只要将ABR作为下一跳.因此，多区域内的路由交换需要使用星型拓扑来保证无环，所以所有非骨干区域必须和骨干区域挂靠。

#### OSPF基础配置

**以下是华为配置**
</br>
![基础配置1](/源库/OSPF/基础配置1.png)

![基础配置2](/源库/OSPF/基础配置2.png)

**以下是神州数码配置**
</br>

```shell

OSPF协议
router ospf 100                                                 //创建ospf进程为100
router-id {id号}                                                //将本路由器router-id配置为指定id号
network {ip地址所在网段} {反掩码} area 0                         //将直连网段发布进ospf
ip ospf passive                                                 //禁用路由更新

OSPF报文
接口模式下
ip ospf network point-to-point                                //设置接口网络类型加快网络收敛
ip ospf hello-interval 5                                      //发送hello报文的时间间隔
ip ospf dead-interval 15                                      //对端邻居失效时间

明文认证
interface fastEthernet {端口号}                               //进入相邻接口
ip ospf authentication                                       //开启明文认证
ip ospf authentication-key 0 network                         //零表示不加密，设置密钥为network
show ip ospf neighbor                                       //查看邻接关系

密文认证
ip ospf authentication message-digest                     //开启接口密文认证
ip ospf message-digest-key 1 md5 network                 //密钥编号1，方式MD5，密钥为network

路由区域汇总
router ospf 100 
area {序列号} range {聚合后网段} {子网掩码}

区域明文认证
router ospf 100
area 0 authentication                                                     
interface fastEthernet 1/0
ip ospf authentication-key 0 network                    //开启area0的区域验证，0为不加密，密钥network

区域密文认证
router ospf 100
area 1 authentication message-digest
interface fastEthernet 1/0
ip ospf message-digest-key 1 md5 network               //开启area0的密文认证，密钥编号1，方式MD5，密钥network

虚连接
router ospf 100
area {序列} virtual-link {router-id}                  //通过区域使两台区域边界交换机互指，area要同一个

末节区（stub）
router ospf 1003
area {序列} stub                                   //修改区域为末节区
area {序列} stub no-summary                        //修改区域为完全末节区

路由重分发
router ospf 100
redistribute rip metric 200 subnets                 //将rip网络路由重分发到ospf网络，指定度量200
router rip
redistribute ospf 100 metric 10                     //将ospf网络路由重分发到rip网络，指定度量为10
神州数码需要route-map
route-map {}
set metric-type type-1                                         //分发为E1
                type-2                                         //分发为E2
set metric {}                                                  //更改开销
注意：在无ospf的区域，静态路由要配置回程路由
```
