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

| Code=0x02 | Identifier | Data Length |        PSM         | Source CID |
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
| Code=0x03 | Identifier | Data Length | Destination CID | Source CID |  Result  |  Status  |
| :-------: | :--------: | :---------: | :-------------: | :--------: | :------: | :------: |
|   0x03    |  octet 1   |  octet 2~3  |    2 octets     |  2 octets  | 2 octets | 2 octets |

```
Destination CID: 发送此响应的端点CID
Source CID: 之前发4.2请求的ID
Result: 0x00表示成功,收到此成功结果时,一个通道就建立了.如果非0,则DCID和SCID无效.
Status: Result为Pending时有效.
```

### Result
| Value  |                    Description                    |
| :----: | :-----------------------------------------------: |
| 0x0000 |              Connection successful.               |
| 0x0001 |                Connection pending                 |
| 0x0002 |      Connection refused – PSM not supported.      |
| 0x0003 |       Connection refused – security block.        |
| 0x0004 |   Connection refused – no resources available.    |
| 0x0006 |      Connection refused – invalid Source CID      |
| 0x0007 | Connection refused – Source CID already allocated |
| Other  |             Reserved for future use.              |

### Status
Result为Pending时
| Value  |           Description            |
| :----: | :------------------------------: |
| 0x0000 | No further information available |
| 0x0001 |      Authentication pending      |
| 0x0002 |      Authorization pending       |
| Other  |     Reserved for future use      |

## 4.4 L2CAP_CONFIGURATION_REQ (CODE 0x04)
```
用来初始化逻辑连接传输合约,包含一些需要设置的参数,如果没包含,则为默认值或之前设置的值.
如果没有任何参数需要设置,则C flag要设置为0.
即使全部都用默认值,也应该发无配置参数的包.
```
| Code=0x04 | Identifier | Data Length | Destination CID |  Flags   | Configuration Options |
| :-------: | :--------: | :---------: | :-------------: | :------: | :-------------------: |
|   0x04    |  octet 1   |  octet 2~3  |    2 octets     | 2 octets |          var          |

```
Destination CID: 接收此请求的端点.
```

### Flags 
         
|  RFU  |   C   |
| :---: | :---: |
|  MSB  |  LSB  |
```
1. C表示有多个包要继续发.
2. 如果两个L2CAP实体都支持扩展流控指定配置,则C一直为0.
3. 尽量一个包把参数配置完,但要考虑MTUsig可能需要多包配置,不要把一个option拆开放到两包中.
4. 每个req要不同id,并且有同id的rsp.
5. rsp包含req的options(除非出错,更适合回复REJECT_RSP),或者一个"Success"并不包含option.
6. options要延迟到req包全部收到,此时表示req事件发生.
7. rsp的C flag应该与req相同,如果req发了C=0的包,而rsp回复了1,则表示rsp端有options要发给req端,
    此时req一直发null option包给rsp方,直到rsp方发C=0的包.此时表示rsp事件发生.
8. 所有options都应该成功,才表示此业务成功.
9. 接收方要能处理未知option,空的req可被用来请求一个rsp.
```

## 4.5 L2CAP_CONFIGURATION_RSP (CODE 0x05)

| Code=0x05 | Identifier | Data Length | Source CID |  Flags   |  Result  | Config |
| :-------: | :--------: | :---------: | :--------: | :------: | :------: | :----: |
|   0x05    |  octet 1   |  octet 2~3  |  2 octets  | 2 octets | 2 octets |  var   |


```
Source CID:  发req的端点,即接收此rsp的端点.
Flags: 同req,仅最低位C有用.
```
| Result |               Description               |
| :----: | :-------------------------------------: |
| 0x0000 |                 Success                 |
| 0x0001 |    Failure – unacceptable parameters    |
| 0x0002 | Failure – rejected (no reason provided) |
| 0x0003 |        Failure – unknown options        |
| 0x0004 |                 Pending                 |
| 0x0005 |      Failure - flow spec rejected       |
| Other  |         Reserved for future use         |

```
与Result有关.
0x000与0x0004: 成功或pending,包含配置参数的返回值
0x0001: 失败
0x0003: 包含接收方不理解的option, 如果是不理解的req,则视为hints.
```

