Driver-USB-modem-HUAWEI-E1550
=============================

USB modem HUAWEI E1550
Driver к USB-modem Hauwei Mobile Broadband E1550


СКАЧАЙ driver HUAWEI switch USB-modem
Как оказалось, для перевода данного устройства в режим модема дополнительные программы не требуются. Достаточно из контекстного меню его иконки на рабочем столе выбрать "Извлечь том". Затем надо выполнить команду
update_drv -a -i '"usb19d2,2003"' usbsacm
и в /dev/term/ появятся желанные файлы устройств.(keremet)
после перейдите в директорию и запустите скомпилированный исполняемый скрипт
Код: [Выделить]
cd .../USB-modem
#./switch_huawei
использование Билайновских модемов в OpenSolaris'e. Оказалось что проблем в общем-то нет. Но есть несколько тонкостей. И так, по порядку:
$ uname -a
SunOS os-laptop 5.11 snv_101a i86pc i386 i86pc Solaris
Дальше надо указать какой драйвер для устройств использовать (Тут есть первая тонкость - модем определить в системе без перезагрузки с подключенным устройством у меня не получилось. Т.е. сначала подключаем модем - потом грузим систему. В Ubuntu 8.04 тоже была такая проблема, в Ubuntu 8.10 проблем нет).
$ pfexec update_drv -a -i "usbif12d1,1003.0.config1.0" usbsacm
$ pfexec update_drv -a -i "usbif12d1,1003.0.config1.1" usbsacm
$ pfexec devfsadm
Проверяем работает ли устройство:
$ pfexec tip /dev/term/0
connected
atz
OK
ati
Manufacturer: huawei
Model: E219
Revision: 11.310.16.11.00
IMEI: 359ХХХХХХХХХХХХ
+GCAP: +CGSM,+FCLASS,+DS

OK
Далее создаем файлы для ррр:
$ cat /etc/ppp/peers/beeline 
term/0
460800
idle 7200
lock
crtscts
modem
noauth
defaultroute
user removed
password 1
connect "/usr/bin/chat -Vv -f /etc/ppp/chat-beeline"
noipdefault
usepeerdns
novj
nodetach
$ cat /etc/ppp/chat-beeline 
ABORT BUSY
ABORT 'NO CARRIER'
ABORT ERROR
REPORT CONNECT
TIMEOUT 120
"" "AT&F"
OK "ATZ"
OK "ATE1"
OK "AT+CPIN=?"
OK "AT+COPS=?"
OK "AT+COPS?"
OK "AT&V"
OK "AT+CSQ"
OK 'AT+CFUN=?'
OK 'AT+CGDCONT=?'
OK 'AT+CPBR=?'
OK 'AT+CPBS=?'
OK 'AT+CGDCONT=1,"IP","home.beeline.ru"'
SAY "Calling Beeline \n"
OK "ATDT*99#"
TIMEOUT 120
CONNECT c
Если кто-то будет использовать просто gprs для выхода в интернет - то строчку "home.beeline.ru" надо изменить на "internet.beeline.ru" (теже изменения надо сделать если оператор МТС или Мегафон соответственно, конечно не забываем менять имя пользователя и пароль)
$ pfexec cat /etc/ppp/chap-secrets
"beeline" * "beeline"
НЕ ЗАБЫВАЕМ проверить маску и права на файл с паролями
$ ls -al /etc/ppp/chap-secrets 
-rw------- 1 root sys 2684 2008-11-12 02:45 /etc/ppp/chap-secrets
Следующий шаг возможно не очень корректен с точки зрения идеалогии, но для домашнего бука я закрыл на это глаза:
$ pfexec rm /etc/resolv.conf 
$ pfexec ln -s /etc/ppp/resolv.conf /etc/resolv.conf
Обязательно надо проверить указанно ли в системе использование ДНС:
$ cat /etc/nsswitch.conf | grep dns
hosts: files dns
Вот и настал момент истины:
$ pfexec pppd call beeline debug
Removed stale lock on /dev/term/0 (pid 3140)
serial speed set to 460800 bps
connect option: '/usr/bin/chat -Vv -f /etc/ppp/chat-beeline' started (pid 3209)
AT&F
OK
ATZ
OK
ATE1
OK
AT+CPIN=?
OK
AT+COPS=?
+COPS: (2,"Beeline","Beeline","25099",0),(1,"Beeline","Beeline","25099",2),(1,"MTS-RUS","MTS","25001",0),(1,"TELE2","TELE2","25020",0),(1,"MegaFon RUS","MegaFon","25002",0),,(0,1,3,4),(0,1,2)

OK
AT+COPS?
+COPS: 0,2,"25099",0

