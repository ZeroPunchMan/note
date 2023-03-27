#   # SIGNALING PACKET FORMATS
```
1. 对于ACL-U逻辑连接,用CID-0x0001;而LE-U则用0x0005.
2. ACL-U中一个C-frame可能有多个命令,而LE-U一个C-frame只能包含一个命令.
3. C-frame的payload不要超过MTUsig,如果收到超过的包,则发一个L2CAP_COMMAND_REJECT_RSP,其中包含了MTUsig.
```

|                   Logical Link                   |   Minimum MTUsig   |
| :----------------------------------------------: | :----------------: |
| ACL-U not supporting Extended Flow Specification |     48 octets      |
| ACL-U supporting the Extended Flow Specification | feature 672 octets |
|                       LE-U                       |     23 octets      |

## C-frame结构
| PDU Length | Channel ID | payload |
| :--------: | :--------: | :-----: |
|  16 bits   |  16 bits   |   var   |


## 命令结构
|  Code   | Identifier | Data Length | Data  |
| :-----: | :--------: | :---------: | :---: |
| octet 0 |  octet 1   |  octet 2~3  |  var  |

### Code定义
|   Code    |              Description              | CIDs on which Code is Allowed |
| :-------: | :-----------------------------------: | :---------------------------: |
|   0x01    |       L2CAP_COMMAND_REJECT_RSP        |       0x0001 and 0x0005       |
|   0x02    |         L2CAP_CONNECTION_REQ          |            0x0001             |
|   0x03    |         L2CAP_CONNECTION_RSP          |            0x0001             |
|   0x04    |        L2CAP_CONFIGURATION_REQ        |            0x0001             |
|   0x05    |        L2CAP_CONFIGURATION_RSP        |            0x0001             |
|   0x06    |        L2CAP_DISCONNECTION_REQ        |       0x0001 and 0x0005       |
|   0x07    |        L2CAP_DISCONNECTION_RSP        |       0x0001 and 0x0005       |
|   0x08    |            L2CAP_ECHO_REQ             |            0x0001             |
|   0x09    |            L2CAP_ECHO_RSP             |            0x0001             |
|   0x0A    |         L2CAP_INFORMATION_REQ         |            0x0001             |
|   0x0B    |         L2CAP_INFORMATION_RSP         |            0x0001             |
| 0x0C~0x11 |            Previously used            |             None              |
|   0x12    | L2CAP_CONNECTION_PARAMETER_UPDATE_REQ |            0x0005             |
|   0x13    | L2CAP_CONNECTION_PARAMETER_UPDATE_RSP |            0x0005             |
|   0x14    | L2CAP_LE_CREDIT_BASED_CONNECTION_REQ  |            0x0005             |
|   0x15    | L2CAP_LE_CREDIT_BASED_CONNECTION_RSP  |            0x0005             |
|   0x16    |     L2CAP_FLOW_CONTROL_CREDIT_IND     |       0x0001 and 0x0005       |
|   0x17    |   L2CAP_CREDIT_BASED_CONNECTION_REQ   |       0x0001 and 0x0005       |
|   0x18    |   L2CAP_CREDIT_BASED_CONNECTION_RSP   |       0x0001 and 0x0005       |
|   0x19    |  L2CAP_CREDIT_BASED_RECONFIGURE_REQ   |       0x0001 and 0x0005       |
|   0x1A    |  L2CAP_CREDIT_BASED_RECONFIGURE_RSP   |       0x0001 and 0x0005       |
|   Other   |        Reserved for future use        |              Any              |

### Identifier
```
1. Req与Rsp用相同的id.
2. 每一个成功的命令,都使用不同id,用完后则循环使用.
3. 如果超时,重发也要用同样id;收到同样的id,一样响应.
4. 响应id无效,则无视掉;0x00是无效id.
```
### Data Length And Data
仅数据段长度,后跟数据段

## 4.1 L2CAP_COMMAND_REJECT_RSP (CODE 0x01)
用来回复一些未知的命令,或是对应的回复不当时,以及payload超过了MTUsig.

| Code=0x01 | Identifier | Data Length |  Reason   | Reason Data |
| :-------: | :--------: | :---------: | :-------: | :---------: |
|   0x01    |  octet 1   |  octet 2~3  | octet 4~5 |     var     |

| Reason |       Description       |                             Reason Data                              |
| :----: | :---------------------: | :------------------------------------------------------------------: |
| 0x0000 | Command not understood  |                               no data                                |
| 0x0001 | Signaling MTU exceeded  |                           2 octets MTUsig                            |
| 0x0002 | Invalid CID in request  | src CID + des CID, 4 octets,if only one CID, the other shall be null |
| Other  | Reserved for future use |                                                                      |

## 4.2 L2CAP_CONNECTION_REQ (CODE 0x02)
用来在两个设备间创建L2CAP通道,通道要在配置开始之前确立.

| Code=0x01 | Identifier | Data Length |        PSM         | Source CID |
| :-------: | :--------: | :---------: | :----------------: | :--------: |
|   0x02    |  octet 1   |  octet 2~3  | 2 octets (minimum) |  2 octets  |

### PSM-- Protocol/Service Multiplexer
```
1. 最高字节的最低位为0,其他字节的最低位都是1.比如0x0100-0x01FF不可用.
2. 只要找到第一个偶数的字节,就是PSM结尾.
3. 分为两段,第一段由蓝牙SIG和指定协议分配;第二段由Service Discovery protocol(SDP)结合动态分配,用来支持特定协议的多重实现.
```

```
PSM分段:
1. 0x0001~0x0EFF, 固定的
2. >0x1000, SDP分配
```
### SCID --Source CID
```
1. 是发起请求的端点ID,也接收此请求的响应.
2. 应该是动态分配的,且与设备上其他通道的CID不同.
```

## 4.3 L2CAP_CONNECTION_RSP (CODE 0x03)
