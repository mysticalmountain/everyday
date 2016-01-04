# ISO 8583
定义：金融交易卡组织信息交换协议
## message format
消息结构
- 0 field - 消息 类型标志
- 1 field - bitmap，64或者128位，标志field的出现或缺席
- 2..128 field - 其他的field
### field
- fixed length
    - numeric
    - alphanumeric
    - binary
- variable length with a max length 99
    + numeric
    + alphanumeric
    + binary
- variable length with a max length 999
    + numeric
    + alphanumeric
    + binary
- variable length with a max length 9999(avaliable starting in iso 8583 version 2003)
    + numeric
    + alphanumeric
    + binary
- nested message

## wire protocol
TCP/IP, UDP, X.25, SDLC, SNA, ASYNC, QTP, SSL, HTTP。通讯协议不是iso8583标准的一部分。所以不同的供应商使用不同的协议。

