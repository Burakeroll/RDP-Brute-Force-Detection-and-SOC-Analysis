SCADA Network Brute-Force Detection & SOC Monitoring Lab

Bu proje, bir SCADA ağı üzerinde gerçekleştirilen RDP brute-force saldırısının; uç nokta logları (Windows), savunma bileşenleri (Wazuh, Snort IDS, FortiGate Firewall) ve merkezi izleme sistemi (Splunk Enterprise) tarafından nasıl algılandığını incelemek amacıyla oluşturulmuş çok katmanlı bir SOC simülasyon ortamıdır.

1. Kurulum Sırasında Dikkat Edilmesi Gerekenler

FortiGate kurulumu yapılırken cihaz varsayılan ayarlarıyla kullanılmalı, herhangi bir değişiklik yapılmamalıdır.

FortiGate üzerinde üç farklı ağ segmenti oluşturulmalıdır:

Operator LAN (10.0.0.0/24)

OT LAN (10.10.10.0/24)

SOC / Monitoring LAN (10.20.20.0/24)

Wazuh Server ve Splunk Enterprise Ubuntu 22.04 Desktop üzerine kurulmalıdır. Kaynak sorunu yaşanıyorsa, her iki bileşen aynı makine üzerinde de çalıştırılabilir.

Ubuntu 24, uyumluluk sorunları nedeniyle tercih edilmemelidir.

Windows 10 Victim makinesine Splunk Universal Forwarder kurulmalı ve Splunk Enterprise ile bağlantı sağlanmalıdır.

Snort IDS kurulduktan sonra Wazuh Agent ile bağlantı yapılandırılmalıdır.

2. Ağ Topolojisi

Laboratuvar ortamı aşağıdaki üç segmentten oluşmaktadır:

Operator LAN (10.0.0.0/24)

Windows 10 (SCADA Remote Operator): 10.0.0.10

Kali Linux (RDP Attacker): 10.0.0.20

FortiGate Port1: 10.0.0.1

OT LAN (10.10.10.0/24)

Windows 10 Victim (SCADA + Splunk UF): 10.10.10.10

Ubuntu 22.04 (Snort IDS + Wazuh Agent): 10.10.10.20

FortiGate Port2: 10.10.10.1

SOC / Monitoring LAN (10.20.20.0/24)

Wazuh Server: 10.20.20.20

Splunk Enterprise: 10.20.20.30

FortiGate Port3: 10.20.20.1

Internet / WAN

FortiGate WAN portu üzerinden sağlanır.

Topoloji görseli README içerisinde aşağıdaki şekilde kullanılabilir:

![Network Topology](images/topology.png)

3. Ortamda Kullanılan Bileşenler
FortiGate Firewall

Üç LAN segmenti arasında yönlendirme sağlar.

Snort ve Wazuh tarafından üretilen uyarıları doğrulamak amacıyla trafik logları kullanılır.

Kali Linux (Attacker)

Hydra aracıyla RDP brute-force saldırısı gerçekleştirilir.

Saldırgan IP adresi 10.0.0.20'dir.

Windows 10 (Victim – SCADA)

Brute-force saldırısının hedefidir.

4624 ve 4625 olay kayıtları Splunk Universal Forwarder üzerinden Splunk Enterprise’a iletilir.

Ubuntu 22.04 – Snort IDS + Wazuh Agent

Snort IDS saldırı imzalarını tespit eder.

Wazuh Agent hem Snort hem de sistem loglarını Wazuh Server’a iletir.

Wazuh Server

Tüm agent loglarını toplar ve analiz eder.

Host-based ve network-based uyarıları Splunk’a aktarır.

Splunk Enterprise

Tüm logların toplandığı, sorgulandığı ve dashboard’ların oluşturulduğu ana izleme bileşenidir.

4. Saldırı Senaryosu (Görev-1)

Saldırı, Operator LAN üzerinde bulunan Kali makinesinden başlatılmaktadır.

Kullanıcı ve parola wordlist dosyaları hazırlanır.

Hydra komutu ile RDP brute-force saldırısı gerçekleştirilir:

hydra -L userlist.txt -P passlist.txt rdp://10.10.10.10


“burak / abcxyz” kimlik bilgisi başarıyla tespit edilir.

Windows Event Logs üzerinden 4625 (başarısız) ve 4624 (başarılı) olaylar Splunk’a iletilir.

Başarısız denemeler ile tek başarılı giriş arasındaki örüntü brute-force saldırısını doğrular.

5. Savunma Loglarının Doğrulanması (Görev-2)

Bu aşamada saldırıya ilişkin loglar üç farklı katmanda incelenir:

Windows Security Logs

Wazuh Alerts (Snort IDS dahil)

FortiGate Firewall Logs

SPL korelasyon sorguları ile:

Saldırgan IP’nin tüm katmanlarda görülüp görülmediği

IDS, Firewall ve Windows loglarının zaman açısından örtüşüp örtüşmediği

değerlendirilir.

6. SOC Dashboard (Görev-3)

Splunk üzerinde oluşturulan dashboard üç temel panelden oluşmaktadır:

Panel 1 – Zaman Bazlı Saldırı Akışı

Dakika bazında başarılı ve başarısız giriş denemeleri görüntülenir.

Saldırının yoğunlaştığı zaman aralıkları belirlenir.

Panel 2 – Alarm Durumu (Wazuh)

Alarm seviyelerinin (Level 3, Level 5, Level 7 vb.) dağılımı grafik olarak sunulur.

Kritik alarm sayısı gösterilir.

Panel 3 – SCADA Kullanıcı Hedef Analizi

En çok denenmiş kullanıcı adları pie chart ile gösterilir.

Saldırganın odaklandığı hesaplar hızlı şekilde tespit edilir.