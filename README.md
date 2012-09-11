Driver-USB-modem-HUAWEI-E1550
=============================

USB modem HUAWEI E1550
СКАЧАЙ driver HUAWEI switch USB-modem
после перейдите в директорию и запустите скомпилированный испольняемый скрипт
Код: [Выделить]
cd .../USB-modem
#./switch_huawei
специально для HUAWEI E1550

Протокол PPP в Solaris реализован демоном aspppd. Настройка aspppd храняться в файле /etc/asppp.cf состоит из двух разделов: раздела ifconfig и раздела path. 
Как оказалось, для перевода данного устройства в режим модема дополнительные программы не требуются. Достаточно из контекстного меню его иконки на рабочем столе выбрать "Извлечь том". Затем надо выполнить команду
update_drv -a -i '"usb19d2,2003"' usbsacm
и в /dev/term/ появятся желанные файлы устройств.
Set up dial-up configurations:

    # cat /etc/ppp/peers/huawei
    term/0
    460800
    connect "/usr/bin/chat -V -t60 -f /etc/ppp/huawei-chat"
    asyncmap 00000000
    crtscts
    defaultroute
    holdoff 1
    lcp-echo-interval 0
    noauth
    noccp
    novj
    passive
    updetach
    usepeerdns
    name CARD
    password CARD

    # cat /etc/ppp/huawei-chat
    ABORT BUSY
    ABORT 'NO CARRIER'
    ABORT ERROR
    REPORT CONNECT
    '' 'AT+CPIN?'
    TIMEOUT 5
    READY-AT+CPIN=????-OK 'AT&F'
    OK ATE
    OK 'ATZ Q0 V1 E1 S0=0 &C1 &D2 +FCLASS=0'
    SAY '\\nCalling ChinaTelcom'
    OK 'ATDT#777'
    TIMEOUT 60
    CONNECT \\n
    $ cat /etc/ppp/ip-up
    #!/usr/bin/ksh
    cp /etc/ppp/resolv.conf /etc
    cp /etc/nsswitch.dns /etc/nsswitch.conf

    $ cat /etc/ppp/ip-down
    #!/usr/bin/ksh
    cp /etc/nsswitch.nis /etc/nsswitch.conf

$ pppd call huawei
AT+CPIN?
+CPIN:READY

OK
AT&F
OK
Calling ChinaTelcom
AATZ Q0 V1 E1 S0=0 &C1 &D2 +FCLASS=0
OK
ATDT#777
CONNECTchat:  Nov 16 13:58:09 CONNECT
Serial connection established.
Using interface sppp0
Connect: sppp0 <--> /dev/term/4
Remote message: \^@
local  IP address 124.127.63.2
remote IP address 115.168.64.86
primary   DNS address 219.141.136.10
secondary DNS address 219.141.140.10
[root@~]$ mdb -k
Loading modules: [ unix genunix specfs dtrace mac cpu.generic uppc pcplusmp rootnex scsi_vhci ufs sockfs ip hook neti sctp arp usba uhci stmf fctl nca lofs zfs idm cpc random nfs fcip logindmux ptm sppp sd ]
> ::prtusb

INDEX   DRIVER      INST  NODE            VID.PID     PRODUCT             
1       ehci        0     pci1028,151     0000.0000   No Product String
2       hubd        5     hub             050d.0234   No Product String
3       ehci        1     pci1735,e0      0000.0000   No Product String
4       uhci        0     pci1028,151     0000.0000   No Product String
5       uhci        1     pci1028,151     0000.0000   No Product String
6       uhci        2     pci1028,151     0000.0000   No Product String
7       uhci        3     pci1028,151     0000.0000   No Product String
8       ohci        0     pci1735,35      0000.0000   No Product String
9       hubd        0     hub             0451.2046   No Product String
a       ohci        1     pci1735,35      0000.0000   No Product String
b       hubd        1     hub             0451.2046   No Product String
c       usb_mid     0     device          050d.0105   Belkin OmniView KVM Switch
d       hid         2     mouse           413c.3010   No Product String
e       usbsacm     4     device          12d1.1001   HUAWEI Mobile
>  ::prtusb -i e -v
INDEX   DRIVER      INST  NODE            VID.PID     PRODUCT           
e       usbsacm     4     device          12d1.1001   HUAWEI Mobile

Device Descriptor
{
    bLength = 0x12
    bDescriptorType = 0x1
    bcdUSB = 0x110
    bDeviceClass = 0
    bDeviceSubClass = 0
    bDeviceProtocol = 0
    bMaxPacketSize0 = 0x40
    idVendor = 0x12d1
    idProduct = 0x1001
    bcdDevice = 0
    iManufacturer = 0x1
    iProduct = 0x2
    iSerialNumber = 0x4
    bNumConfigurations = 0x1
}
    -- Active Config Index 0
    Configuration Descriptor
    {
        bLength = 0x9
        bDescriptorType = 0x2
        wTotalLength = 0x6c
        bNumInterfaces = 0x4
        bConfigurationValue = 0x1
        iConfiguration = 0x0
        bmAttributes = 0xa0
        bMaxPower = 0xfa
    }
        Interface Descriptor
>> More [<space>, <cr>, q, n, c, a] ?
==================================