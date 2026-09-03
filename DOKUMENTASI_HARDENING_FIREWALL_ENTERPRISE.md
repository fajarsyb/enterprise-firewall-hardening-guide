# DOKUMENTASI STANDAR HARDENING FIREWALL ENTERPRISE
## Panduan Komprehensif Arsitektur, Pengerasan Keamanan, Prosedur Operasional, dan Kepatuhan Audit
*Standar Acuan: CIS Firewall Benchmark | NIST SP 800-41 Rev. 1 | ISO/IEC 27001:2022 (Annex A.8.20, A.8.22) | PCI-DSS v4.0 (Requirement 1)*

---

## DAFTAR ISI

1. [BAB 1 — PENDAHULUAN](#bab-1--pendahuluan)
   * 1.1 [Latar Belakang Urgensi Hardening Firewall](#11-latar-belakang-urgensi-hardening-firewall)
   * 1.2 [Tujuan Dokumen](#12-tujuan-dokumen)
   * 1.3 [Target Audiens](#13-target-audiens)
   * 1.4 [Ruang Lingkup (Scope & Out-of-Scope)](#14-ruang-lingkup-scope--out-of-scope)
2. [BAB 2 — GLOSARIUM & TERMINOLOGI LENGKAP](#bab-2--glosarium--terminologi-lengkap)
   * 2.1 [Terminologi Dasar Firewall](#21-terminologi-dasar-firewall)
   * 2.2 [Terminologi Arsitektur & Zonasi](#22-terminologi-arsitektur--zonasi)
   * 2.3 [Terminologi Rule & Kebijakan Akses](#23-terminologi-rule--kebijakan-akses)
   * 2.4 [Terminologi Ancaman & Deteksi](#24-terminologi-ancaman--deteksi)
   * 2.5 [Terminologi Manajemen & Tata Kelola](#25-terminologi-manajemen--tata-kelola)
3. [BAB 3 — PRINSIP DASAR HARDENING](#bab-3--prinsip-dasar-hardening)
   * 3.1 [Prinsip Least Privilege](#31-prinsip-least-privilege)
   * 3.2 [Prinsip Default Deny](#32-prinsip-default-deny)
   * 3.3 [Prinsip Defense in Depth](#33-prinsip-defense-in-depth)
   * 3.4 [Prinsip Minimalisasi Attack Surface](#34-prinsip-minimalisasi-attack-surface)
   * 3.5 [Prinsip Fail Secure vs Fail Open](#35-prinsip-fail-secure-vs-fail-open)
   * 3.6 [Prinsip Segregation of Duties](#36-prinsip-segregation-of-duties)
4. [BAB 4 — CHECKLIST HARDENING TEKNIS](#bab-4--checklist-hardening-teknis)
   * 4.1 [Manajemen Akses Administratif](#41-manajemen-akses-administratif)
   * 4.2 [Kebijakan Rule/ACL](#42-kebijakan-ruleacl)
   * 4.3 [Manajemen NAT & Port Forwarding](#43-manajemen-nat--port-forwarding)
   * 4.4 [Logging & Monitoring](#44-logging--monitoring)
   * 4.5 [Manajemen Patch & Firmware](#45-manajemen-patch--firmware)
   * 4.6 [Proteksi terhadap Serangan Umum](#46-proteksi-terhadap-serangan-umum)
   * 4.7 [High Availability & Backup](#47-high-availability--backup)
5. [BAB 5 — PROSEDUR OPERASIONAL](#bab-5--prosedur-operasional)
   * 5.1 [Prosedur Perubahan Konfigurasi (Change Control Workflow)](#51-prosedur-perubahan-konfigurasi-change-control-workflow)
   * 5.2 [Prosedur Audit Berkala](#52-prosedur-audit-berkala)
   * 5.3 [Prosedur Review Rule (Rule Recertification)](#53-prosedur-review-rule-rule-recertification)
   * 5.4 [Prosedur Incident Response Terkait Firewall](#54-prosedur-incident-response-terkait-firewall)
6. [BAB 6 — TEMPLATE & CONTOH IMPLEMENTATIF](#bab-6--template--contoh-implementatif)
   * 6.1 [Template Dokumentasi Rule Firewall](#61-template-dokumentasi-rule-firewall)
   * 6.2 [Template Checklist Audit Kepatuhan](#62-template-checklist-audit-kepatuhan)
7. [BAB 7 — REFERENSI & STANDAR ACUAN](#bab-7--referensi--standar-acuan)
   * 7.1 [CIS Benchmark](#71-cis-benchmark)
   * 7.2 [NIST SP 800-41 Rev. 1](#72-nist-sp-800-41-rev-1)
   * 7.3 [ISO/IEC 27001:2022 Annex A](#73-isoiec-270012022-annex-a)
   * 7.4 [PCI-DSS v4.0 Requirement 1](#74-pci-dss-v40-requirement-1)

---

## BAB 1 — PENDAHULUAN

### 1.1 Latar Belakang Urgensi Hardening Firewall
Firewall adalah garis pertahanan pertama (*first line of defense*) dan pengendali gerbang (*gatekeeper*) utama dalam infrastruktur jaringan komputer. Dalam lanskap ancaman siber modern, mayoritas insiden peretasan dan kebocoran data (*data breach*) yang melibatkan infrastruktur enterprise bukan disebabkan oleh kegagalan enkripsi perangkat keras, melainkan oleh **salah konfigurasi (*misconfiguration*)**, lemahnya tata kelola akses administratif, keberadaan aturan usang yang terlalu longgar (*overly permissive rules*), serta ketiadaan audit berkala.

**Risiko Nyata Penggunaan Konfigurasi Bawaan (Default Configuration):**
* Kredensial default pabrik (seperti username `admin` dengan password kosong atau standar) mudah ditebak oleh botnet otomatis dalam hitungan detik setelah firewall terhubung ke Internet.
* Layanan manajemen berbasis teks terbuka (*cleartext*) seperti HTTP dan Telnet sering kali aktif secara default pada antarmuka, memungkinkan intersepsi kredensial melalui *Man-in-the-Middle*.
* Tidak adanya pembatasan alamat IP sumber yang boleh mengakses antarmuka Web GUI atau SSH membuka peluang serangan *brute force* masif dan eksploitasi kerentanan *zero-day* pada modul manajemen perimeter.

**Insiden Umum Akibat Firewall yang Tidak Di-hardening:**
1. **Penyusupan Perimeter via Zero-Day SSL-VPN/Web GUI:** Port manajemen terbuka ke publik (`0.0.0.0/0`) sehingga kerentanan autentikasi langsung dieksploitasi peretas untuk menanamkan *web shell*.
2. **Pergerakan Lateral (Lateral Movement) Ransomware:** Ketiadaan segmentasi internal (*East-West inspection*) menyebabkan infeksi malware dari satu laptop pegawai menyebar seketika ke seluruh server basis data datacenter melalui port SMB (TCP 445) dan RDP (TCP 3389).
3. **Penyelundupan Data (Data Exfiltration) via Aturan Any-Any:** Firewall mengizinkan akses keluar (*outbound*) dengan rule `Permit ANY`, sehingga malware bebas berkomunikasi dengan server Command & Control (C2) di Internet menggunakan port non-standar.

### 1.2 Tujuan Dokumen
Dokumen ini disusun untuk:
1. **Panduan Konfigurasi Praktis:** Memberikan pedoman teknis langkah-demi-langkah bagi administrator jaringan dalam mengonfigurasi dan mengamankan perangkat firewall.
2. **Acuan Standar Audit:** Menyediakan kriteria evaluasi yang terukur dan objektif bagi tim audit internal dan eksternal untuk menguji kepatuhan postur keamanan perimeter.
3. **Materi Onboarding Terstandar:** Menjadi manual operasional resmi untuk melatih staf baru di tim Network Engineering, Security Operations Center (SOC), dan Network Operations Center (NOC).

### 1.3 Target Audiens
Dokumen ini ditujukan kepada:
* **Network Engineer / Firewall Administrator:** Bertanggung jawab langsung atas konfigurasi, *provisioning*, manajemen rule, dan pemeliharaan perangkat firewall harian.
* **Security Analyst (SOC/Incident Responder):** Menggunakan dokumentasi ini untuk memahami arsitektur baseline jaringan, mengkorelasikan log ancaman, dan mengeksekusi isolasi darurat saat terjadi insiden.
* **Auditor IT & Tim Kepatuhan (Compliance):** Menggunakan checklist teknis untuk memvalidasi kepatuhan terhadap standar internasional (ISO 27001, PCI-DSS, NIST).
* **Manajemen IT & CISO:** Sebagai dasar penentuan kebijakan tata kelola risiko keamanan siber, alokasi sumber daya, dan penyusunan SOP institusi.

### 1.4 Ruang Lingkup (Scope & Out-of-Scope)

**Perangkat dan Lingkungan yang Dicakup (In-Scope):**
* **Perimeter Firewall:** Firewall gerbang utama yang membatasi jaringan privat organisasi dengan Internet publik.
* **Internal Segmentation Firewall (ISFW):** Firewall yang mengontrol lalu lintas antar-segmen internal (misal antara segmen pengguna, server farm, dan sistem manajemen).
* **Host-based Firewall:** Firewall lokal pada sistem operasi server (seperti Linux `nftables/iptables` dan `Windows Defender Firewall with Advanced Security`).
* **Cloud Security Groups & Virtual Appliances:** Firewall virtual dan access control list di lingkungan cloud (AWS Security Groups, Azure Network Security Groups, GCP Firewall Rules).

**Batasan yang Tidak Dicakup (Out-of-Scope):**
* Panduan teknis pengujian penetrasi ofensif (*penetration testing*), skrip eksploitasi, atau teknik *firewall evasion* (dokumen ini murni defensif).
* Konfigurasi mendalam *signature tuning* Intrusion Prevention System (IPS) aplikasi spesifik (cukup disinggung sebagai integrasi perlindungan konten).
* Aturan spesifik aplikasi web pada Web Application Firewall (WAF) seperti modul penanganan SQLi/XSS custom (disinggung sebagai referensi silang arsitektur pertahanan berlapis).

---

## BAB 2 — GLOSARIUM & TERMINOLOGI LENGKAP

Bagian ini menyajikan kamus istilah resmi yang sering digunakan dalam administrasi firewall, dikelompokkan ke dalam 5 sub-kategori. Setiap istilah disajikan dalam format **Tabel: Istilah + Definisi Teknis + Konteks Penggunaan & Contoh Kasus Riil**.

### 2.1 Terminologi Dasar Firewall

| Istilah | Definisi Teknis | Konteks Penggunaan & Contoh Kasus Riil |
| :--- | :--- | :--- |
| **Firewall** | Perangkat keamanan jaringan (perangkat keras, perangkat lunak, atau virtual) yang memantau dan mengontrol lalu lintas jaringan masuk dan keluar berdasarkan aturan keamanan yang telah ditentukan. | Ditempatkan di titik temu antara jaringan internal kantor dengan ISP Internet publik untuk memblokir akses yang tidak sah. |
| **Packet Filtering** | Metode penyaringan paket generasi awal (L3/L4) yang mengevaluasi setiap paket data secara terisolasi murni berdasarkan header IP (sumber/tujuan), protokol, dan nomor port tanpa melacak status koneksi (*stateless*). | Digunakan pada access list (ACL) sederhana di router border edge untuk menyaring paket dengan kecepatan kawat (*wire-speed*). |
| **Stateful Inspection** | Teknologi penyaringan paket cerdas yang melacak status aktif dan konteks dari setiap sesi koneksi jaringan (seperti TCP SYN, ESTABLISHED) di dalam tabel memori internal (*State Table*). | Ketika laptop internal membuka situs web perbankan, firewall mencatat sesi TCP tersebut. Paket balasan dari bank otomatis diizinkan masuk kembali tanpa perlu aturan *inbound permit* baru. |
| **Stateless Filtering** | Mekanisme filter yang memperlakukan setiap paket data sebagai entitas independen tanpa mengingat paket sebelumnya. Tidak menyimpan state sesi di memori. | Efektif digunakan untuk memitigasi serangan DDoS volume tinggi pada level router terluar karena tidak membebani memori CPU untuk mencatat tabel sesi. |
| **Proxy Firewall / Application-Level Gateway (ALG)** | Firewall yang bertindak sebagai perantara penuh pada Layer 7 (Aplikasi). Menghubungkan diri ke klien, memeriksa seluruh muatan data, lalu membuka koneksi terpisah baru ke server tujuan. | Digunakan untuk menginspeksi protokol spesifik seperti FTP atau SIP, memastikan bahwa hanya perintah yang sah yang diteruskan ke server internal. |
| **Next-Generation Firewall (NGFW)** | Firewall modern yang menggabungkan inspeksi stateful tradisional dengan kemampuan identifikasi aplikasi Layer 7 (*App-ID*), IPS terintegrasi, kontrol identitas pengguna (*User-ID*), dan dekripsi SSL/TLS mendalam. | Sebuah rule NGFW mengizinkan port 443 (HTTPS), tetapi secara spesifik hanya mengizinkan aplikasi *Zoom Video Conferencing* dan otomatis memblokir *BitTorrent* pada port yang sama. |
| **Circuit-Level Gateway** | Mekanisme keamanan yang memvalidasi jabat tangan TCP (*TCP Handshake*) antara dua host sebelum mengizinkan sesi transfer data berlangsung, tanpa memeriksa isi muatan data aplikasi. | Digunakan pada server proxy SOCKS untuk mengamankan koneksi keluar pengguna tanpa membebani pemrosesan payload konten. |
| **Host-based Firewall vs Network-based Firewall** | **Network-based:** Firewall appliance fisik/virtual yang melindungi seluruh subnet jaringan. **Host-based:** Aplikasi firewall lokal yang terinstal di dalam satu sistem operasi host (misal Linux `iptables` atau Windows Firewall) untuk melindungi dirinya sendiri. | Strategi Defense-in-Depth: Network firewall membatasi akses ke subnet DMZ, sementara Host-based firewall di server Linux hanya mengizinkan port 22 dibuka dari IP jump-host. |
| **Unified Threat Management (UTM)** | Arsitektur keamanan terintegrasi tunggal (*all-in-one appliance*) yang menjalankan fungsi Firewall, Antivirus jaringan, Intrusion Prevention System (IPS), Web Filtering, dan Anti-Spam secara simultan. | Sangat populer diimplementasikan pada kantor cabang enterprise untuk menyederhanakan manajemen keamanan perimeter tanpa memerlukan banyak perangkat keras terpisah. |

### 2.2 Terminologi Arsitektur & Zonasi

| Istilah | Definisi Teknis | Konteks Penggunaan & Contoh Kasus Riil |
| :--- | :--- | :--- |
| **Zone-Based Firewall** | Arsitektur firewall di mana antarmuka fisik atau virtual dikelompokkan ke dalam wadah logis (*Security Zones*) dengan tingkat kepercayaan seragam. Kebijakan lalu lintas diterapkan antar-zona (*inter-zone*). | Membuat zona `WAN_Zone`, `DMZ_Zone`, dan `LAN_Zone`. Kebijakan keamanan mendefinisikan aturan dari `LAN_Zone` menuju `WAN_Zone`. |
| **DMZ (Demilitarized Zone)** | Subnet jaringan perimeter terisolasi yang menampung layanan publik organisasi (Web Server, API Gateway, Mail Relay), bertindak sebagai zona penyangga antara Internet publik dan jaringan privat internal. | Server web portal publik ditempatkan di DMZ. Jika server web tersebut dieksploitasi oleh hacker, penyerang terisolasi di DMZ dan tidak dapat langsung merembes ke server database di LAN internal. |
| **Trust Zone / Untrust Zone** | **Trust Zone:** Segmen jaringan internal yang berada di bawah kendali administratif organisasi dengan tingkat kepercayaan tinggi. **Untrust Zone:** Jaringan luar yang tidak terpercaya dan berada di luar kendali organisasi (misal Internet publik). | Seluruh lalu lintas dari Untrust Zone menuju Trust Zone secara default diblokir total kecuali melewati aturan Virtual IP (VIP) yang sangat spesifik. |
| **Bastion Host / Jump Box** | Komputer server khusus yang diperkuat secara ekstrem (*hardened*) dan ditempatkan di perimeter jaringan untuk menjadi satu-satunya gerbang resmi bagi administrator dalam mengakses sistem internal secara remote. | Administrator wajib login ke Bastion Host menggunakan VPN dan MFA terlebih dahulu sebelum dapat membuka sesi SSH/RDP ke server-server produksi di datacenter. |
| **Choke Point** | Titik penyempitan jaringan strategis di mana seluruh arus lalu lintas data dipaksa melewati perangkat kendali keamanan tunggal tanpa kemungkinan adanya jalur pintas (*bypass*). | Seluruh komunikasi antara gedung kantor cabang dan datacenter pusat dipaksa melewati sepasang firewall perimeter redundan sebagai choke point. |
| **Defense in Depth** | Strategi pertahanan keamanan siber berlapis di mana beberapa lapis mekanisme kendali keamanan redundan ditempatkan di sepanjang siklus transmisi data. | Mengamankan portal web dengan border router ACL, perimeter NGFW, Web Application Firewall (WAF), segmentasi switch L3, hingga antivirus EDR pada host server. |
| **Network Segmentation & Microsegmentation** | **Segmentation:** Pembagian jaringan fisik menjadi beberapa segmen VLAN/subnet logis. **Microsegmentation:** Isolasi granular hingga ke level beban kerja tunggal (*workload/VM/container*) untuk mencegah pergerakan lateral. | Mencegah penyebaran ransomware dengan mengisolasi setiap virtual machine di datacenter sehingga server aplikasi tidak dapat saling berkomunikasi via port SMB tanpa izin. |
| **Air-Gapped Network** | Arsitektur jaringan yang terisolasi secara fisik sepenuhnya dari jaringan eksternal mana pun, termasuk Internet dan jaringan korporat biasa, tanpa koneksi kabel maupun nirkabel. | Diterapkan pada jaringan kendali industri (SCADA) pembangkit listrik atau sistem penyimpanan sertifikat akar (Root CA) perbankan. |

### 2.3 Terminologi Rule & Kebijakan Akses

| Istilah | Definisi Teknis | Konteks Penggunaan & Contoh Kasus Riil |
| :--- | :--- | :--- |
| **ACL (Access Control List)** | Daftar pernyataan aturan berurutan yang menginstruksikan firewall atau router mengenai paket data mana yang diizinkan (*permit*) atau ditolak (*deny*) berdasarkan atribut header. | Diterapkan pada switch core: `permit tcp 10.0.0.0/24 any eq 443`, `deny ip any any`. |
| **Allow-List vs Deny-List** | Pergeseran istilah modern dari *Whitelist/Blacklist*. **Allow-list (Filosofi Positif):** Menolak semua dan hanya mengizinkan entitas yang terdaftar. **Deny-list (Filosofi Reaktif):** Mengizinkan semua dan hanya memblokir entitas yang terdaftar. | Standar enterprise mewajibkan pendekatan Allow-list: hanya port 443 dan 80 yang dibuka keluar untuk pengguna; semua port lainnya diblokir. |
| **Implicit Deny / Default Deny** | Aturan pamungkas tak terlihat di baris paling bawah dari sistem firewall yang secara otomatis membuang (*drop*) semua paket yang tidak cocok dengan aturan-aturan di atasnya. | Jika paket datang menuju port TCP 8080 dan tidak ada aturan yang mengizinkan port tersebut, paket langsung di-drop oleh mekanisme Implicit Deny. |
| **Rule Base & Rule Ordering** | Susunan tabel aturan firewall. Evaluasi dilakukan dari atas ke bawah (*Top-Down matching*); begitu sebuah paket cocok dengan satu aturan, aksi langsung dieksekusi dan evaluasi dihentikan (*First-Match*). | Aturan pemblokiran botnet IP diletakkan di baris #2, sedangkan aturan browsing umum diletakkan di baris #20 agar lalu lintas botnet langsung dipotong sebelum dievaluasi rule lain. |
| **Shadow Rule (Rule Terbayang)** | Kondisi anomali konfigurasi di mana suatu aturan spesifik diletakkan di bawah aturan yang lebih umum, sehingga aturan spesifik tersebut tidak akan pernah dievaluasi atau dieksekusi (*unreachable*). | Rule #5: `Permit ANY to ANY`. Rule #10: `Deny IP_Hacker to ANY`. Rule #10 menjadi shadow rule karena penyerang dari `IP_Hacker` sudah diizinkan terlebih dahulu oleh Rule #5. |
| **NAT (Network Address Translation) & PAT** | **NAT:** Translasi alamat IP sumber atau tujuan. **PAT (Port Address Translation / Overload):** Mentranslasikan ribuan IP privat LAN ke dalam 1 IP publik WAN menggunakan port sumber dinamis yang berbeda. | Kantor dengan 2.000 karyawan mengakses Internet secara bersamaan melalui 1 alamat IP publik ISP menggunakan mekanisme PAT Overload. |
| **Port Forwarding / DNAT** | Destination NAT yang memetakan lalu lintas dari port IP publik eksternal ke alamat IP privat dan port internal spesifik di dalam jaringan. | Publik mengakses `http://203.0.113.10:8080`, firewall meneruskan paket tersebut ke server web internal pada IP `192.168.1.50:80`. |
| **Object Grouping** | Pengelompokan beberapa alamat IP (*Address Objects*) atau nomor port (*Service Objects*) ke dalam satu nama kelompok (*Group Object*) logis untuk menyederhanakan penulisan aturan. | Membuat grup `GRP_DATABASE_SERVERS` yang berisi 5 alamat IP server. Rule firewall cukup memanggil grup tersebut sekali, tidak perlu membuat 5 rule terpisah. |

### 2.4 Terminologi Ancaman & Deteksi

| Istilah | Definisi Teknis | Konteks Penggunaan & Contoh Kasus Riil |
| :--- | :--- | :--- |
| **IDS vs IPS** | **IDS (Intrusion Detection System):** Pasif, hanya memantau dan memberi peringatan alarm saat mendeteksi serangan. **IPS (Intrusion Prevention System):** Aktif berada di jalur data (*in-line*), memantau sekaligus langsung memblokir paket serangan. | Sensor IPS mendeteksi upaya eksploitasi celah Log4j pada paket HTTP dan langsung mereset koneksi TCP (*TCP Reset*) sebelum paket menyentuh server web. |
| **DPI (Deep Packet Inspection)** | Pemeriksaan mendalam yang membongkar hingga ke muatan data bersih (*payload*) pada Layer 7 aplikasi untuk menganalisis protokol, mendeteksi virus, spam, atau kebocoran data sensitif. | Firewall memeriksa file dokumen Word yang sedang diunduh pengguna lewat email dan memblokirnya karena di dalamnya ditemukan skrip makro berbahaya. |
| **Anomaly-based vs Signature-based Detection** | **Signature-based:** Mencocokkan paket dengan pola tanda tangan serangan yang sudah dikenal (cepat, namun gagal terhadap ancaman baru). **Anomaly-based:** Membandingkan trafik dengan baseline normal untuk menemukan perilaku ganjil (*heuristic/AI*). | Deteksi anomali memicu alarm ketika server database internal tiba-tiba mengirimkan 10 GB data keluar pada jam 2 dini hari ke alamat IP antah-berantah. |
| **Rate Limiting / Throttling** | Pembatasan kuota volume lalu lintas atau jumlah koneksi baru per satuan waktu (misal maksimal 50 paket per detik) yang diizinkan melewati antarmuka firewall. | Diterapkan pada protokol ICMP (Ping) dan DNS untuk mencegah firewall kewalahan saat menerima serangan banjir permintaan (*request flooding*). |
| **IP Spoofing & MAC Spoofing** | Teknik pemalsuan di mana penyerang memanipulasi header paket jaringan untuk menyamarkan alamat IP atau alamat MAC sumber agar menyerupai perangkat terpercaya. | Penyerang di Internet mengirim paket dengan IP sumber `192.168.1.1` (IP gateway internal) untuk mengelabui filter firewall. Dimigitasi menggunakan *Anti-Spoofing / uRPF*. |
| **SYN Flood, UDP Flood, ICMP Flood** | Kategori serangan Denial of Service (DoS): **SYN Flood:** Menghabiskan tabel koneksi TCP dengan mengirim SYN tanpa ACK. **UDP/ICMP Flood:** Menghabiskan bandwidth antarmuka dengan paket sampah tanpa koneksi. | Firewall enterprise mengaktifkan fitur *SYN Proxy / SYN Cookies* untuk menahan jutaan paket SYN palsu per detik saat diserang oleh botnet. |
| **Port Scanning & Reconnaissance** | Aktivitas pemindaian otomatis (menggunakan tool seperti Nmap) untuk memetakan port terbuka, layanan yang aktif, dan sistem operasi yang berjalan pada target sebelum melancarkan serangan. | Firewall mendeteksi ada IP luar yang mencoba mengetuk 100 port berbeda dalam rentang waktu 3 detik, dan secara otomatis memasukkan IP tersebut ke daftar blokir sementara (*shun/blacklist*). |
| **Man-in-the-Middle (MitM)** | Serangan di mana penyerang menyusup secara diam-diam di antara dua pihak yang sedang berkomunikasi untuk menyadap atau mengubah data yang melintas. | Dalam konteks pengamanan firewall, firewall bertindak sebagai "MitM Resmi" (*SSL Decryption*) untuk memeriksa paket terenkripsi HTTPS demi mencegah malware masuk tanpa terdeteksi. |

### 2.5 Terminologi Manajemen & Tata Kelola

| Istilah | Definisi Teknis | Konteks Penggunaan & Contoh Kasus Riil |
| :--- | :--- | :--- |
| **Change Management / Change Control** | Proses formal tata kelola IT untuk memastikan bahwa seluruh modifikasi pada konfigurasi firewall diajukan, dianalisis risikonya, disetujui, diuji, dan didokumentasikan secara resmi. | Administrator dilarang membuka port firewall langsung di CLI tanpa tiket Change Request (CR) yang telah disetujui oleh Dewan Penasihat Perubahan (*CAB*). |
| **Rule Recertification / Rule Review** | Proses audit berkala terjadwal untuk meninjau kembali seluruh aturan firewall yang aktif, memastikan apakah justifikasi bisnis aturan tersebut masih valid atau sudah dapat dihapus. | Setiap 6 bulan, pemilik aplikasi dihubungi untuk mengonfirmasi apakah port akses database lama masih digunakan; jika proyek sudah selesai, aturan tersebut dihapus. |
| **Baseline Configuration** | Kumpulan spesifikasi konfigurasi sistem yang telah disetujui, diuji, dan dinyatakan aman sebagai standar patokan minimum operasional perangkat. | Dokumen hardening ini berfungsi sebagai Baseline Configuration resmi: seluruh firewall baru yang dibeli wajib disetel mengikuti parameter dokumen ini. |
| **Configuration Drift** | Fenomena degradasi bertahap di mana konfigurasi aktual pada perangkat firewall menyimpang dari baseline resmi akibat perubahan darurat (*ad-hoc fixes*) yang tidak terdokumentasi. | Audit kuartalan menemukan ada 15 rule baru di firewall yang tidak tercatat di tiket CR, menandakan telah terjadinya configuration drift yang harus dikoreksi. |
| **SIEM (Security Information and Event Management)** | Platform perangkat lunak terpusat yang mengumpulkan, mengagregasi, dan menganalisis log keamanan dari firewall, server, dan switch secara real-time untuk mendeteksi korelasi ancaman. | Firewall mengirimkan log koneksi ke SIEM via Syslog TLS. SIEM mengkorelasikan log firewall dengan log Active Directory untuk mendeteksi upaya login mencurigakan. |
| **Log Retention Policy** | Kebijakan institusional yang menetapkan durasi minimum berapa lama file rekaman log audit jaringan wajib disimpan dan diarsipkan secara aman untuk kebutuhan investigasi dan kepatuhan. | Standar kepatuhan PCI-DSS dan ISO 27001 mewajibkan log firewall disimpan minimal selama **1 tahun (365 hari)** dengan minimal 90 hari tersedia untuk diakses seketika (*hot storage*). |
| **High Availability (HA: Active-Passive vs Active-Active)** | Desain redundansi perangkat keras. **Active-Passive:** 1 unit memproses seluruh trafik sementara unit cadangan siaga penuh (*standby*). **Active-Active:** Kedua unit memproses trafik secara bersamaan membagi beban. | Datacenter perbankan menerapkan mode Active-Passive deterministik: jika Firewall Primer tersambar petir atau mati listrik, Firewall Sekunder mengambil alih peran dalam hitungan milidetik tanpa memutus sesi pengguna. |
| **Failover & Failback** | **Failover:** Proses pengalihan operasional otomatis dari perangkat primer yang rusak ke perangkat sekunder. **Failback:** Proses pengembalian operasional ke perangkat primer setelah perangkat tersebut berhasil diperbaiki. | Kabel uplink ISP pada Firewall-01 putus; sistem secara otomatis melakukan failover ke Firewall-02. Setelah kabel diperbaiki, dilakukan failback terencana saat jendela pemeliharaan. |

---

## BAB 3 — PRINSIP DASAR HARDENING

Hardening firewall bukan sekadar mencentang checklist teknis, melainkan menerapkan prinsip arsitektur keamanan siber yang fundamental. Setiap keputusan konfigurasi harus berakar pada enam pilar prinsip berikut:

```
+-------------------------------------------------------------------------------------------------------+
|                                    6 PILAR UTAMA PRINSIP HARDENING                                    |
+-----------------------------------+-----------------------------------+-------------------------------+
| 1. LEAST PRIVILEGE                | 2. DEFAULT DENY                   | 3. DEFENSE IN DEPTH           |
| - Batasi akses seminimal mungkin  | - Tolak semua trafik secara pasif | - Firewall bukan lapis tunggal|
| - Rationale: Isolasi dampak breach| - Rationale: Cegah port gelap lolos| - Rationale: Redundansi kontrol|
+-----------------------------------+-----------------------------------+-------------------------------+
| 4. MINIMALISASI ATTACK SURFACE    | 5. FAIL SECURE VS FAIL OPEN       | 6. SEGREGATION OF DUTIES      |
| - Tutup port & servis tak terpakai| - Perilaku saat sistem crash/mati | - Pembuat rule != Penyetuju   |
| - Rationale: Eliminasi celah CVE  | - Rationale: Prioritas proteksi   | - Rationale: Cegah fraud & khilaf|
+-----------------------------------+-----------------------------------+-------------------------------+
```

### 3.1 Prinsip Least Privilege
* **Konsep:** Setiap entitas pengguna, aplikasi, sistem, atau perangkat jaringan hanya diberikan hak akses minimum yang mutlak diperlukan untuk menyelesaikan tugas pekerjaannya yang sah, dan tidak lebih dari itu.
* **Alasan & Rationale Keamanan:** Memberikan akses yang lebih luas dari kebutuhan riil memperbesar dampak kerusakan jika akun atau sistem tersebut berhasil diretas (*blast radius expansion*). Jika server web hanya membutuhkan akses ke server database pada port TCP 5432 (PostgreSQL), maka tidak boleh ada pembukaan port SSH (TCP 22) atau ICMP dari server web ke database. Jika server web terkompromi oleh injeksi kode, penyerang tidak dapat mengeksploitasi konsol terminal database.

### 3.2 Prinsip Default Deny
* **Konsep:** Seluruh lalu lintas jaringan, paket data, dan permintaan koneksi dianggap terlarang (*forbidden by default*) kecuali secara eksplisit terdapat aturan resmi yang mengizinkannya (*explicit allow*).
* **Alasan & Rationale Keamanan:** Manusia tidak dapat memprediksi seluruh jenis paket berbahaya yang ada di dunia (pendekatan Blacklist selalu kalah langkah dengan ancaman baru). Dengan mengunci seluruh pintu secara default dan hanya membuka lubang kunci yang sangat spesifik, organisasi secara otomatis terlindungi dari ribuan port serangan yang tidak terduga.

### 3.3 Prinsip Defense in Depth
* **Konsep:** Pengamanan sistem informasi harus dirancang berlapis-lapis, mengombinasikan kontrol administratif, fisik, dan teknis pada berbagai tingkatan infrastruktur jaringan.
* **Alasan & Rationale Keamanan:** Tidak ada satu produk keamanan pun yang kebal 100% dari kegagalan atau kerentanan zero-day. Jika perimeter firewall gagal memblokir serangan (misal karena lolos melalui enkripsi TLS), lapisan pengamanan berikutnya (seperti Web Application Firewall di depan server, segregasi internal VLAN, dan agen EDR pada endpoint) siap mendeteksi dan menetralkan serangan tersebut sebelum menyentuh aset data kritis.

### 3.4 Prinsip Minimalisasi Attack Surface
* **Konsep:** Mengurangi jumlah total titik masuk (*entry points*), layanan yang berjalan, protokol yang aktif, dan antarmuka yang dapat diakses oleh pihak luar seminimal mungkin.
* **Alasan & Rationale Keamanan:** Setiap baris kode perangkat lunak dan setiap port jaringan yang terbuka memiliki potensi mengandung celah kerentanan (*vulnerability*). Mematikan layanan manajemen HTTP, menutup respons Ping ICMP di antarmuka publik, menghapus antarmuka virtual yang tidak terpakai, dan menonaktifkan protokol warisan (seperti Telnet, SNMPv1/v2c) secara drastis mempersempit peluang penyerang untuk menemukan celah masuk.

### 3.5 Prinsip Fail Secure vs Fail Open
* **Konsep:** Menentukan bagaimana perilaku firewall ketika terjadi kondisi kegagalan sistem kritis (*system crash*, kehabisan memori RAM, atau kegagalan modul inspeksi):
  * **Fail Secure (Fail Closed):** Firewall memutus seluruh lalu lintas data dan menolak semua koneksi saat sistem mengalami kegagalan.
  * **Fail Open:** Firewall meloloskan seluruh lalu lintas data tanpa inspeksi agar ketersediaan jaringan (*uptime*) tetap terjaga saat sistem crash.
* **Alasan & Rationale Keamanan:** Dalam arsitektur keamanan enterprise berisiko tinggi (keuangan, pertahanan, data sensitif), perilaku **Fail Secure adalah kewajiban mutlak**. Memilih *Fail Open* berarti membuka pintu gerbang benteng selebar-lebarnya tanpa satpam saat listrik padam, memungkinkan penyerang sengaja membanjiri firewall hingga crash demi meloloskan paket serangan tanpa inspeksi.

### 3.6 Prinsip Segregation of Duties (Pemisahan Tugas)
* **Konsep:** Memastikan bahwa tidak ada satu individu tunggal yang memiliki kewenangan penuh dari hulu ke hilir untuk mengajukan, menyetujui, mengonfigurasi, dan mengaudit aturan firewall secara mandiri tanpa pengawasan pihak lain.
* **Alasan & Rationale Keamanan:** Mencegah terjadinya penyalahgunaan wewenang secara sengaja (*internal fraud/malicious insider*) serta meminimalisasi kesalahan manusiawi (*human error*). Administrator jaringan yang bertugas mengeksekusi rule di firewall tidak boleh merangkap sebagai pihak yang menyetujui tiket perubahan (*approver*), dan tim auditor harus sepenuhnya independen dari tim pengelola firewall harian.

---

## BAB 4 — CHECKLIST HARDENING TEKNIS

Bab ini menyajikan panduan operasional teknis yang siap diaplikasikan pada perangkat firewall. Disusun ke dalam 7 kategori dengan format: **Checklist Status + Penjelasan Singkat Rationale**.

---

### 4.1 Manajemen Akses Administratif

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST 4.1: MANAJEMEN AKSES ADMINISTRATIF                                                          |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Item Parameter Tindakan Pengerasan                                          | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| 1.1 | Wajibkan Autentikasi Multi-Faktor (MFA) untuk seluruh akun admin            | KRITIS  | [ ] OK  |
| 1.2 | Akses manajemen HANYA via protokol terenkripsi (HTTPS & SSH, matikan HTTP)  | KRITIS  | [ ] OK  |
| 1.3 | Pembatasan Alamat IP Sumber yang boleh mengakses antarmuka manajemen        | KRITIS  | [ ] OK  |
| 1.4 | Penonaktifan akun bawaan pabrik (default user "admin") dan ubah password    | TINGGI  | [ ] OK  |
| 1.5 | Terapkan kebijakan password kompleks dan rotasi berkala                     | TINGGI  | [ ] OK  |
| 1.6 | Terapkan Role-Based Access Control (RBAC) dengan prinsip least privilege    | TINGGI  | [ ] OK  |
| 1.7 | Konfigurasikan batas waktu idle session timeout maksimal 10 menit (600 detik)| SEDANG  | [ ] OK  |
| 1.8 | Aktifkan proteksi brute-force account lockout (kunci setelah 3-5 kali gagal)| TINGGI  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

* **Kenapa Item Ini Penting?**
  * *MFA (1.1) & IP Whitelist (1.3):* Celah keamanan terbesar peralatan perimeter adalah antarmuka manajemen web yang terekspos ke publik. Dengan mengunci akses login hanya dari IP subnet SOC/OOB internal dan mewajibkan token MFA, penyerang dari Internet tidak akan dapat menyentuh layar login firewall meskipun memiliki kredensial curian.
  * *Matikan HTTP/Telnet (1.2):* Menghilangkan risiko pencurian kata sandi admin melalui penyadapan teks terbuka di jaringan lokal.
  * *RBAC (1.6):* Menjamin staf operasional junior hanya memiliki hak baca (*read-only*) untuk pemantauan, sementara hak perubahan konfigurasi hanya dipegang oleh *Senior Security Engineer*.

---

### 4.2 Kebijakan Rule/ACL

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST 4.2: KEBIJAKAN RULE / ACL                                                                   |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Item Parameter Tindakan Pengerasan                                          | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| 2.1 | Hapus seluruh aturan permisif luas (Rule "ANY-ANY-ANY PERMIT")              | KRITIS  | [ ] OK  |
| 2.2 | Pastikan aturan pamungkas terbawah adalah Explicit Cleanup Deny-All         | KRITIS  | [ ] OK  |
| 2.3 | Setiap rule WAJIB memiliki dokumentasi tujuan bisnis dan nomor tiket resmi  | TINGGI  | [ ] OK  |
| 2.4 | Urutkan aturan berdasarkan spesifisitas (paling spesifik di baris teratas)  | TINGGI  | [ ] OK  |
| 2.5 | Lakukan audit dan eliminasi Shadow Rules secara rutin                        | TINGGI  | [ ] OK  |
| 2.6 | Bersihkan aturan dan objek yang tidak pernah terpakai (Zero Hit Count)      | SEDANG  | [ ] OK  |
| 2.7 | Terapkan batasan waktu (Time-based Rules / Expiry Date) untuk akses temporer| SEDANG  | [ ] OK  |
| 2.8 | Batasi protokol berisiko tinggi antar-segmen internal (Blokir SMB/RPC/RDP)  | TINGGI  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

* **Kenapa Item Ini Penting?**
  * *Hapus Any-Any-Permit (2.1):* Rule any-any mematikan seluruh fungsi firewall, mengubahnya menjadi router biasa yang melewatkan segala bentuk lalu lintas malware.
  * *Explicit Cleanup Deny-All (2.2):* Memastikan setiap paket liar yang tidak memiliki izin resmi dibuang secara terkontrol dan dicatat ke dalam log untuk dianalisis oleh tim SOC.
  * *Eliminasi Shadow Rules (2.5):* Mencegah anomali logika di mana aturan keamanan ketat menjadi mandul karena tertutup oleh aturan longgar di atasnya.
  * *Time-based Rules (2.7):* Akses vendor pihak ketiga yang hanya dibuka untuk pemeliharaan 2 hari tidak boleh tertinggal dan terlupakan menjadi pintu belakang permanen selama bertahun-tahun.

---

### 4.3 Manajemen NAT & Port Forwarding

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST 4.3: MANAJEMEN NAT & PORT FORWARDING                                                        |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Item Parameter Tindakan Pengerasan                                          | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| 3.1 | Minimalkan port forwarding (DNAT) langsung ke server jaringan internal      | TINGGI  | [ ] OK  |
| 3.2 | DILARANG mempublikasikan port manajemen lokal (443/22) via VIP ke publik   | KRITIS  | [ ] OK  |
| 3.3 | Batasi alamat IP sumber (Source Whitelist) pada seluruh aturan port forward | KRITIS  | [ ] OK  |
| 3.4 | Gunakan SNAT untuk menyembunyikan struktur topologi IP privat internal     | TINGGI  | [ ] OK  |
| 3.5 | Pisahkan IP Pool SNAT pengguna umum dari IP Pool server datacenter         | SEDANG  | [ ] OK  |
| 3.6 | Lakukan audit berkala terhadap seluruh aturan DNAT/VIP aktif                | TINGGI  | [ ] OK  |
| 3.7 | Pasang rute Blackhole (Null0) pada subnet publik untuk mencegah loop        | TINGGI  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

* **Kenapa Item Ini Penting?**
  * *Larangan Port Manajemen di VIP (3.2):* Menyembunyikan antarmuka login firewall dari pemindai botnet global.
  * *Source Whitelisting di DNAT (3.3):* Jika server di DMZ hanya perlu diakses oleh kantor cabang atau rekanan API B2B tertentu, mengunci alamat IP sumber mencegah seluruh pengguna gelap di Internet mencoba membobol port tersebut.
  * *Blackhole Routing (3.7):* Mengeliminasi risiko pemborosan bandwidth akibat *routing loop* saat paket menuju alamat IP publik lokal yang sedang tidak diasosiasikan dengan server aktif.

---

### 4.4 Logging & Monitoring

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST 4.4: LOGGING & MONITORING                                                                   |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Item Parameter Tindakan Pengerasan                                          | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| 4.1 | Aktifkan pencatatan log untuk seluruh aturan penolakan (Deny Rules)         | TINGGI  | [ ] OK  |
| 4.2 | Konfigurasikan log aturan izin pada mode akhir sesi (Session-End/Close)     | SEDANG  | [ ] OK  |
| 4.3 | Terapkan kebijakan retensi penyimpanan log sesuai kepatuhan (180 - 365 hari)| TINGGI  | [ ] OK  |
| 4.4 | Wajibkan pengiriman log terenkripsi secara real-time ke SIEM terpusat       | KRITIS  | [ ] OK  |
| 4.5 | Sinkronisasikan waktu perangkat menggunakan server NTP terpercaya           | TINGGI  | [ ] OK  |
| 4.6 | Aktifkan mekanisme peringatan instan (Alerting) untuk pola anomali trafik   | TINGGI  | [ ] OK  |
| 4.7 | Aktifkan pencatatan log audit terhadap seluruh perubahan konfigurasi admin  | KRITIS  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

* **Kenapa Item Ini Penting?**
  * *Integrasi SIEM (4.4) & Retensi (4.3):* Penyimpanan lokal firewall sangat terbatas dan mudah tertimpa. Mengalirkan log ke SIEM menjamin ketersediaan rekaman bukti forensik saat terjadi investigasi insiden siber beberapa bulan kemudian.
  * *Sinkronisasi NTP (4.5):* Menjamin stempel waktu pada log firewall identik dengan stempel waktu di server web dan database, memungkinkan korelasi kronologis serangan secara presisi.
  * *Session-End Logging (4.2):* Mencatat volume data riil (*bytes transferred*) dan durasi koneksi untuk analisis anomali kebocoran data (*data exfiltration*).

---

### 4.5 Manajemen Patch & Firmware

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST 4.5: MANAJEMEN PATCH & FIRMWARE                                                             |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Item Parameter Tindakan Pengerasan                                          | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| 5.1 | Tetapkan jadwal rutin pengecekan buletin keamanan vendor (minimal mingguan) | TINGGI  | [ ] OK  |
| 5.2 | Terapkan kebijakan firmware stabil (Mature/LTS Release Branch N-1)          | TINGGI  | [ ] OK  |
| 5.3 | Wajibkan pengujian patch di lingkungan lab/staging sebelum ke produksi      | TINGGI  | [ ] OK  |
| 5.4 | Siapkan dan dokumentasikan rencana pengembalian darurat (Rollback Plan)     | KRITIS  | [ ] OK  |
| 5.5 | Lakukan verifikasi integritas file image firmware menggunakan hash SHA-256  | TINGGI  | [ ] OK  |
| 5.6 | Dokumentasikan riwayat pembaruan versi firmware dan nomor CVE yang ditutup  | SEDANG  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

* **Kenapa Item Ini Penting?**
  * *Verifikasi Hash SHA-256 (5.5):* Mencegah pemasangan file firmware palsu yang telah disusupi oleh penyerang melalui serangan *Man-in-the-Middle* atau peretasan server mirror.
  * *Rollback Plan (5.4):* Menjamin tim infrastruktur memiliki prosedur evakuasi cepat jika firmware baru menyebabkan malfungsi sistem atau gangguan pada aplikasi bisnis.

---

### 4.6 Proteksi terhadap Serangan Umum

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST 4.6: PROTEKSI TERHADAP SERANGAN UMUM                                                        |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Item Parameter Tindakan Pengerasan                                          | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| 6.1 | Aktifkan proteksi TCP SYN Flood (SYN Proxy / SYN Cookie Defense)            | KRITIS  | [ ] OK  |
| 6.2 | Terapkan Rate Limiting pada lalu lintas ICMP dan permintaan koneksi baru    | SEDANG  | [ ] OK  |
| 6.3 | Aktifkan filter Anti-Spoofing (uRPF / Drop Martian IP RFC 1918 di WAN)      | KRITIS  | [ ] OK  |
| 6.4 | Terapkan Geo-IP Filtering untuk memblokir negara tanpa kepentingan bisnis   | SEDANG  | [ ] OK  |
| 6.5 | Tetapkan ambang batas peringatan dini untuk pendeteksian Port Scanning     | TINGGI  | [ ] OK  |
| 6.6 | Aktifkan opsi pembuangan paket berstatus TCP Flag cacat (Drop Invalid State)| TINGGI  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

* **Kenapa Item Ini Penting?**
  * *SYN Cookies (6.1):* Mencegah penyerang melumpuhkan memori tabel koneksi firewall hanya dengan modal skrip banjir paket SYN sederhana.
  * *Anti-Spoofing (6.3):* Memastikan penyerang di luar jaringan tidak dapat memalsukan alamat IP sumber untuk menyamar sebagai host internal terpercaya.
  * *Drop Invalid State (6.6):* Menggagalkan teknik pemindaian siluman (*stealth port scan*) seperti FIN scan dan XMAS scan yang dirancang untuk memetakan arsitektur target.

---

### 4.7 High Availability & Backup

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST 4.7: HIGH AVAILABILITY & BACKUP                                                             |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Item Parameter Tindakan Pengerasan                                          | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| 7.1 | Konfigurasikan pencadangan (backup) otomatis terjadwal harian               | KRITIS  | [ ] OK  |
| 7.2 | Simpan file backup di repositori terpisah, aman, dan terenkripsi            | KRITIS  | [ ] OK  |
| 7.3 | Lakukan pencadangan manual (Pre-Change Backup) setiap sebelum ada perubahan | KRITIS  | [ ] OK  |
| 7.4 | Terapkan kluster High Availability (Active-Passive) dengan dual link fisik  | KRITIS  | [ ] OK  |
| 7.5 | Aktifkan sinkronisasi tabel sesi koneksi (Session State Synchronization)   | TINGGI  | [ ] OK  |
| 7.6 | Konfigurasikan pemantauan antarmuka (Interface Link Monitoring / pmon)      | KRITIS  | [ ] OK  |
| 7.7 | Lakukan pengujian simulasi kegagalan (Failover Test) secara berkala         | TINGGI  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

* **Kenapa Item Ini Penting?**
  * *Backup Terpisah & Terenkripsi (7.2):* File backup konfigurasi mengandung rahasia penting (hash password, pre-shared key VPN). Jika disimpan tanpa enkripsi atau di server yang sama, peretas dapat membaca seluruh arsitektur jaringan Anda.
  * *Dual Physical Heartbeat (7.4):* Mengeliminasi risiko fatal kondisi *Split-Brain* di mana kedua unit firewall sama-sama menjadi master aktif akibat kabel komunikasi tunggal putus.
  * *Interface Monitoring (7.6):* Menjamin failover otomatis tetap terpicu meskipun firewall tidak mati listrik, asalkan kabel uplink utama ke switch/ISP terputus.

---

## BAB 5 — PROSEDUR OPERASIONAL

Bagian ini menyajikan alur kerja formal (*workflow*) yang wajib dijalankan oleh organisasi dalam mengoperasikan dan memelihara firewall.

---

### 5.1 Prosedur Perubahan Konfigurasi (Change Control Workflow)

Setiap modifikasi pada firewall wajib mengikuti alur baku enam tahap berikut:

```
+---------------------------------------------------------------------------------------------------+
|                        ALUR KERJA MANAJEMEN PERUBAHAN FIREWALL (CHANGE CONTROL)                   |
+---------------------------------------------------------------------------------------------------+
  [ 1. REQUEST ]     -> Pemohon mengisi Formulir Change Request (CR) resmi di sistem tiket.
          |
          v
  [ 2. REVIEW ]      -> Security Engineer menganalisis risiko, validasi least privilege, & uji shadow.
          |
          v
  [ 3. APPROVAL ]    -> Dewan Penasihat Perubahan (CAB) mengevaluasi dampak bisnis & menyetujui jadwal.
          |
          v
  [ 4. IMPLEMENT ]   -> Pre-Change Backup dieksekusi -> Perubahan diterapkan pada Maintenance Window.
          |
          v
  [ 5. VERIFY ]      -> Uji konektivitas & verifikasi log selama 15 menit pasca-perubahan.
          |
          +---> [ GAGAL / ANOMALI ] -> Eksekusi Prosedur Rollback seketika (< 15 menit).
          |
          v [ BERHASIL ]
  [ 6. DOCUMENT ]    -> Update matriks inventaris rule, perbarui baseline, dan tutup tiket resmi.
```

1. **Tahap 1: Permohonan (Request):** Pemohon bisnis atau tim pengembang mengajukan tiket resmi yang mencantumkan: IP Sumber, IP Tujuan, Port/Protokol, Justifikasi Bisnis, dan Durasi Akses yang dibutuhkan.
2. **Tahap 2: Tinjauan Teknis (Review):** *Security Engineer* memeriksa apakah port yang diminta terlalu luas, apakah ada potensi benturan dengan rule yang sudah ada (*shadowing*), serta menentukan profil inspeksi UTM yang wajib disertakan.
3. **Tahap 3: Otorisasi (Approval):** Tiket disetujui oleh Kepala Unit Keamanan Informasi dan Dewan CAB dengan menetapkan jendela pemeliharaan (*Maintenance Window*).
4. **Tahap 4: Eksekusi (Implementation):** Administrator mengunduh cadangan konfigurasi terbaru (*Pre-Change Backup*), lalu menerapkan perintah modifikasi melalui antarmuka resmi.
5. **Tahap 5: Verifikasi (Verification):** Tim pemohon melakukan uji fungsional aplikasi sementara administrator memantau log sesi firewall. Jika terjadi gangguan yang tidak terduga, konfigurasi langsung dikembalikan (*Rollback*) menggunakan backup tahap 4.
6. **Tahap 6: Dokumentasi (Documentation):** Perubahan dicatat ke dalam dokumen matriks rule dan tiket ditutup secara formal.

---

### 5.2 Prosedur Audit Berkala

Untuk memastikan firewall tidak mengalami penurunan standar (*configuration drift*), organisasi menetapkan jadwal audit berjenjang:

| Frekuensi Audit | Penanggung Jawab | Ruang Lingkup & Fokus Pemeriksaan | Dokumen Luaran (Deliverables) |
| :--- | :--- | :--- | :--- |
| **Mingguan (Weekly)** | Network Operator / NOC | Status kesehatan perangkat: utilisasi CPU, konsumsi RAM, integritas kluster HA, kegagalan login admin berulang, dan status pembaruan signatur AV/IPS. | Checklist Kesehatan Perangkat Mingguan |
| **Bulanan (Monthly)** | Security Engineer | Tinjauan top 10 aturan dengan volume trafik tertinggi, analisis log audit perubahan konfigurasi admin, identifikasi IP pemindai terbanyak, dan verifikasi backup harian. | Laporan Analisis Keamanan Bulanan |
| **Triwulanan (Quarterly)** | Tim SOC & Internal Auditor | Pembersihan aturan tidak terpakai (*Zero-Hit Pruning*), audit shadow rules, review kepatuhan terhadap dokumen hardening, dan pengujian simulasi failover HA. | Laporan Audit Kepatuhan Triwulanan |
| **Tahunan (Annual)** | Auditor Eksternal Independen | Audit kepatuhan penuh terhadap standar ISO/IEC 27001, uji penetrasi perimeter eksternal, dan evaluasi siklus hidup firmware perangkat (*End-of-Life check*). | Sertifikasi Audit Kepatuhan Resmi |

---

### 5.3 Prosedur Review Rule (Rule Recertification)

Aturan firewall yang dibiarkan aktif selamanya tanpa pemantauan adalah sumber utama kerentanan perimeter. Siklus resertifikasi rule dijalankan dengan ketentuan:
1. **Siklus Resertifikasi Berkala:** Setiap aturan aktif wajib ditinjau ulang secara komprehensif setiap **6 bulan sekali** (atau maksimal 30 hari untuk akses proyek temporer).
2. **Kriteria Penghapusan Aturan (Rule Decommissioning):** Sebuah aturan wajib dihapus jika memenuhi salah satu kondisi berikut:
   * Memiliki nilai **Hit Count = 0 (Nol)** selama 90 hari berturut-turut.
   * Proyek atau aplikasi bisnis terkait telah dinonaktifkan atau dihentikan operasionalnya.
   * Pemilik aset (*Application Owner*) tidak lagi bekerja di institusi dan tidak ada unit kerja yang mengklaim kepemilikan aturan tersebut.
3. **Alur Eksekusi Penghapusan:**
   * Tim Security mengirimkan notifikasi resmi kepada pemilik aturan 14 hari sebelum masa kedaluwarsa.
   * Jika tidak ada konfirmasi justifikasi bisnis ulang, aturan akan **dinonaktifkan (*disabled*)** selama 14 hari masa uji (*grace period*).
   * Jika selama masa penonaktifan tidak ada komplain operasional yang sah, aturan akan **dihapus secara permanen** dari sistem firewall.

---

### 5.4 Prosedur Incident Response Terkait Firewall

Alur penanganan cepat ketika firewall mendeteksi serangan siber aktif atau anomali kritis:

```
+---------------------------------------------------------------------------------------------------+
|                        ALUR TANGGAP INSIDEN KEAMANAN FIREWALL                                     |
+---------------------------------------------------------------------------------------------------+
  [ 1. DETEKSI ]    -> Alarm SIEM berbunyi / Lonjakan bandwidth / Peringatan IPS Exploit.
         |
         v
  [ 2. TRIAGE ]     -> Validasi insiden: identifikasi IP sumber, port target, dan tanda tangan serangan.
         |
         v
  [ 3. ISOLASI ]    -> Tindakan Cepat: Masukkan IP sumber ke Emergency Blocklist (Baris #1).
         |             Jika server DMZ terkompromi, putuskan rute inter-zone ke Datacenter.
         v
  [ 4. ESKALASI ]   -> Beritahu Tim Tanggap Insiden Siber (CSIRT / CISO) dan koordinasikan dengan ISP.
         |
         v
  [ 5. REMEDIASI ]  -> Perbarui signatur IPS darurat, sesuaikan profil rate limiting, bersihkan artefak.
         |
         v
  [ 6. POST-MORTEM] -> Susun Laporan Pasca-Insiden: Analisis akar masalah (RCA) & perbarui baseline rule.
```

1. **Langkah 1: Deteksi & Validasi:** Tim SOC memverifikasi apakah lonjakan trafik merupakan serangan riil (seperti serangan DDoS SYN Flood) atau lonjakan bisnis yang sah.
2. **Langkah 2: Pembendungan Darurat (Containment):**
   * Terapkan *Emergency Blacklist* pada baris teratas firewall untuk memotong paket dari penyerang seketika.
   * Jika insiden melibatkan peretasan server di DMZ, administrator segera menonaktifkan aturan inter-zone dari DMZ ke internal guna mencegah pergerakan lateral (*lateral movement*).
3. **Langkah 3: Eskalasi:** Laporkan insiden kepada pimpinan CSIRT dan hubungi penyedia upstream ISP jika diperlukan mitigasi serangan DDoS berbasis *BGP Blackholing* di sisi provider.
4. **Langkah 4: Pemulihan & Rollback:** Jika insiden dipicu oleh salah konfigurasi aturan baru, lakukan rollback seketika ke konfigurasi stabil sebelumnya.
5. **Langkah 5: Evaluasi Pasca-Insiden (Post-Mortem):** Seluruh kronologi kejadian, log rekaman paket, dan estimasi dampak didokumentasikan dalam laporan investigasi resmi untuk memperbarui aturan pencegahan di masa depan.

---

## BAB 6 — TEMPLATE & CONTOH IMPLEMENTATIF

Gunakan format tabel siap pakai berikut sebagai instrumen tata kelola dan dokumentasi audit harian organisasi:

---

### 6.1 Template Dokumentasi Rule Firewall

Seluruh aturan yang terpasang di firewall wajib tercatat ke dalam format tabel matriks berikut:

| No | Source | Destination | Port/Protokol | Justifikasi Bisnis | Owner | Tanggal Dibuat | Tanggal Review |
|:--:|:-------|:------------|:--------------|:-------------------|:------|:--------------:|:--------------:|
| **1** | `GRP_WAN_IP_BKN` | `H_DMZ_WebSSO (10.10.230.15)` | `TCP/443 (HTTPS)` | Integrasi API Single Sign-On Kepegawaian Nasional | Biro SDM | 10-Jan-2026 | 10-Jul-2026 |
| **2** | `NET_Campus_LAN (10.0.0.0/8)` | `ANY (Internet)` | `TCP/80, 443, UDP/53` | Akses browsing kedinasan pegawai dengan inspeksi AV & WebFilter | Divisi TI | 15-Jan-2026 | 15-Jan-2027 |
| **3** | `H_DMZ_WebSSO (10.10.230.15)` | `H_DB_Prod (10.10.240.20)` | `TCP/5432 (PostgreSQL)`| Komunikasi backend portal ke cluster database internal | Tim Pengembang | 20-Jan-2026 | 20-Jul-2026 |
| **4** | `H_JumpHost_SOC (10.10.2.50)` | `GRP_Firewall_Mgmt` | `TCP/22 (SSH), 443` | Akses administrasi terpusat tim SOC via jaringan OOB | Tim Keamanan | 01-Jan-2026 | 01-Jan-2027 |
| **99**| `ANY` | `ANY` | `ALL` | Explicit Cleanup Rule (Drop all unauthorized traffic & Log) | Tim Keamanan | 01-Jan-2026 | PERMANEN |

---

### 6.2 Template Checklist Audit Kepatuhan

Gunakan format tabel berikut sebagai lembar verifikasi saat melaksanakan audit kepatuhan internal maupun persiapan sertifikasi eksternal:

| No | Item Pemeriksaan Hardening | Status (Ya/Tidak) | Catatan Temuan & Analisis Bukti | PIC Pelaksana |
|:--:|:----------------------------|:-----------------:|:--------------------------------|:-------------:|
| **1** | Autentikasi Multi-Faktor (MFA) telah aktif untuk seluruh akun login admin | [ ] YA / [ ] TIDAK | Terintegrasi dengan token TOTP dan SAML SSO | Admin SOC |
| **2** | Akses manajemen HTTP dan Telnet telah dinonaktifkan di seluruh antarmuka | [ ] YA / [ ] TIDAK | Hanya protokol HTTPS dan SSHv2 yang diizinkan | NetOps |
| **3** | Alamat IP admin dibatasi ketat menggunakan mekanisme Trusted Hosts | [ ] YA / [ ] TIDAK | Hanya subnet OOB 10.10.2.0/24 yang memiliki hak akses | SecOps |
| **4** | Tidak ada aturan yang mengizinkan lalu lintas "Any-Any-Permit" | [ ] YA / [ ] TIDAK | Seluruh aturan mendefinisikan port dan host spesifik | SecOps |
| **5** | Aturan terakhir adalah Explicit Cleanup Deny-All dengan logging aktif | [ ] YA / [ ] TIDAK | Rule ID 99 terkonfigurasi pada baris paling bawah | NetOps |
| **6** | Seluruh aturan aktif memiliki catatan justifikasi bisnis dan pemilik aset | [ ] YA / [ ] TIDAK | Terdokumentasi lengkap di matriks inventaris rule | Compliance |
| **7** | Tidak ada port manajemen (443/22) yang dipublikasikan via VIP ke publik | [ ] YA / [ ] TIDAK | Verifikasi tabel VIP menunjukkan tidak ada pemetaan GUI | SecOps |
| **8** | Log firewall terkirim secara real-time ke SIEM via Syslog TLS terenkripsi | [ ] YA / [ ] TIDAK | SIEM menerima stream log secara stabil | Analis SOC |
| **9** | Waktu perangkat disinkronkan secara presisi menggunakan server NTP Stratum-1/2 | [ ] YA / [ ] TIDAK | Menggunakan pool ntp.kemlu.go.id dan id.pool.ntp.org | NetOps |
| **10**| Versi firmware berada pada cabang stabil (Mature Branch) dan bebas CVE kritis| [ ] YA / [ ] TIDAK | Menjalankan rilis patch ke-6, verifikasi SHA-256 cocok | NetOps |
| **11**| Proteksi TCP SYN Flood (SYN Proxy/Cookie) telah diaktifkan | [ ] YA / [ ] TIDAK | Profil DoS defense aktif pada antarmuka WAN | SecOps |
| **12**| Kluster High Availability (Active-Passive) terhubung dengan dual kabel fisik | [ ] YA / [ ] TIDAK | Port HA1 dan HA2 terhubung langsung antar sasis | NetOps |
| **13**| File backup konfigurasi harian disimpan terenkripsi di server terpisah | [ ] YA / [ ] TIDAK | Otomasi backup tersimpan di repositori Git private | Admin TI |
| **14**| Siklus resertifikasi rule 6 bulanan berjalan tertib dan terdokumentasi | [ ] YA / [ ] TIDAK | Terakhir kali resertifikasi dijalankan tanggal 15-Jan-2026 | Compliance |

---

## BAB 7 — REFERENSI & STANDAR ACUAN

Dokumentasi hardening ini diselaraskan secara ketat dengan berbagai kerangka kerja standar keamanan siber internasional:

### 7.1 CIS Benchmark
* **Center for Internet Security (CIS) Firewall Benchmark v2.0:**
  * Memberikan pedoman pengerasan teknis spesifik vendor (Cisco ASA/Firepower, Fortinet FortiOS, Palo Alto Networks PAN-OS, Check Point GAiA).
  * Menetapkan rekomendasi Level 1 (kebutuhan dasar) dan Level 2 (kebutuhan keamanan tinggi / pertahanan mendalam), termasuk pembatasan akses administrative plane, penonaktifan protokol usang, dan penataan tabel ACL.

### 7.2 NIST SP 800-41 Rev. 1
* **National Institute of Standards and Technology Special Publication 800-41 Rev. 1:**
  * *Guidelines on Firewalls and Firewall Policy:* Dokumen acuan utama pemerintah dan industri global mengenai prinsip arsitektur firewall, kebijakan default deny, pemisahan zona DMZ, pertahanan spoofing, serta pemeliharaan kebijakan keamanan jaringan yang adaptif.

### 7.3 ISO/IEC 27001:2022 Annex A
* **Sistem Manajemen Keamanan Informasi (SMKI):**
  * **Kontrol A.8.20 (Network Security):** Mewajibkan jaringan dikelola dan dikendalikan secara tepat untuk melindungi informasi di dalam sistem dan aplikasi.
  * **Kontrol A.8.22 (Segregation in Networks):** Mengharuskan kelompok layanan informasi, pengguna, dan sistem informasi dipisahkan ke dalam zona jaringan yang terpisah sesuai dengan tingkat risikonya.
  * **Kontrol A.8.24 (Use of Cryptography):** Menetapkan keharusan penggunaan algoritma enkripsi modern dan pengelolaan kunci yang aman pada komunikasi data jarak jauh (seperti IPsec VPN dan administrasi SSH).

### 7.4 PCI-DSS v4.0 Requirement 1
* **Payment Card Industry Data Security Standard (PCI-DSS):**
  * **Requirement 1 (Install and Maintain Network Security Controls):** Mengatur kewajiban formal bagi organisasi untuk membangun dan memelihara firewall:
    * *Req 1.1.2:* Memelihara dokumen matriks aturan firewall yang mencantumkan justifikasi bisnis formal pada setiap port dan protokol yang diizinkan.
    * *Req 1.2.1:* Membatasi seluruh koneksi masuk dan keluar hanya pada apa yang mutlak diperlukan, dan secara eksplisit menolak seluruh lalu lintas lainnya.
    * *Req 1.2.7:* Melakukan tinjauan aturan firewall (*rule review*) minimal setiap enam bulan sekali.
    * *Req 1.3.1:* Memisahkan lingkungan data pemegang kartu (*Cardholder Data Environment - CDE*) dari jaringan korporat biasa dan jaringan nirkabel publik.

---

### Catatan Penggunaan Dokumen
Dokumen ini disusun sebagai **master baseline komprehensif** yang bersifat langsung dapat digunakan (*actionable*), siap diekspor ke format PDF/Word, dan bebas dari data konfidensial sehingga aman dijadikan portofolio repositori resmi di GitHub institusi Anda.
