### Adım 1: Entegre Hizmet Yönlendiricisi (ISR) Pratik

**Teorik Mantık:** ISR, bir cihazın sadece **yönlendirme** değil, aynı zamanda **güvenlik, ses ve switch (anahtarlama)** gibi birden fazla işlevi tek bir cihazda sunmasıdır.

#### Packet Tracer Test Hedefi: Modülerlik ile Entegrasyon

Packet Tracer'da, 1941'in **modül ekleyerek** nasıl "entegre hizmet" verdiğini göreceğim.

| Gerekli Hizmet | Gerçek Hayattaki Modül Tipi (Teori) | Packet Tracer'da Temsili ve Amaç |
| :--- | :--- | :--- |
| **Anahtarlama (Switching)** | EtherSwitch EHWIC/HWIC | Router'a LAN portları ekleyerek **switch** görevi yüklemek. |
| **Ses (Voice)** | VIC/VWIC | IP telefon gibi cihazları desteklemek için bağlantı portları eklemek. |
| **WAN/Mobilite** | HWIC-3G/4G | Router'a mobil internet (3G/4G) veya seri WAN bağlantısı eklemek. |

#### Packet Tracer'da Yapılacak Detaylı Testler (Aşama 1)

1.  **Cihaz Ekleme:** Packet Tracer'da bir **1941** router'ı ekle.
   
2.  **Güç Kapatma (Gerçekçilik):** Router'a tıkla, **"Physical" (Fiziksel)** sekmesine git ve açma/kapama düğmesini kullanarak cihazı kapat. *(Gerçek hayatta modül takmadan önce bunu yapmak gerekir.)*

3.  **Switch Hizmetini Ekleme (Anahtarlama):**
      * Modüller listesinden **HWIC-4ESW** modülünü bul (Bu, 4 portlu Ethernet **Switch** modülüdür).
      * Bu modülü router'ın boş slotlarından birine tak (Genellikle **Slot 0** veya **Slot 1**).

4.  **Seri WAN Hizmetini Ekleme (Geniş Alan Ağı):**
      * Modüller listesinden bir **HWIC-2T** modülü bul (Bu, **Seri WAN** bağlantıları için kullanılan bir modüldür).
      * Bu modülü başka bir boş slota tak.

5.  **Gücü Açma:** Router'ın açma/kapama düğmesine basarak gücü tekrar aç.

6.  **Doğrulama (Komut Satırı):** Router'ın **CLI**'sına (Komut Satırı Arayüzü) gir ve şu komutu yaz:
    ```
    show ip interface brief
    ```
    *(Bu komutun çıktısında artık sadece GigabitEthernet portlarını değil, eklediğin **Switch** portlarını ve **Serial** portlarını da görmen gerekir.
    Bu, router'ın artık hem yönlendirici, hem switch hem de WAN cihazı (ISR) olarak çalıştığını kanıtlar.)*

-----

Ben 1941 router ekledikten sonra CLI kısmından varsayılan portları gördüm:

```
Router>show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     unassigned      YES unset  administratively down down 
GigabitEthernet0/1     unassigned      YES unset  administratively down down 
Vlan1                  unassigned      YES unset  administratively down down
```

Daha sonra HWIC-4ESW & HWIC-2T modüllerini ekledim.

![img](https://i.ibb.co/rRHkzhSj/1-IRS-Mod-l.png)

Ve kontrol ettim tekrar:

```
Router>show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     unassigned      YES unset  administratively down down 
GigabitEthernet0/1     unassigned      YES unset  administratively down down 
Serial0/0/0            unassigned      YES unset  administratively down down 
Serial0/0/1            unassigned      YES unset  administratively down down 
FastEthernet0/1/0      unassigned      YES unset  up                    down 
FastEthernet0/1/1      unassigned      YES unset  up                    down 
FastEthernet0/1/2      unassigned      YES unset  up                    down 
FastEthernet0/1/3      unassigned      YES unset  up                    down 
Vlan1                  unassigned      YES unset  administratively down down
```

Modüller hemen entegre edilmişti.
