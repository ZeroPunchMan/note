# 1.1 L2CAP features
有图

## Protocol/channel multiplexing
```
1. L2CAP可在不同controller多路复用,一个L2CAP的channel只能操作一个controller,一个channel不能在不同controller之间转换.
2. 不同逻辑channel对于上层实体可区分,同时可能有多个channel
```

## Segmentation and reassembly
```
对于上层应用,会做数据切片/重组处理,主要考虑重传/纠错/资源管理方便
```

## Flow control per L2CAP channel
```
1. controller提供了传输的错误和流控,HCI流控则控制HCI传输.
2. 多个L2CAP的channel使用同一controller,则提供基于窗口的流控? //note--
```

## Error control and retransmissions
```
L2CAP在controller之上额外做了错误检查,还做了丢包处理与重传处理
```

## Support for Streaming
```
channel支持stream模式,比如音频,在发送端使用flush超时保持数据流,同时关闭接收端HCI和controller的流控制
```

##  Fragmentation and Recombination
```
各层之间有拆包和重组,L2CAP会重组其PDU
```

##  Quality of Service
```
L2CAP监视协议资源,保证QoS可信.
对于BR/EDR or BR/EDR/LE Controller,L2CAP提供isochronous(Guaranteed) and asynchronous(Best Effort)两种数据流,
需要标记packet为"可自动flush"或"不可自动flush", 设置Packet_Boundary_Flag in the HCI ACL Data packet
自动flush的数据会在超时(ACL逻辑连接中设置)时flush掉.
```

# 1.2 ASSUMPTIONS
```
1. 假设数据包按顺序发送,可能有单个包污染或多发;
    BR/EDR与BR/EDR/LE controller,两个设备间最多一个ACL-U逻辑连接;
    BR/EDR/LE与LE controller,两个设备间最多一个LE-U逻辑连接;

2. controller提供全双工channel,但不代表所有L2CAP通信都是双向;单向数据不需要双工通道.

3. L2CAP层提供一定的可靠性,基于controller中的机制,以及enhanced L2CAP中可使能:包拆分,错误检测,重传;
    一些controller会做累加校验,并重发数据,直到ACK或超时;
    另外一些controller会重发一定次数后flush掉;
    因为ACK可能丢包,数据发送成功后仍然可能发生超时;
    PS: BR/EDR or BR/EDR/LE的广播包不是可靠的,且所有广播开启L2CAP packet的first segment with the same sequence bit?

4. controller提供空中数据的错误和流控,HCI流控for数据在HCI的传递,但一些应用可能想要更好的效果;
    错误和流控模块提供四种模式:
        重传模式
        重传模式with: 片段化,流控,L2CAP PDU重传
        流控模式: 仅片段化和流控
        Streaming模式: 片段化,接收端的包flushing
```

# 1.3 SCOPE
```
1. L2CAP不提供同步数据(特定的SCO和eSCO逻辑传输).
2. L2CAP不支持可靠广播channel.
```
# 1.4 TERMINOLOGY 

```
1. Upper Layer
    与L2CAP用SDUs交换数据,通常为应用或更高层协议(Service Level Protocol),与L2CAP具体接口未指定.

2. Lower Layer
    与L2CAP用PDUs或者PDU片段交互,通常存在与controller中,但HCI驱动也可以视同.除了HCI功能以外,具体接口都未指定.

3. L2CAP channel
    对等设备两个端点之间的逻辑连接,用CID(Channel Identifiers)描述,可复用在一个controller逻辑连接上.

4. SDU
    L2CAP与上层交换,并通过channel传输的数据,仅与上层发起数据有关,不包括L2CAP的协议信息.

5. Segment, or SDU segment
    SDU片段,一个SDU可能分为一个或多个片段.仅在强化重传模式,streaming模式,重传模式,流控模式.在基本的L2CAP模式中无效.

6. Segmentation
    片段化,产生5的过程.

7. Reassembly
    6的逆过程,片段重组.

8. PDU
    数据包,包含L2CAP协议信息字段,以及上层信息数据.总是以基本L2CAP header开始,类型有B-frames,还有I/S/C/G/K-frames.

9. Basic L2CAP header
    包含最小的L2CAP协议信息:PDU长度,CID.

10. B-frame
    Basic information frame, 包含一个完整的SDU作为payload,用一个basic L2CAP header封包.

11. I-frame
    Information frame, 包括SDU片段,附加的协议信息,用一个basic L2CAP header封包.用于加强重传模式,streaming模式,重传模式,流控模式.

12. S-frame
    Supervisory frame, 仅包含协议信息,用一个basic L2CAP header封包,不包含SDU数据.用于强化重传模式,重传模式,流控模式.

13. C-frame
    Control frame, 用于L2CAP实体之间交换信号消息,仅用于信号channel.

14. G-frame
    Group frame, 仅用于L2CAP无连接channel,包含PSM,后跟完整的SDU,用basic L2CAP header封包,用于广播激活外设,或发送单播数据到单一远程设备.

15. K-frame
    Credit-based frame, 用于LE Credit Based流控模式,包含SDU片段和附加协议信息,用basic L2CAP header封.

16. Fragment
    PDU的一部分,仅用于与下层交换数据,不用于P2P传输.根据PDU可以为start或continuation,不包含PDU以外的协议信息,在下层中区分传输?//note--

17. Fragmentation
    碎片化,用于HCI驱动或controller,把PDU碎片化以放进HCI-ACL数据包或controller数据包中.可用于所有的L2CAP模式.

18. Recombination
    逆碎片化,重新建立PDU,其在协议栈的位置,不一定与发送方的碎片化相同.

19. MTU
    Maximum Transmission Unit, SDU最大尺寸,上层实体有能力处理.

20. Payload Size
    PDU中的SDU数据量

21. MPS
    Maximum PDU payload Size, 在基本L2CAP模式中,会与MTU值相同.

22. MTUsig
    Signaling MTU, C-frame中除去header外的大小,当C-frame太大而被peer拒收时,会被发现.

23. MTUcnl
    Connectionless MTU, 连接包信息的最大尺寸,G-frame中除去header和PSM以外的最大尺寸,可以发L2CAP_INFORMATION_REQ包来发现. 

24. MaxTransmit
    在强化重传模式和重传模式中,PDU丢包的重传次数,0表示无限次,1表示禁止重传;设置为1时,重传PDU失败会导致连接断开,作为比较,在流控模式中则不会导致连接断开.
```
