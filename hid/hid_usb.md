# 设备描述符
```
bLength                  : 0x09 (9 bytes)
bDescriptorType          : 0x04 (Interface Descriptor)
bInterfaceNumber         : 0x00 (Interface 0)
bAlternateSetting        : 0x00
bNumEndpoints            : 0x01 (1 Endpoint)
bInterfaceClass          : 0x03 (HID - Human Interface Device)  //指定接口为HID类
bInterfaceSubClass       : 0x01 (Boot Interface)  //指定是否支持BOOT固定协议,BIOS中操作
bInterfaceProtocol       : 0x01 (Keyboard)
iInterface               : 0x00 (No String Descriptor)
Data (HexDump)           : 09 04 00 00 01 03 01 01 00    
```

# HID描述符
跟在接口描述符后面
```
bLength                  : 0x09 (9 bytes) //一个报告时为9,每多一个则+2
bDescriptorType          : 0x21 (HID Descriptor)
bcdHID                   : 0x0111 (HID Version 1.11)
bCountryCode             : 0x00 (00 = not localized)
bNumDescriptors          : 0x01  //可以有多个描述符
Descriptor 1:
bDescriptorType          : 0x22 (Class=Report)
wDescriptorLength        : 0x004D (77 bytes)
Descriptor 2:
bDescriptorType          : 0x22 (Class=Report)
wDescriptorLength        : 0x004D (77 bytes)
```

通过标准请求(Get_Descriptor Request)获取HID报告描述符,wValue指定描述符类型
```
0x21 HID
0x22 Report
0x23 Physical descriptor
0x24 - 0x2F Reserved
```
