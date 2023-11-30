# BLE HID要求
|          Service           | Requirement |
| :------------------------: | :---------: |
|        HID Service         |      M      |
|      Battery Service       |      M      |
| Device Information Service |      M      |
|  Scan Parameters Service   |      O      |
```
all services should be Primary Service

The HID Device shall use LE Security Mode 1 and either Security Level 2 or 3

adv requirements:
    1.service UUIDs AD Type
    2.local name AD Type
    3.appearance AD Type
```

## BAS
```
Battery Level Characteristics is mandatory;
Battery Level descripto is mandatory if a device has more than one instance of Battery Service;
all other chars are optional.

Read Characteristic Descriptors are mandatory.
```

## DIS 
PnP ID Characteristics is mandatory by HID spec
```
Mandatory Characteristics
The Device Information Service shall include the PnP ID characteristic for reading the
PnP ID fields for the HID Device.
```

### PnP ID 0x2a50
```
1.Vendor ID Source 
    0x01: bluetooth sig assigned
    0x02: usb assigned

2. VID,PID
    from usb, 2 bytes

3. Product Version
    0xJJMN for version JJ.M.N (JJ – major version number, M – minor version number, N – sub-minor version number)
```

## HID

|        GATT Sub-Procedure        | Requirement |
| :------------------------------: | :---------: |
|  Read Long Characteristic Value  |      M      |
|      Write Without Response      |      M      |
|    Write Characteristic Value    |      M      |
|          Notifications           |      M      |
| Read Characteristic Descriptors  |      M      |
| Write Characteristic Descriptors |      M      |

### chars
|     Characteristic Name     | Requirement |       Mandatory Properties        | Optional Properties | Security Permissions |
| :-------------------------: | :---------: | :-------------------------------: | :-----------------: | :------------------: |
|        Protocol Mode        |     C.4     |    Read /WriteWithoutResponse     |                     |         None         |
|           Report            |      O      |                                   |                     |                      |
|  Report: Input-Report Type  |     C.1     |            Read/Notify            |        Write        |         None         |
| Report: Output-Report Type  |     C.1     | Read/Write/Write Without Response |                     |         None         |
| Report: Feature Report Type |     C.1     |            Read/Write             |                     |         None         |
|         Report Map          |      M      |               Read                |                     |         None         |
| Boot Keyboard Input Report  |     C.2     |            Read/Notify            |        Write        |         None         |
| Boot Keyboard Output Report |     C.2     | Read/Write/Write Without Response |                     |         None         |
|   Boot Mouse Input Report   |     C.3     |            Read/Write             |                     |         None         |
|       HID Information       |      M      |               Read                |                     |         None         |
|      HID Control Point      |      M      |       WriteWithoutResponse        |                     |         None         |

```
C.1: Mandatory to support at least one Report Type if the Report characteristic is supported
C.2: Mandatory for HID Devices operating as keyboards, else excluded.
C.3: Mandatory for HID Devices operating as mice, else excluded.
C.4: Mandatory for HID Devices supporting Boot Protocol Mode, otherwise optional. 

little endian
```

```
Protocol Mode: boot or report
HID Information： 2 octets bcdHID, 1 octet country code, 1 octet flag: bit0-remotewakeup, bit1-adv when bound but not connected
HID Control Point: suspend(0) and exit-suspend(1)
```

Report:
|  Report Type   | Requirement | Read  | Write | Write Without Response | Notify |
| :------------: | :---------: | :---: | :---: | :--------------------: | :----: |
|  Input Report  |     C.1     |   M   |   O   |           X            |   M    |
| Output Report  |     C.1     |   M   |   M   |           M            |   X    |
| Feature Report |     C.1     |   M   |   M   |           X            |   X    |
```
1.Client Characteristic Configuration Descriptor
    chars with input report

2.Report Reference Characteristic Descriptor
    Report ID(1 octet) and Report Type(bit1-input; bit2-output; bit3-feature)
    in each report characteristic for Report Protocol Mode.
```



# xbox one s HID服务
```
BT_GATT_PRIMARY_SERVICE(BT_UUID_DECLARE_16(0x1812)),
BT_GATT_CHARACTERISTIC(BT_UUID_DECLARE_16(0x2a4a), BT_GATT_CHRC_READ,
                        BT_GATT_PERM_READ_ENCRYPT, read_info, NULL, &hidInfo),
BT_GATT_CHARACTERISTIC(BT_UUID_DECLARE_16(0x2a4b), BT_GATT_CHRC_READ,
                        BT_GATT_PERM_READ_ENCRYPT, read_report_map, NULL, NULL),
BT_GATT_CHARACTERISTIC(BT_UUID_DECLARE_16(0x2a4d),
                        BT_GATT_CHRC_READ | BT_GATT_CHRC_NOTIFY,
                        BT_GATT_PERM_READ_ENCRYPT,
                        read_input_report, NULL, NULL),
BT_GATT_CCC(input_ccc_changed,
            BT_GATT_PERM_READ_ENCRYPT | BT_GATT_PERM_WRITE_ENCRYPT), // todo smp
BT_GATT_DESCRIPTOR(BT_UUID_DECLARE_16(0x2908), BT_GATT_PERM_READ_ENCRYPT,
                    read_report_meta, NULL, &inputMeta),
BT_GATT_CHARACTERISTIC(BT_UUID_DECLARE_16(0x2a4c),
                        BT_GATT_CHRC_WRITE_WITHOUT_RESP,
                        BT_GATT_PERM_WRITE_ENCRYPT,
                        NULL, write_ctrl_point, &ctrl_point),

BT_GATT_CHARACTERISTIC(BT_UUID_DECLARE_16(0x2a4d),
                        BT_GATT_CHRC_READ | BT_GATT_CHRC_WRITE_WITHOUT_RESP | BT_GATT_CHRC_WRITE,
                        BT_GATT_PERM_READ_ENCRYPT | BT_GATT_PERM_WRITE_ENCRYPT,
                        read_output_report, write_output_report, NULL),

BT_GATT_DESCRIPTOR(BT_UUID_DECLARE_16(0x2908), BT_GATT_PERM_READ_ENCRYPT,
                    read_report_meta, NULL, &outputMeta), 
```
