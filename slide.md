# WireGuard簡介
<img src='https://www.wireguard.com/img/wireguard.svg' alt='WireGuard logo' style='background-color: white' />

<table>
    <tr>
        <th>簡報</th>
        <td><a href='http://bit.ly/itzxa-wireguard-introduction' target='_blank'>http://bit.ly/itzxa-wireguard-introduction</a></td>
    </tr>
    <tr>
        <th>源碼</th>
        <td><a href='http://bit.ly/itzxa-wireguard-intro-source' target='_blank'>http://bit.ly/itzxa-wireguard-intro-source</a></td>
    </tr>
</table>

---

## WireGuard特性<br><small>Characteristics</small>
* 基於<ruby>公私鑰加密<rp>(</rp><rt>類似SSH</rt><rp>)</rp></ruby>，每個<ruby>對等點<rp>(</rp><rt>peer</rt><rp>)</rp></ruby>之間各有一組屬於自己的公私鑰
* 基於UDP通訊協定
* 跟OpenVPN、IPsec、PPTP等其他實作相比效能卓越

---

## 效能比較<br><small>Performance Comparison</small>
![](https://i.imgur.com/p6BBqFt.png)

---

## 效能比較<br><small>Performance Comparison</small>
![](https://i.imgur.com/mCfbh9G.png)

---

## 目標<br><small>Goals</small>
* 外網可以存取內網私有IP地址的服務
* 外網可以透過內網 gateway 存取<ruby>網際網路<rt><ruby>互联网<rt>翻墙</rt></ruby></rt></ruby>

---

## 安裝<br><small>Installation</small>
* WireGuard本身為核心層級的實作，需要建構並載入第三方<ruby>核心模組<rp>(</rp><rt>kernel module</rt><rp>)</rp></ruby>
* 另有其他 userspace 實作，如 CloudFlare 的 [BoringTun](https://github.com/cloudflare/boringtun)

Windows/Mac/iOS/Android皆有移植版本

[Installation - WireGuard](https://www.wireguard.com/install)

---

## 設定<br><small>Setup</small>
![](https://i.imgur.com/MK5SPto.gif)

---

### 配置範例<br><small>Configuration example</small>
* 內網網段：192.168.1.0/24
* 內網對外域名：itzxa.com
* 內網<ruby>對等點<rt>peer</rt></ruby>：
    * IP地址：192.168.1.127, 192.168.2.254
    * 主機名稱：internal
* 外網主機名稱：external
* VPN tunnel 網段：192.168.2.0/24

---

### 產生內網<ruby>對等點<rp>(</rp><rt>peer</rt><rp>)</rp></ruby>公私鑰組合
```
internal $ wg genkey > private.key
internal $ wg pubkey < private.key > public.key
internal $ cat private.key public.key
qMSa7W71pDChtarPTqVqSPdmJEagDMbXD4+OS0sNhVY=
NsjvQcy/e+dMP8zM1GSjzxTDnE31z4gwnemvQJTBimk=
```

---

### 產生外網<ruby>對等點<rp>(</rp><rt>peer</rt><rp>)</rp></ruby>公私鑰組合
以下為GNU/Linux系統的命令，其他系統請參閱WireGuard支援軟體的指示

```
external $ wg genkey > private.key
external $ wg pubkey < private.key > public.key
external $ cat private.key public.key
6Mayo53eXWkbUezMZG3On7FqLxIf1ZC/zpd8eqd+50k=
6boi5zHwfEhbX+XMUKe12f4lQ9v78LjpEcGK0hGU3RQ=
```

---

### 設定內網<ruby>對等點<rp>(</rp><rt>peer</rt><rp>)</rp></ruby>WireGuard網路界面
編輯 /etc/wireguard/wg0.conf 的 Interface 區段：

```ini
[Interface]
Address = 192.168.2.254/24
PrivateKey = qMSa7W71pDChtarPTqVqSPdmJEagDMbXD4+OS0sNhVY=
ListenPort = 51840

[Peer]
PublicKey = 6boi5zHwfEhbX+XMUKe12f4lQ9v78LjpEcGK0hGU3RQ=
AllowedIPs = 192.168.2.1/32

```

---

### 設定外網<ruby>對等點<rp>(</rp><rt>peer</rt><rp>)</rp></ruby>WireGuard網路界面
編輯 /etc/wireguard/wg0.conf 的 Interface 區段：

```ini
[Interface]
Address = 192.168.2.1/24
PrivateKey = qMSa7W71pDChtarPTqVqSPdmJEagDMbXD4+OS0sNhVY=
ListenPort = 12345

[Peer]
PublicKey = NsjvQcy/e+dMP8zM1GSjzxTDnE31z4gwnemvQJTBimk=
Endpoint = itzxa.com:51820
AllowedIPs = 192.168.2.0/24, 192.168.1.0/24
```

其他平台請參閱各平台WireGuard支援軟體的指示

---

### 啟用 / 停用<ruby>對等點<rp>(</rp><rt>peer</rt><rp>)</rp></ruby>WireGuard網路界面
```bash
$ sudo wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip address add 192.168.2.1/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] ip route add 192.168.1.0/24 dev wg0
$ sudo wg-quick down wg0
[#] ip link delete dev wg0
```

---

### 查看狀態
```
interface: wg0
  public key: 6boi5zHwfEhbX+XMUKe12f4lQ9v78LjpEcGK0hGU3RQ=
  private key: (hidden)
  listening port: 12345

peer: NsjvQcy/e+dMP8zM1GSjzxTDnE31z4gwnemvQJTBimk=
  endpoint: 192.168.1.127:51820
  allowed ips: 192.168.2.0/24, 192.168.1.0/24
  latest handshake: Now
  transfer: 92 B received, 148 B sent
```

---

## 檢查是否 tunnel 有通
```bash
$ ping 192.168.2.254
PING 192.168.2.254 (192.168.2.254) 56(84) bytes of data.
64 bytes from 192.168.2.254: icmp_seq=1 ttl=64 time=73.3 ms
64 bytes from 192.168.2.254: icmp_seq=2 ttl=64 time=25.3 ms
64 bytes from 192.168.2.254: icmp_seq=3 ttl=64 time=31.0 ms
64 bytes from 192.168.2.254: icmp_seq=4 ttl=64 time=32.8 ms
^C
--- 192.168.2.254 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 7ms
rtt min/avg/max/mdev = 25.321/40.589/73.252/19.059 ms
```

---

## 設定VPN tunnel→內網路由<br><small>Setup routing</small>
於內網Wireguard<ruby>對等點<rp>(</rp><rt>peer</rt><rp>)</rp></ruby>編輯 /etc/sysctl.d/99-zz-wireguard-settings.conf (Ubuntu)：

```
# Enable package forwarding
net.ipv4.ip_forward=1

# Enable ProxyARP
net.ipv4.conf.all.proxy_arp = 1
```

重讀核心參數(Ubuntu)

```
sudo service procps start
```

---

## 設定VPN tunnel←內網路由<br><small>Setup routing(reverse)</small>
![](https://i.imgur.com/Z0IWL4y.png)

---

## 測試<br><small>Verification</small>
![](https://i.imgur.com/ZHXyunY.gif)

---

### 測試是否有透過VPN通到外網
```
external $ tracepath -n 168.95.1.1
 1?: [LOCALHOST]                      pmtu 1420
 1:  192.168.2.254                               63.216ms 
 1:  192.168.2.254                               30.588ms 
 2:  192.168.1.254                               37.240ms 
 3:  168.95.98.254                               33.943ms 
 4:  168.95.24.42                                29.387ms 
 5:  220.128.3.134                               44.694ms asymm  9 
 6:  220.128.3.189                               42.765ms asymm  7 
 7:  220.128.1.249                               38.236ms 
 8:  210.59.204.229                              37.682ms 
^C
external $ 
```

---

## 監控<br><small>Monitoring</small>
![http://www.funnyanimalsite.com/pictures/Monitoring_Cat.jpg](https://i.imgur.com/PnFf03u.png)

---

* 受到簡單的架構以及連線無狀態性(stateless)的影響，使用WireGuard本身監控一個使用者是否正在使用VPN是不可能的
* 但可搭配其他工具實現（如防火牆）
* WireGuard仍有一定程度的運行紀錄可以審閱，參閱：
    * [Dynamic debug — The Linux Kernel documentation](https://www.kernel.org/doc/html/latest/admin-guide/dynamic-debug-howto.html)
    * net_dbg_函式輸出的運行紀錄可以透過安裝除錯版本的核心模組來啟用

---

### 啟用 Dynamic debug
1. 停用 wg-quick 服務
1. 卸載 wireguard 作業系統核心模組
1. 以SuperUser身份編輯 `/etc/modprobe.d/wireguard.conf` 設定檔：

    ```
    options wireguard dyndbg=+flmpt
    ```

1. 重新載入 wireguard 作業系統核心模組
1. 重新啟用 wg-quick 服務

---

### 安裝除錯版本的 wireguard 作業系統核心模組
1. 停用 wg-quick 服務
1. 卸載 wireguard 作業系統核心模組
1. 以SuperUser身份編輯 `/etc/dkms/wireguard.conf` 設定檔：

    ```
    MAKE[0]="make -C ${kernel_source_dir} SUBDIRS=${dkms_tree}/${PACKAGE_NAME}/${PACKAGE_VERSION}/build CONFIG_WIREGUARD_DEBUG=y modules"
    ```

1. 重新安裝 wireguard-dkms 軟體包(Ubuntu)
1. 重新載入 wireguard 作業系統核心模組
1. 重新啟用 wg-quick 服務

---

```
[1707993.957550] [9441] wireguard:wg_receive_handshake_packet:158: wireguard: wg0: Receiving handshake initiation from peer 6 (1.200.48.72:19543)
[1707993.957554] [9441] wireguard:wg_packet_send_handshake_response:93: wireguard: wg0: Sending handshake response to peer 6 (1.200.48.72:19543)
[1707993.957752] <intr> wireguard:wg_noise_handshake_begin_session:795: wireguard: wg0: Keypair 1 created for peer 6
[1707993.989288] <intr> wireguard:wg_packet_consume_data_done:367: wireguard: wg0: Receiving keepalive packet from peer 6 (1.200.48.72:19543)
[1708050.945541] <intr> wireguard:wg_packet_send_keepalive:230: wireguard: wg0: Sending keepalive packet to peer 6 (1.200.48.72:19543)
[1708090.880061] <intr> wireguard:wg_packet_send_keepalive:230: wireguard: wg0: Sending keepalive packet to peer 6 (1.200.48.72:19543)
[1708110.847462] <intr> wireguard:wg_packet_send_keepalive:230: wireguard: wg0: Sending keepalive packet to peer 6 (1.200.48.72:19543)
[1708120.769582] [9711] wireguard:wg_receive_handshake_packet:158: wireguard: wg0: Receiving handshake initiation from peer 6 (1.200.48.72:19543)
[1708120.769586] [9711] wireguard:wg_packet_send_handshake_response:93: wireguard: wg0: Sending handshake response to peer 6 (1.200.48.72:19543)
[1708120.769785] <intr> wireguard:wg_noise_handshake_begin_session:795: wireguard: wg0: Keypair 2 created for peer 6
[1708120.825587] <intr> wireguard:wg_packet_consume_data_done:367: wireguard: wg0: Receiving keepalive packet from peer 6 (1.200.48.72:19543)
```

---

## 待辦事項<br><small>TODO</small>
* 研究使用其他userspace實作的可能性（容器化可能）
* 評估實用性
    * 管理便利度
    * 翻牆

---

## 延伸閱讀<br><small>Additional info</small>
由 Eric 所撰寫的一系列文章：

* [Such geek. Wow. – Wireguard - Part One (Installation)](https://www.ericlight.com/wireguard-part-one-installation.html)
* [Such geek. Wow. – Wireguard - Part Two (VPN routing)](https://www.ericlight.com/wireguard-part-two-vpn-routing.html)
* [Such geek. Wow. – Wireguard - Part Three (Troubleshooting)](https://www.ericlight.com/wireguard-part-three-troubleshooting.html)
* [Such geek. Wow. – New things I didn't know about Wireguard](https://www.ericlight.com/new-things-i-didnt-know-about-wireguard.html)

---

{%youtube CejbCQ5wS7Q %}

原作者於[Linux Plumbers Conference](https://linuxplumbersconf.org/) 2018的演說

---

## ...Q&A?
![](https://i.imgur.com/oKM7sfa.png)

---

![200 OK](https://i.imgur.com/ODxWOWS.png)