OK
AT&V
&C: 2; &D: 2; &F: 0; E: 1; L: 0; M: 0; Q: 0; V: 1; X: 0; Z: 0; S0: 0;
S3: 13; S4: 10; S5: 8; S6: 2; S7: 50; S8: 2; S9: 6; S10: 14; S11: 95;
+FCLASS: 0; +ICF: 3,3; +IFC: 2,2; +IPR: 115200; +DR: 0; +DS: 0,0,2048,6;
+WS46: 12; +CBST: 0,0,1;
+CRLP: (61,61,48,6,0),(61,61,48,6,1),(240,240,52,6,2);
+CV120: 1,1,1,0,0,0; +CHSN: 0,0,0,0; +CSSN: 0,0; +CREG: 0; +CGREG: 0;
+CFUN:; +CSCS: "IRA"; +CSTA: 129; +CR: 0; +CRC: 0; +CMEE: 0; +CGDCONT: (1,"IP","home.beeline.ru","0.0.0.0",0,0)
; +CGDSCONT: ; +CGTFT: ; +CGEQREQ: (1,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(2,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(3,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(4,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(5,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(6,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(7,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(8,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(9,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(10,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(11,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(12,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(13,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(14,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(15,4,0,0,0,0,2,0,"0E0","0E0",3,0,0),(16,4,0,0,0,0,2,0,"0E0","0E0",3,0,0)
; +CGEQMIN: ; +CGQREQ: ; +CGQMIN: ; ; +CGEREP: 0,0; +CGCLASS: "A";
+CGSMS: 3; +CSMS: 0; +CMGF: 0; +CSCA: "+79037011111",145; +CSMP: ,,0,0;
+CSDH: 0; +CSCB: 0,"",""; +FDD: 0; +FAR: 0; +FCL: 0; +FIT: 0,0; +ES: ,,;
+ESA: 0,,,,0,0,255,; +CMOD: 0; +CVHU: 1; ; +CPIN: ,; +CMEC: 0,0,0;
+CKPD: 1,1; +CGATT: 1; +CGACT: 0; +CPBS: "SM"; +CPMS: "SM","SM","SM";
+CNMI: 0,0,0,0,0; +CMMS: 2; +FTS: 0; +FRS: 0; +FTH: 3; +FRH: 3; +FTM: 96;
+FRM: 96; +CCUG: 0,0,0; +COPS: 0,2,""; +CUSD: 0; +CAOC: 1; +CCWA: 0;
+CLVL: 2; +CMUT: 0; +CPOL: 0,2,"",0,0,0; +CPLS: 0; +CTZR: 0; +CLIP: 0;
+COLP: 0; +CLIR: 0; ^PORTSEL: 0; ^CPIN: ,; ^ATRECORD: 0;
^FREQLOCK: 12501720,13166028^@ 

OK
AT+CSQ
+CSQ: 25,99

OK
AT+CFUN=?
+CFUN: (0-1,4-7),(0-1)

OK
AT+CGDCONT=?
+CGDCONT: (1-16),"IP",,,(0-2),(0-4)
+CGDCONT: (1-16),"PPP",,,(0-2),(0-4)
+CGDCONT: (1-16),"IPV6",,,(0-2),(0-4)

OK
AT+CPBR=?
+CPBR: (1-220),24,26

OK
AT+CPBS=?
+CPBS: ("SM","DC","FD","LD","MC","ME","RC","EN","ON")

OKCalling Beeline 

AT+CGDCONT=1,"IP","home.beeline.ru"
OK
ATDT*99#
CONNECTchat: Nov 12 02:51:10 CONNECT
Serial connection established.
serial speed set to 460800 bps
Using interface sppp0
Connect: sppp0 <--> /dev/term/0
sent [LCP ConfReq id=0x8b ]
rcvd [LCP ConfReq id=0x0 ]
sent [LCP ConfAck id=0x0 ]
rcvd [LCP ConfAck id=0x8b ]
sent [LCP Ident id=0x8c magic=0x12910ba5 "ppp-2.4.0b1 (Sun Microsystems, Inc.)"]
Authenticating to peer with standard CHAP
rcvd [LCP DiscReq id=0x1 magic=0xeb227c]
rcvd [CHAP Challenge id=0x1 <0a2ba7d51a0bfbd1a84396b844293491>, name = "UMTS_CHAP_SRVR"]
sent [CHAP Response id=0x1 <794420abd89b050094e26f920c5110c9>, name = "removed"]
rcvd [CHAP Success id=0x1 ""]
sent [IPCP ConfReq id=0x42 ]
sent [CCP ConfReq id=0x4b ]
rcvd [LCP ProtRej id=0x2 80 fd 01 4b 00 0f 1a 04 78 00 18 04 78 00 15 03 2f]
rcvd [IPCP ConfReq id=0x0 ]
sent [IPCP ConfAck id=0x0 ]
rcvd [IPCP ConfNak id=0x42 ]
sent [IPCP ConfReq id=0x43 ]
rcvd [IPCP ConfAck id=0x43 ]
local IP address 172.19.2.227
remote IP address 217.118.88.114
primary DNS address 217.118.66.244
secondary DNS address 193.232.88.17
И последняя проверочка:
$ ifconfig -a
sppp0: flags=10010008d1 mtu 1500 index 4
inet 172.19.2.227 --> 217.118.88.114 netmask ffff0000 
$ ping google.com
google.com is alive
P.S.: Статья результат прочтения форумов opensolaris.org

