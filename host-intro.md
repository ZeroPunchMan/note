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
