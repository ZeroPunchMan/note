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
| PDU Length |   CID   | infomation payload |
| :--------: | :-----: | :----------------: |
|  16 bits   | 16 bits |   0~65535 Bytes    |

```
1. payload不要超过MTU,即SDU的最大尺寸.
2. 动态分配CID的通道,其MTU由配置时决定.
3. signaling通道有最小MTU值.
```

## CONNECTIONLESS DATA CHANNEL IN BASIC L2CAP MODE
也叫G-frame

LSB first
| PDU Length |  CID   |    PSM    | infomation payload |
| :--------: | :----: | :-------: | :----------------: |
|  16 bits   | 0x0002 | >=16 bits |   0~65533 Bytes    |

```
1. 可用来广播或给指定给远程设备发数据.
2. 受MTU限制,具体实现应支持48字节,可以显式修改此值.
3. PSM增加,则payload要缩小.
```

## CONNECTION-ORIENTED CHANNEL IN RETRANSMISSION/FLOW CONTROL/STREAMING MODES 
I-frame用S-frame应答.

### S-frame结构
LSB first
| PDU Length |  CID   |   Control   |     FCS     |
| :--------: | :----: | :---------: | :---------: |
|  16 bits   | 0x0002 | 16or32 bits | 0 or16 bits |

### I-frame结构
LSB first
| PDU Length |  CID   |   Control   | SDU length  | infomation payload |     FCS     |
| :--------: | :----: | :---------: | :---------: | :----------------: | :---------: |
|  16 bits   | 0x0002 | 16or32 bits | 0 or16 bits |        var         | 0 or16 bits |

```
1. FCS是可选的.
2. SDU长度仅在L2CAP SDU的起始帧存在,SAR=0b01
3. 标准和强化的Conctrol是16位,扩展是32位.
```

```
PDU长度包括Control,SDU长度(可选),payload(可选),FCS.
所以S-frame的长度为2,4,6.
I-frame中的payload则根据情况限制最大值.
```
### Control字段
```
1. 标准control字段用于重传和流控模式.
2. 强化和扩展用于强化重传和streaming模式.
3. 在Extended Window Size option成功之前,用强化control,之后用扩展.
```

```
S-frame类型有RR(Receiver Ready), REJ(Reject), RNR(Receiver Not Ready), SREJ(Selective Reject).
```

#### I-frame Standard Control Field
LSB first
| Type  | TxSeq  |   R   | ReqSeq |  SAR   |
| :---: | :----: | :---: | :----: | :----: |
| 1 bit | 6 bits | 1 bit | 6 bits | 2 bits |

#### I-frame Enhanced Control Field
LSB first
| Type  | TxSeq  |   F   | ReqSeq |  SAR   |
| :---: | :----: | :---: | :----: | :----: |
| 1 bit | 6 bits | 1 bit | 6 bits | 2 bits |

#### I-frame Extended Control Field
LSB first
| Type  |   F   | ReqSeq  |  SAR   |  TxSeq  |
| :---: | :---: | :-----: | :----: | :-----: |
| 1 bit | 1 bit | 14 bits | 2 bits | 14 bits |


#### S-frame Standard Control Field
LSB first
| Type  |  RFU  |   S    |  RFU   |   R   | ReqSeq |  RFU   |
| :---: | :---: | :----: | :----: | :---: | :----: | :----: |
| 1 bit | 1 bit | 2 bits | 3 bits | 1 bit | 6 bits | 2 bits |

#### S-frame Enhanced Control Field
LSB first
| Type  |  RFU  |   S    |   P   |  RFU   |   F   | ReqSeq |  RFU   |
| :---: | :---: | :----: | :---: | :----: | :---: | :----: | :----: |
| 1 bit | 1 bit | 2 bits | 1 bit | 2 bits | 1 bit | 6 bits | 2 bits |

#### S-frame Extended Control Field
LSB first
| Type  |   F   | ReqSeq  |   S    |   P    |   RFU   |
| :---: | :---: | :-----: | :----: | :----: | :-----: |
| 1 bit | 1 bit | 14 bits | 2 bits | 1 bits | 13 bits |

```
Type字段: 
    I-frame为0,S-frame为1

TxSeq字段:
    Send Sequence Number, 用来给I-frame编号,使PDU frame序列化,支持重传.

ReqSeq字段:
    Receive Sequence Number, 接收方应答I-frame, 在REJ和SREJ帧中用来请求特定TxSeq的I-frame重传

R字段:
    Retransmission Disable Bit,用来实现流控,接收方设置1用来告知发送方,停止重发I-frame.
    R与ReqSeq独立

SAR:
    Segmentation and Reassembly,用来标识SDU的片段化.
    0b00: 没有片段化的SDU
    0b01: start of SDU
    0b10: end of SDU
    0b11: continuation of SDU

S:
    Supervisory function, 标识S-frame的类型,以下四种
    0b00: RR - Receiver Ready
    0b01: REJ - Reject
    0b10: RNR - Receiver Not Ready
    0b11: SREJ - Select Reject

P: 
    poll, 设置1用来向接收方征求回复,接收方应立即回复,并设置F为1

F:
    final, 用来应答P
```

### L2CAP SDU Length field (2 octets)
```
SAR为0b01时,在payload前,指定SDU总大小,且不应该大于对等设备的MTU.
如果SAR为0b00,则SDU未片段化,不需要此字段.
```

### Information Payload field
```
不要超过MPS,如果包含未片段化的SDU,也不要超过MTU
```

### Frame Check Sequence (2 octets) 
重传和流控模式是强制要求的,强化重传和streaming模式是可能存在.

算法有图 fcs.png,例子如下:
```
1. I Frame
PDU Length = 14
Channel ID = 0x0040
Control = 0x0002 (SAR=0b00, ReqSeq=0b000000, R=0, TxSeq=0b000001)
Information Payload = 00 01 02 03 04 05 06 07 08 09 (10 octets, hexadecimal notation)
==> FCS = 0x6138
==> Data to Send = 0E 00 40 00 02 00 00 01 02 03 04 05 06 07 08 09 38 61
(hexadecimal notation)


2. RR Frame
PDU Length = 4
Channel ID = 0x0040
Control = 0x0101 (ReqSeq=0b000001, R=0, S=0b00)
==> FCS = 0x14D4
==> Data to Send = 04 00 40 00 01 01 D4 14 (hexadecimal notation)
```

### Invalid Frame Detection
```
一些情况的包可以丢掉,如CID不对,SAR顺序不对,FCS错误,包字节数过少,PDU的payload长度超过MPS,等等
```

## 3.4 CONNECTION-ORIENTED CHANNELS IN LE CREDIT BASED FLOW CONTROL MODE AND ENHANCED CREDIT BASED FLOW CONTROL MODE
面向连接的通道,LE Credit流控模式和强化Credit流控模式.

K-frame
| PDU Length |  CID   | SDU length  | infomation payload |
| :--------: | :----: | :---------: | :----------------: |
|  16 bits   | 0x0002 | 0 or16 bits |        var         |

```
第一个K-frame存在SDU长度,后续的帧全是payload,以下情况接收方会直接断开通道:
1. SDU长度超过接收方的MTU
2. payload超过MPS
3. 总的payload超过了指定的SDU长度
```

