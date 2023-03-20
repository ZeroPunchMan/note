# ymodem


|                     SENDER                      |     RECEIVER    |
|-------------------------------------------------|:---------------:|
|                                                 | "sb foo.*<CR>"  |
|        "sending in batch mode etc."             |                 |  
|                                                 |  C (command:rb) |  
|         SOH 00 FF foo.c NUL[123] CRC CRC        |                 |  
|                                                 |      ACK C      |  
|         SOH 01 FE Data[128] CRC CRC             |                 |  
|                                                 |      ACK        |  
|         SOH 02 FC Data[128] CRC CRC             |                 |  
|                                                 |      ACK        |  
|         SOH 03 FB Data[100] CPMEOF[28] CRC CRC  |                 |  
|                                                 |      ACK        |  
|         EOT                                     |                 |   
|                                                 |      NAK        |   
|         EOT--resend                             |                 |   
|                                                 |      ACK  C     |   
|         SOH 00 FF NUL[128] CRC CRC              |                 |   
|                                                 |      ACK        |   

```
    SOH = 0x01,
    STX = 0x02,
    EOT = 0x04,
    ACK = 0x06,
    NAK = 0x15,
    CAN = 0x18,
    C = 0x43,

    NULL = 0x00,
    CPMEOF = 0X1a,
```

receiver-driven

## Receive_Program_Considerations

```
The receiver has a 10-second timeout. It sends a <nak> every time it
 times out. The receiver's first timeout, which sends a <nak>, signals the
 transmitter to start. Optionally, the receiver could send a <nak>
 immediately, in case the sender was ready. This would save the initial 10
 second timeout. However, the receiver MUST continue to timeout every 10
 seconds in case the sender wasn't ready.

开始时10秒超时,接收方发<nak>,提示发送方开始;
如果发送方就绪,也可以直接发<nak>,节约10s.
```

```
Once into a receiving a block, the receiver goes into a one-second timeout
 for each character and the checksum. If the receiver wishes to <nak> a
 block for any reason (invalid header, timeout receiving data), it must
 wait for the line to clear. See "programming tips" for ideas

接收一个block内1秒超时,接收方发<nak>需等到行结结束.
```

```
Synchronizing: If a valid block number is received, it will be: 1) the
 expected one, in which case everything is fine; or 2) a repeat of the
 previously received block. This should be considered OK, and only
 indicates that the receivers <ack> got glitched, and the sender re-
 transmitted; 3) any other block number indicates a fatal loss of
 synchronization, such as the rare case of the sender getting a line-glitch
 that looked like an <ack>. Abort the transmission, sending a <can>

接收到正确的block,或重复发之前的block,接收方都<ack>;
其他错误情况,接收方发<can>终止.
```

## Sending_program_considerations

```
While waiting for transmission to begin, the sender has only a single very
 long timeout, say one minute. In the current protocol, the sender has a
 10 second timeout before retrying. I suggest NOT doing this, and letting
 the protocol be completely receiver-driven. This will be compatible with
 existing programs.

等待开始时,发送方仅有1min的超时判定,但目前协议用10s超时?
```

```
When the sender has no more data, it sends an <eot>, and awaits an <ack>,
 resending the <eot> if it doesn't get one. Again, the protocol could be
 receiver-driven, with the sender only having the high-level 1-minute
 timeout to abort.

发送方发<eot>后等待<ack>,如果没有则重发,sender仅保留1min超时判断.
```

## 例子
```
         SENDER                             RECEIVER
                                times out after 10 seconds,    
                                <---        <nak>
 <soh> 01 FE -data- <xx>        --->
                                <---        <ack>
 <soh> 02 FD -data- xx          --->        (data gets line hit)
                                <---        <nak>
 <soh> 02 FD -data- xx          --->
                                <---        <ack>
 <soh> 03 FC -data- xx          --->
 (ack gets garbaged)            <---        <ack>
 <soh> 03 FC -data- xx          --->        <ack>
 <eot>                          --->
                                <---        <anything except ack>
 <eot>                          --->
                                <---        <ack>

 (finished)
```

## 超时判定
接收方:
```
1.一包内1s超时
2.包间10s超时
```

发送方:
```
仅保留1min超时判断(实现改为10s)
```
