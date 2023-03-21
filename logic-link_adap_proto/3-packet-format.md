# DATA PACKET FORMAT
```
1. 除0x0002(无连接通道),0x0001和0x0005(发信号通道),其他都是面向连接的.
2. 字节序都是小端,不考虑payload.
3. 更高层协议字节序由上层协议指定.
4. 收到来自未分配CID的PDU,则无视.
5. PDU的长度字段,不包括PDU的基本L2CAP头,所以PDU最长为65535+4.
6. PDU中的payload,不要超过对应设备的MPS
```

## CONNECTION-ORIENTED CHANNELS IN BASIC L2CAP MODE
在基本L2CAP模式,面向连接通道的PDU,也较多B-frame.

LSB first
| PDU Length |  CID  | infomation payload |
|:----------:|:-----:|:------------------:|
|   16 bits  |16 bits|    0~65535 Bytes   |

```
1. payload不要超过MTU,即SDU的最大尺寸.
2. 动态分配CID的通道,其MTU由配置时决定.
3. signaling通道有最小MTU值.
```

## CONNECTIONLESS DATA CHANNEL IN BASIC L2CAP MODE
也叫G-frame

LSB first
| PDU Length |  CID  |     PSM     | infomation payload |
|:----------:|:-----:|:-----------:|:------------------:|
|   16 bits  | 0x0002|  >=16 bits  |    0~65533 Bytes   |

```
1. 可用来广播或给指定给远程设备发数据.
2. 受MTU限制,具体实现应支持48字节,可以显式修改此值.
3. PSM增加,则payload要缩小.
```

## CONNECTION-ORIENTED CHANNEL IN RETRANSMISSION/FLOW CONTROL/STREAMING MODES 


