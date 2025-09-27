## Kimlik Yönetimi (AAA) ve Uzak Erişim (A'dan Z'ye)

**Teori:** Cisco 1941, sadece bir parola yerine **kullanıcı adı/şifre** ile oturum açmayı zorunlu kılarak **uzaktan erişimi (mobilite)** güvence altına alır.

### **Adım 1: Topolojiyi Kurma ve IP Adresleme**

| Cihaz | Port | IP Adresi | Açıklama |
| :--- | :--- | :--- | :--- |
| **Router1 (1941)** | Gi0/1 | `192.168.1.1` | LAN Gateway |
| **PC** | LAN Portu | `192.168.1.10` | Uzak Yönetim İstemcisi |

1.  **Cihazları Ekle:** Bir adet **Cisco 1941 Router** ve bir adet **PC** ekle.
2.  **Kablolama:** **Düz Kablo** kullanarak PC'yi Router1'in **GigabitEthernet0/1** portuna bağla.
3.  **PC'yi Yapılandır:** PC'ye `192.168.1.10` IP adresini ve `192.168.1.1` Gateway adresini ver.
4.  **Router1 Temel Yapılandırma:**
    
          ```cli
          Router>en
          Router#conf t
          Enter configuration commands, one per line.  End with CNTL/Z.
          Router(config)#interface GigabitEthernet0/1
          Router(config-if)#ip address 192.168.1.1 255.255.255.0
          Router(config-if)#no shutdown
          
          Router(config-if)#
          %LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to up
          
          %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
          
          Router(config-if)#end
          ```

### **Adım 2: AAA'yı Etkinleştirme ve Yerel Kullanıcı Tanımlama**

Bu, router'ın artık merkezi kimlik yönetimi modeline geçtiğini ilan eden komutlardır.

| Komut | Mantığı | Detay |
| :--- | :--- | :--- |
| `aaa new-model` | **AAA'yı Aktifleştirir** | Bu komutu girer girmez, router'ın tüm eski erişim mekanizmaları devre dışı kalır. |
| `username admin privilege 15 secret cisco123` | **Kullanıcı Tanımlar** | `privilege 15` ile **en yüksek yetkiyi** atar. |

```cli
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#aaa new-model	
Router(config)#username admin privilege 15 secret cisco123
```

### **Adım 3: Doğrulama (Authentication) Kuralını Tanımlama**

Bu kural, router'a kullanıcı girişi (login) yapıldığında kimliği nerede arayacağını söyler.

| Komut | Mantığı | Detay |
| :--- | :--- | :--- |
| `aaa authentication login DEFAULT local` | **Giriş Yöntemini Belirler** | **`login`**: Tüm giriş denemelerinde tetiklenir. **`DEFAULT`**: Kuralı tüm hatlara uygula. **`local`**: Kullanıcı adı/şifreyi router'ın kendi veritabanında ara. |

```cli
Router(config)# aaa authentication login DEFAULT local
```

### **Adım 4: Kuralı Erişim Hatlarına Atama**

Bu adım, kuralın fiziksel ve sanal erişim noktalarında aktifleşmesini sağlar.

| Hat | Komut | Mantığı |
| :--- | :--- | :--- |
| **Konsol (Fiziksel)** | `line console 0` | Router'a konsol kablosuyla bağlanan kullanıcıları hedefler. |
| **VTY (Uzak/Telnet)** | `line vty 0 4` | Telnet veya SSH ile uzaktan bağlanan 5 kullanıcıyı hedefler. **Bu, PC testimiz için kritiktir.** |

```cli
Router(config)# line console 0
Router(config-line)# login authentication DEFAULT // Konsol erişimine AAA'yı uygula
Router(config-line)# exit

Router(config)# line vty 0 4
Router(config-line)# login authentication DEFAULT // Uzak erişime AAA'yı uygula
Router(config-line)# exit

Router(config)# end
Router# write memory
```

-----

## Doğrulama

### **A. Konsol (Yerel) Doğrulama**

1.  Router CLI'sını kapatıp tekrar aç veya `exit` yazıp çıkış yap.
2.  **Gözlem:** Router, artık sadece bir şifre yerine **`Username: admin`** ve **`Password: cisco123`** isteyecektir.

```
Username: admin
Password: 
Router>
```

### **B. Telnet (Uzak Erişim) Doğrulama**

1.  PC'ye git, **Command Prompt** (Komut İstemi) aç.
2.  Router'a bağlanmaya çalış: `telnet 192.168.1.1`

**Gözlem:** AAA'nın VTY hattına uygulanması sayesinde, PC ekranında aynı şekilde **`Username: admin`** ve **`Password: cisco123`** isteği belirecektir.
