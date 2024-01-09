# K230 音视频编码

- [K230视频编解码API参考文档](https://github.com/kendryte/k230_docs/blob/main/zh/01_software/board/mpp/K230_%E8%A7%86%E9%A2%91%E7%BC%96%E8%A7%A3%E7%A0%81_API%E5%8F%82%E8%80%83.md)
- [视频讲解](https://riscv-edu.cn/course/230/replay/6374)

> **以下两个实验注意连接网线， 保证开发板和主机处于同一局域网下**

或者使用[@linglan111](https://github.com/linglan111)同学编写的连接脚本将开发板连接至wifi
```bash
#!/bin/sh

# Enable wlan0 interface
ifconfig wlan0 up

# Start wpa_supplicant with the specified configuration file
wpa_supplicant -D nl80211 -i wlan0 -c /etc/wpa_supplicant.conf -B

# Scan for available networks
wpa_cli -i wlan0 scan -B

# Display scan results
wpa_cli -i wlan0 scan_result

# Add a new network
wpa_cli -i wlan0 add_network

# Set the network parameters (replace ssid and psk with your actual values)
wpa_cli -i wlan0 set_network 1 psk '"11111111"'
wpa_cli -i wlan0 set_network 1 ssid '"Redmi Note 11T Pro"'

# Select the configured network
wpa_cli -i wlan0 select_network 1

# Obtain an IP address using udhcpc
udhcpc -i wlan0 -q
```
- 使用方法
1. 将net.sh复制进SD卡
    - 修改wifi密码和wifi名
        ```bash
        wpa_cli -i wlan0 set_network 1 psk '"11111111"' # 密码
        wpa_cli -i wlan0 set_network 1 ssid '"Redmi Note 11T Pro"' # wifi
        ```
2. 进入linux小核，执行：
```bash
cd /sharefs
sh net.sh
```
