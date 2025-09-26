### Yüksek Performans ve Gecikme Analizi

**Teori:** 1941 Yönlendirici, birden fazla hizmet yükü altında bile yüksek hızlı WAN ortamlarında (25 Mbps'e kadar) performans düşüşü yaşatmamalıdır.
Ama bu Packet Tracer'da nasıl oluyor merak ettim.

Bu testte, yönlendiricinin üzerine **yüksek trafik yükü** atacağız ve **gecikme süresini (latency)** ölçerek performansını değerlendireceğiz.

#### **Adım 1: Temel Topolojiyi Kurma**

1.  **Cihazları Ekle:** Çalışma alanına iki adet **Cisco 1941** (Router1 ve Router2), bir adet **PC** (PC1) ve bir adet **PC** (PC2) ekle.
2.  **Modül Kurulumu:** Her iki router'ı da kapat. **HWIC-2T** (Seri Port Modülü) bul ve her iki router'ın da **Slot 0**'ına tak. Router'ları aç.
3.  **Kablolama:**
      * **WAN Bağlantısı:** **Seri DCE Kablosu** ile Router1'in **Serial0/0/0** portunu, Router2'nin **Serial0/0/0** portuna bağla.
      * **LAN Bağlantıları:** **Düz Kablo** ile PC1'i Router1'in **GigabitEthernet0/1** portuna bağla. PC2'yi Router2'nin **GigabitEthernet0/1** portuna bağla.

#### **Adım 2: Temel Yapılandırma (IP Adresleme ve Yönlendirme)**

##### **Router2 Yapılandırması**

```cli
Router>enable
Router#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#interface serial0/0/0
Router(config-if)#ip address 10.0.0.2 255.255.255.252
Router(config-if)#no shutdown
%LINK-5-CHANGED: Interface Serial0/0/0, changed state to down
Router(config-if)#exit
Router(config)#interface gigabitEthernet0/1
Router(config-if)#ip address 192.168.2.1 255.255.255.0
Router(config-if)#no shutdown 
Router(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
Router(config-if)#end
```

##### **Router1 Yapılandırması**

```cli
Router>enable
Router#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#interface serial0/0/0
Router(config-if)#ip address 10.0.0.1 255.255.255.252
Router(config-if)#clock rate 128000
Router(config-if)#no shutdown
Router(config-if)#
%LINK-5-CHANGED: Interface Serial0/0/0, changed state to up
Router(config-if)#exit
Router(config)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface Serial0/0/0, changed state to up
Router(config)#interface gigabitEthernet0/1
Router(config-if)#ip address 192.168.1.1 255.255.255.0
Router(config-if)#no shutdown
Router(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
Router(config-if)#end
```

##### **Yönlendirme (Statik Route)**

```cli
// Router1: Router2'nin ağına nasıl ulaşacağını söyle

Router(config)#ip route 192.168.2.0 255.255.255.0 Serial0/0/0
%Default route without gateway, if not a point-to-point interface, may impact performance

Router(config)#// Router2: Router1'in ağına nasıl ulaşacağını söyle
Router2(config)#ip route 192.168.1.0 255.255.255.0 Serial0/0/0
```

##### **PC Yapılandırması**

  * **PC1:** IP: `192.168.1.10`, Gateway: `192.168.1.1`
  * **PC2:** IP: `192.168.2.10`, Gateway: `192.168.2.1`

#### **Adım 3: Performans Ölçümü ve Yük Testi**

1.  **Kontrol Ping'i:** PC1'den PC2'ye bir ping at (`ping 192.168.2.10`). Gecikme süresini **(Time)** not et. (Genellikle 1ms olmalı).

```
C:\>ping 192.168.2.1

Pinging 192.168.2.1 with 32 bytes of data:

Reply from 192.168.2.1: bytes=32 time=21ms TTL=254
Reply from 192.168.2.1: bytes=32 time=2ms TTL=254
Reply from 192.168.2.1: bytes=32 time=23ms TTL=254
Reply from 192.168.2.1: bytes=32 time=22ms TTL=254

Ping statistics for 192.168.2.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 2ms, Maximum = 23ms, Average = 17ms
```

2.  **Hizmet Yükü Oluşturma:** Router1'in CLI'sına girerek, Router2'ye **büyük paketlerle** sürekli ping gönder. Bu, **hizmet yükünü** simüle eder.

    ```cli
    Router1#ping
    Target IP address: 192.168.2.1
    Repeat count [5]: 1000             // 1000 paket gönder
    Datagram size [100]: 1500         // Büyük paket boyutu (yük yaratır)
    Extended commands [n]: (Enter)
    Sweep range of sizes [n]: (Enter)
    ```

3.  **Yük Altında Tekrar Ölçme:** **Router1 bu büyük pingleri gönderirken**, hemen **PC1'e geri dön** ve PC2'ye tekrar normal ping at (`ping 192.168.2.10`).

```
C:\>ping 192.168.2.1

Pinging 192.168.2.1 with 32 bytes of data:

Reply from 192.168.2.1: bytes=32 time=26ms TTL=254
Reply from 192.168.2.1: bytes=32 time=14ms TTL=254
Reply from 192.168.2.1: bytes=32 time=23ms TTL=254
Reply from 192.168.2.1: bytes=32 time=36ms TTL=254

Ping statistics for 192.168.2.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 14ms, Maximum = 36ms, Average = 24ms

C:\>
```

Teoriye göre, router'ın **gömülü şifreleme hızlandırıcısı** ve **çok çekirdekli işlemcisi** sayesinde, PC1'den atılan ikinci ping'in gecikme süresi (Adım 3), 
ilk ping'in gecikme süresiyle **aynı kalmalı** veya sadece **çok az** artmalıdır.

Bu, 1941'in **hizmet yükü altında bile yüksek performansı koruduğunu** simülasyonda kanıtlar ama gerçekte de test etmek lazım asıl.