## 4.6 L2CAP_DISCONNECTION_REQ (CODE 0x06)
| Code=0x06 | Identifier | Data Length | Destination CID | Source CID |
| :-------: | :--------: | :---------: | :-------------: | :--------: |
|   0x06    |  octet 1   |  octet 2~3  |    2 octets     |  2 octets  |

```
DCID: 接收此req的端点.错误要回复L2CAP_COMMAND_REJECT_RSP.
SCID: 发req的端点.仅SCID错误,则不管.
```

```
req发出后,发送方不再发其他数据;收到req后,接收方不再发其他数据.
```

## 4.7 L2CAP_DISCONNECTION_RSP (CODE 0x07)
| Code=0x07 | Identifier | Data Length | Destination CID | Source CID |
| :-------: | :--------: | :---------: | :-------------: | :--------: |
|   0x07    |  octet 1   |  octet 2~3  |    2 octets     |  2 octets  |

## 4.8~4.9 L2CAP_ECHO_REQ (CODE 0x08) and L2CAP_ECHO_RSP (CODE 0x09)
|   Code    | Identifier | Data Length | Echo Data |
| :-------: | :--------: | :---------: | :-------: |
| 0x08/0x09 |  octet 1   |  octet 2~3  | optional  |

```
用来测试连接
```

## 4.10 L2CAP_INFORMATION_REQ (CODE 0x0A) 
用来请求一些信息

| Code  | Identifier | Data Length | Info Type |
| :---: | :--------: | :---------: | :-------: |
| 0x0a  |  octet 1   |  octet 2~3  | 2 octets  |

| Value  |         Description         |
| :----: | :-------------------------: |
| 0x0001 |     Connectionless MTU      |
| 0x0002 | Extended features supported |
| 0x0003 |  Fixed channels supported   |
| Other  |   Reserved for future use   |

```
第一次校验前,不要往0x0001发0x0003的InfoType,因为远程设备的扩展特征位没设置?
```

## 4.11 L2CAP_INFORMATION_RSP (CODE 0x0B) 
| Code  | Identifier | Data Length | Info Type |  Result  |       Info       |
| :---: | :--------: | :---------: | :-------: | :------: | :--------------: |
| 0x0b  |  octet 1   |  octet 2~3  | 2 octets  | 2 octets | 0 or more octets |

| Result |       Description       |
| :----: | :---------------------: |
| 0x0000 |         Success         |
| 0x0001 |      Not supported      |
| Other  | Reserved for future use |

| InfoType |           Info           | Info length(octets) |
| :------: | :----------------------: | :-----------------: |
|  0x0001  |    Connectionless MTU    |          2          |
|  0x0002  |  Extended feature mask   |          4          |
|  0x0003  | Fixed channels Supported |          8          |

## 4.12 EXTENDED FEATURE MASK
见图 extended_feature_mask.png

## 4.13 FIXED CHANNELS SUPPORTED
见图 fixed_channels.png

```
除了signaling通道以外,不应该发数据;
除非用4.11包说明了支持的固定通道,或者收到了远程设备在固定通道发的包.
不支持的固定通道发来的包,直接丢掉.
```

## 4.14 ~ 4.19 已废弃

## 4.20 L2CAP_CONNECTION_PARAMETER_UPDATE_REQ(CODE 0x12)
```
主要用作外设给中心设备发,并且是双方controller和host都不支持Connection Parameters Request Link Layer Control procedure时.
外设收到则回一个L2CAP_COMMAND_REJECT_RSP packet with reason 0x0000 (Command not understood).
中心设备收到认可的参数,则发给controller.
```
| Code  | Identifier | Data Length | Interval_Min | Interval_Max | Latency  | Timeout  |
| :---: | :--------: | :---------: | :----------: | :----------: | :------: | :------: |
| 0x12  |  octet 1   |  octet 2~3  |   ? octets   |   ? octets   | ? octets | ? octets |

## 4.21 L2CAP_CONNECTION_PARAMETER_UPDATE_RSP(CODE 0x13)
| Code  | Identifier | Data Length | Result |
| :---: | :--------: | :---------: | :-------: |
| 0x13  |  octet 1   |  octet 2~3  | 2 octets  |

```
Result: 0-成功;1-拒绝;
```

