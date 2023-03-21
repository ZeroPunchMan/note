L2CAP是基于channels,每个channel都与一个CID关联

# 2.1 CID
```
1. CID是设备上channel端点的本地名,null CID不应使用,0x0001~0x003f用作L2CAP的保留功能.
    至少应该支持L2CAP Signaling channel (fixed channel 0x0001),或者在LE上支持L2CAP LE Signaling channel (fixed channel 0x0005).
    如果支持0x0005,也应该支持0x0004和0x0006
    L2CAP_INFORMATION_REQ / L2CAP_INFORMATION_RSP 机制用来决定远程设备支持哪些固定channel.

2. 固定channel在ACL-U或LE-U连接创建时,就会初始化.
    固定channel仅用于ACL-U,APB-U,LE-U逻辑连接.

3. CID的管理,由具体实现决定;
    同时存在的两个L2CAP channel,不共享CID.
    ACL-U,APB-U,LE-U使用不同的CID命名空间.

4. CID的动态分配与每个逻辑连接有关,一个设备可以独立分配CID.
    如果不同remote设备分配了相同的CID,也可以区分,因为连接不同.
    即使相同的CID被分配到同一个remote设备,也可以区分,因为被绑定至不同的逻辑连接? //note--
```

ACL-U的CID命名空间
```
0x0000          Null identifier
0x0001          L2CAP Signaling channel
0x0002          Connectionless channel
0x0003          Previously used,Not applicable
0x0007          BR/EDR Security Manager
0x003F          Previously used,Not applicable
0x0040~0xFFFF   L2CAP动态分配
```

APB-U的CID
```
0x0000          Null identifier
0x0002          Connectionless channel
others          保留 for future use
```

LE-U的CID
```
0x0000          Null identifier
0x0004          Attribute protocol
0x0005          L2CAP LE Signaling channel
0x0006          Security Manager protocol
0x0020~0x003E   已分配(查文档)
0x0040~0x007F   L2CAP动态分配
others          保留 for future use
```

# 2.2 OPERATION BETWEEN DEVICES
有图 channels.png
```
1. 面向连接的数据channel,CID与逻辑连接结合,识别channel的两个端点.
    用于广播时,无连接通道限制了数据流为单向,可用来传数据给所有激活的外设(使用Active Peripheral Broadcast).
    用于单播时,无连接通道可用于中心和外设双向通信.

2. ACL-U中,0x0001用来创建面向连接的数据通道,对接面向连接通道的特性改变,或是发现无连接通道的特性.

3. 0x0001在ACL-U逻辑连接建立后即可以用.
    0x0002用来收发无连接数据,当ACL-U逻辑连接建立后,且发送方得知接收方支持无连接数据,就可以开始数据流.

4. LE-U逻辑连接建立后,0x0005就可以立即使用.
```

|       Channel Type        |        Local CID (sending)        |   Remote CID (receiving)      |
|:-------------------------:|:---------------------------------:|:-----------------------------:|
|    Connection-oriented    |  Dynamically allocated and fixed  |Dynamically allocated and fixed|
|    Connectionless data    |           0x0002 (fixed)          |           0x0002 (fixed)      |
|      L2CAP Signaling      |     0x0001 and 0x0005 (fixed)     |   0x0001 and 0x0005 (fixed)   |

## 2.3 OPERATION BETWEEN LAYERS
L2CAP与上层和下层交互数据,没什么特别的

## 2.4 MODES OF OPERATION 
```
L2CAP通道需要从下面选择一个操作模式:
1. 基本L2CAP模式
2. 流控模式
3. 重传模式
4. 强化重传模式
5. streaming模式
6. 基于LE Credit的流控模式
7. 基于强化Credit的流控模式

默认为基本L2CAP模式;流控(2)和重传(3)模式仅在不支持4,5,7模式时使用.
```

```
在流控(2),重传(3)和强化重传(4)模式时,PDU需要编号和应答,编号用来控制PDU缓存,TxWindow size用来限制所需的缓存空间,并提供一种流控方法.
```

```
1. 流控模式时,不需要重传,仅报告PDU丢失.

2. 重传模式时,需要保证PDU送达, go-back-n机制用来简化协议和限制所需buffer大小.

3. 强化重传模式,类似重传模式,可设置POLL位来征求远程L2CAP实体的回复,增加SREJ S-frame来提高错误回复效率,增加RNR S-frame来代替R-bit来报告本地忙.

4. streaming模式用来实时传输数据,PDU编号但不响应;发送方会flush掉超时的包,接收方缓存不够则会用新的PDU覆盖旧的;丢失的PDU会被检测并报告,TxWindow size在此模式无效.

5. 基于LE Credit的流控模式,用于LE L2CAP面向连接的通道,用基于credit的方案实现L2CAP数据的流控(非signaling包).

6. 基于强化Credit的流控模式,类似5,可用于LE和BR/EDR.

注意legacy profile下,强化重传和streaming模式的性能,最好配置基本模式降低风险? //note--
```

## 2.5 MAPPING CHANNELS TO LOGICAL LINKS
```
L2CAP将通道映射到controller的逻辑连接;
所有逻辑连接都运行于一个物理连接之上,每个BR/EDR对应一个ACL-U逻辑连接,每个LE物理连接对应一个LE-U逻辑连接.

通道用BR/EDR物理连接时,要尽力和保证映射到一个ACL-U逻辑连接? //note--
同理用LE物理连接时,映射到LE-U逻辑连接.
对于BR/EDR控制器,准入控制(创建有保证的逻辑连接),由L2CAP层执行.
```
