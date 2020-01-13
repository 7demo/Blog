# 计算机网络概述

### 概述

局域网：覆盖范围小，自己维护、带宽固定（10m、100m、1000m）

广域网：距离远，专业服务、花钱租带宽

交换机：接入层交换机-汇聚层交换机，电脑到交换机不超过100m，交换机到另外交换机不超过100m。

ISP: 运营商

不同运营商：双机房、双线机房

internet: ISP自己机房、对网民提供网络连接

网关（一般是路由器）：负责转发数据，局域网其他机器的数据转到此并转出。掩码几个255表示ip中有前几个为网络。

网络请求发送时数据时，每个数据包包含: 数据、源ip、目标ip、源mac地址、目标mac地址。数据、源ip、目标ip称为数据包，如果再加上mac地址，就称为数据帧。在传递过程中，经过网关，会更改mac地址。

发送数据时，先放缓存，再发，发送成功后才可以删除，否则可能重发。接收数据时，也有缓存，

### OSI模型

应用层：能产生流量的程序。

表示层：在传输中是否进行加密或者压缩

会话层：来确保传输的数据时针对的请求

传输层：可靠传输、流量控制。（查询dns时，是udp通讯，是不可靠传输）

网络层：负责选择最佳路径、规划ip地址——》配置错误ip地址、子网掩码、错误网关

数据链层：帧的开始与结束、透明传输、差错校验——》mac地址冲突、ADSL、网络速度协定

物理层：定义网络设备标准、电器标准——》查看连接状态、发送接收数据包

![avatar](./images/computer/1.jpg)

![avatar](./images/computer/2.jpg)

### 性能

速率：信道中速度。

带宽：最高支持。

吞吐量：总共大小。

时延：发送时延（数据块大小/信道带宽）、传播时延、处理时延、排队时延。

时延带宽积：传播时延*带宽。

往返时间

利用率——网络利用率（信道利用率加权平均值）、信道利用率（有数据时间/总时间）

Bite: 位

1字节等于8位，8B。常说的带宽大小单位是位。


### 物理层

确定传输媒体的特性：机械特性、电气特性、功能特性、过程特性。

数据通信：

![avatar](./images/computer/3.png)

通信为了传输数据。数据——运送消息的实体。信号是数据的电气或者电磁的表现（模拟信号：取值是连续的，数字信号：取值是跳跃的）。码元：基本波形。1码元可以携带nbit信息量。