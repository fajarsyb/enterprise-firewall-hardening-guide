# DOKUMENTASI STANDAR HARDENING FIREWALL ENTERPRISE
## Panduan Komprehensif Arsitektur, Pengerasan Keamanan, Prosedur Operasional, dan Kepatuhan Audit
*Standar Acuan: CIS Firewall Benchmark v2.0 | NIST SP 800-41 Rev. 1 | ISO/IEC 27001:2022 (A.8.20, A.8.22) | PCI-DSS v4.0 (Req 1)*

---

## DAFTAR ISI

1. [PENDAHULUAN](#1-pendahuluan)
   * 1.1 [Latar Belakang & Urgensi Hardening](#11-latar-belakang--urgensi-hardening)
   * 1.2 [Tujuan Dokumen](#12-tujuan-dokumen)
   * 1.3 [Audiens Sasaran](#13-audiens-sasaran)
   * 1.4 [Ruang Lingkup & Batasan (Scope & Out-of-Scope)](#14-ruang-lingkup--batasan-scope--out-of-scope)
   * 1.5 [Parameter Profil Lingkungan Acuan](#15-parameter-profil-lingkungan-acuan)
2. [GLOSARIUM & TERMINOLOGI LENGKAP](#2-glosarium--terminologi-lengkap)
   * 2.1 [Istilah Dasar Firewall](#21-istilah-dasar-firewall)
   * 2.2 [Istilah Arsitektur Jaringan](#22-istilah-arsitektur-jaringan)
   * 2.3 [Istilah Aturan & Kebijakan (Rule/ACL)](#23-istilah-aturan--kebijakan-ruleacl)
   * 2.4 [Istilah Ancaman, Deteksi & Mitigasi](#24-istilah-ancaman-deteksi--mitigasi)
   * 2.5 [Istilah Manajemen & Tata Kelola](#25-istilah-manajemen--tata-kelola)
3. [PRINSIP DASAR HARDENING FIREWALL](#3-prinsip-dasar-hardening-firewall)
   * 3.1 [Prinsip Least Privilege & Default Deny](#31-prinsip-least-privilege--default-deny)
   * 3.2 [Prinsip Defense in Depth (Pertahanan Berlapis)](#32-prinsip-defense-in-depth-pertahanan-berlapis)
   * 3.3 [Prinsip Minimalisasi Permukaan Serangan (Attack Surface Reduction)](#33-prinsip-minimalisasi-permukaan-serangan-attack-surface-reduction)
   * 3.4 [Segmentasi Jaringan & Zonasi Berbasis Tingkat Kepercayaan (Trust Levels)](#34-segmentasi-jaringan--zonasi-berbasis-tingkat-kepercayaan-trust-levels)
4. [CHECKLIST & PANDUAN HARDENING TEKNIS](#4-checklist--panduan-hardening-teknis)
   * 4.1 [Kategori A: Manajemen Akses Administratif (Management Plane)](#41-kategori-a-manajemen-akses-administratif-management-plane)
   * 4.2 [Kategori B: Kebijakan Aturan Firewall (Rule / ACL Hardening)](#42-kategori-b-kebijakan-aturan-firewall-rule--acl-hardening)
   * 4.3 [Kategori C: Manajemen NAT & Port Forwarding yang Aman](#43-kategori-c-manajemen-nat--port-forwarding-yang-aman)
   * 4.4 [Kategori D: Logging, Monitoring & Telemetri](#44-kategori-d-logging-monitoring--telemetri)
   * 4.5 [Kategori E: Manajemen Patch, Firmware & Kerentanan](#45-kategori-e-manajemen-patch-firmware--kerentanan)
   * 4.6 [Kategori F: Pengamanan Terhadap Serangan Umum](#46-kategori-f-pengamanan-terhadap-serangan-umum)
   * 4.7 [Kategori G: High Availability (HA) & Ketahanan Operasional](#47-kategori-g-high-availability-ha--ketahanan-operasional)
5. [PROSEDUR OPERASIONAL STANDAR (SOP)](#5-prosedur-operasional-standar-sop)
   * 5.1 [SOP Manajemen Perubahan Konfigurasi (Change Control)](#51-sop-manajemen-perubahan-konfigurasi-change-control)
   * 5.2 [SOP Audit & Evaluasi Berkala](#52-sop-audit--evaluasi-berkala)
   * 5.3 [SOP Resertifikasi & Review Aturan (Rule Recertification)](#53-sop-resertifikasi--review-aturan-rule-recertification)
   * 5.4 [SOP Tanggap Insiden Keamanan Terkait Firewall](#54-sop-tanggap-insiden-keamanan-terkait-firewall)
6. [TEMPLATE & DOKUMEN KERJA PRAKTIS](#6-template--dokumen-kerja-praktis)
   * 6.1 [Template Inventaris Aturan Firewall (Rule Inventory Matrix)](#61-template-inventaris-aturan-firewall-rule-inventory-matrix)
   * 6.2 [Template Formulir Permohonan Pembukaan Akses (Firewall Change Request)](#62-template-formulir-permohonan-pembukaan-akses-firewall-change-request)
   * 6.3 [Template Lembar Hasil Audit Kepatuhan (Compliance Audit Sheet)](#63-template-lembar-hasil-audit-kepatuhan-compliance-audit-sheet)
7. [REFERENSI & PENYELARASAN STANDAR](#7-referensi--penyelarasan-standar)

---

## 1. PENDAHULUAN

### 1.1 Latar Belakang & Urgensi Hardening
Firewall merupakan garis pertahanan terdepan (*first line of defense*) dan pengendali gerbang (*gatekeeper*) utama dalam infrastruktur jaringan komputer. Dalam lanskap ancaman modern, mayoritas insiden peretasan dan kebocoran data (*data breach*) yang melibatkan infrastruktur enterprise bukan disebabkan oleh kegagalan teknologi enkripsi perangkat keras, melainkan oleh **salah konfigurasi (*misconfiguration*)**, lemahnya tata kelola akses administratif, keberadaan aturan usang yang terlalu longgar (*overly permissive rules*), serta ketiadaan audit berkala.

Tanpa proses pengerasan (*hardening*) yang ketat dan terstandarisasi:
* Antarmuka manajemen firewall dapat terekspos ke publik dan dieksploitasi melalui serangan *brute force* atau kerentanan *zero-day*.
* Lalu lintas berbahaya (*malicious command-and-control*) dapat bebas keluar masuk karena aturan filter port berbasis "ANY" yang tidak terkendali.
* Pergerakan lateral (*lateral movement*) penyerang dari workstation pegawai yang terinfeksi ransomware ke server basis data produksi tidak dapat dicegah akibat ketiadaan segmentasi internal (*East-West inspection*).

### 1.2 Tujuan Dokumen
Dokumen ini disusun untuk:
1. **Menetapkan Standar Baku (Golden Baseline):** Menyediakan acuan konfigurasi defensif minimum yang wajib diterapkan pada seluruh perangkat firewall institusi.
2. **Pedoman Kepatuhan & Audit (Audit & Compliance Readiness):** Membantu organisasi membuktikan pemenuhan regulasi keamanan informasi berstandar internasional (ISO/IEC 27001, NIST SP 800-41, PCI-DSS, dan CIS Benchmark).
3. **Materi Onboarding & Operasional Tim Teknis:** Menjadi manual operasional resmi bagi Network Administrator, Security Operations Center (SOC), dan Network Operations Center (NOC) dalam mengelola siklus hidup firewall secara aman.

### 1.3 Audiens Sasaran
Dokumen ini dirancang untuk dapat dipahami dan digunakan oleh multi-disiplin peran kerja:
* **Network Security Administrator / Firewall Engineer:** Sebagai panduan teknis langsung (*hands-on*) dalam menerapkan konfigurasi, optimasi rule, dan pemeliharaan firmware.
* **Security Operations Center (SOC) / Incident Responder:** Sebagai panduan korelasi log, deteksi anomali paket, dan prosedur isolasi darurat saat terjadi serangan siber.
* **Auditor Internal / Eksternal (IT Compliance & Governance):** Sebagai lembar verifikasi bukti (*audit evidence checklist*) untuk mengukur tingkat kepatuhan organisasi.
* **Chief Information Security Officer (CISO) & IT Management:** Sebagai kerangka kerja tata kelola risiko dan penentuan kebijakan strategis keamanan jaringan.

### 1.4 Ruang Lingkup & Batasan (Scope & Out-of-Scope)
* **Dalam Lingkup (In-Scope):**
  * Pengerasan bidang manajemen (*Management Plane Hardening*) dan kontrol akses administratif.
  * Perancangan zonasi keamanan (*Security Zones*) dan segmentasi jaringan internal (*East-West & North-South*).
  * Standarisasi penyusunan aturan filter paket, inspeksi aplikasi (Layer 7), dan mitigasi *rule shadowing*.
  * Konfigurasi Network Address Translation (NAT) dan publikasi Virtual IP (VIP).
  * Strategi pencatatan log audit, integrasi SIEM, dan sinkronisasi waktu NTP.
  * Prosedur tata kelola operasional: *Change Management*, *Rule Recertification*, dan *Incident Handling*.
* **Di Luar Lingkup (Out-of-Scope):**
  * Panduan pengujian penetrasi (*penetration testing*), eksploitasi kerentanan, atau pembuatan skrip serangan (dokumen ini murni defensif).
  * Konfigurasi endpoint internal workstation (antivirus klien, sistem operasi workstation pegawai) di luar interaksinya dengan perimeter firewall.

### 1.5 Parameter Profil Lingkungan Acuan
Dokumen ini dirancang fleksibel (*vendor-agnostic*) dengan parameter profil operasional:
* **Platform Firewall Acuan:** Berlaku untuk arsitektur hardware enterprise appliance (**Fortinet FortiGate, Palo Alto Networks PA-Series, Cisco Secure Firewall / ASA, Check Point Quantum, Juniper SRX**) maupun sistem operasi berbasis perangkat lunak (**pfSense, OPNsense, Linux nftables/iptables**).
* **Topologi Jaringan:** Arsitektur terdistribusi yang mencakup *Perimeter Edge*, *Demilitarized Zone (DMZ)*, *Core Datacenter / Private Server Farm*, *Internal Campus LAN / Branch Offices*, serta *Public Cloud VPC/VNet (AWS/GCP/Azure)*.
* **Skala Organisasi:** Dirancang untuk skala Enterprise / Data Center Multi-Tenant / Lembaga Pemerintahan yang beroperasi 24/7/365.
* **Standar Kepatuhan Acuan:** Diselaraskan secara ketat dengan **ISO/IEC 27001:2022 (Kontrol A.8.20 & A.8.22)**, **NIST SP 800-41 Rev. 1**, **CIS Firewall Benchmark v2.0**, dan **PCI-DSS v4.0 (Requirement 1)**.

---

## 2. GLOSARIUM & TERMINOLOGI LENGKAP

Bagian ini menyajikan kamus istilah resmi yang sering digunakan dalam administrasi firewall, dikelompokkan ke dalam tabel referensi cepat beserta penjelasan naratif mendalam:

### 2.1 Istilah Dasar Firewall

| Istilah | Kategori Lapisan OSI | Penjelasan Teknis & Rationale |
| :--- | :---: | :--- |
| **Packet Filtering** | Layer 3 & 4 | Metode penyaringan paket generasi awal yang mengevaluasi setiap paket data secara terisolasi murni berdasarkan IP Asal, IP Tujuan, Protokol, dan Nomor Port tanpa mengingat status koneksi sebelumnya (*stateless*). Sangat cepat tetapi rentan terhadap teknik *packet crafting* dan *spoofing*. |
| **Stateful Packet Inspection (SPI)** | Layer 3 s.d. 5 | Penyaringan cerdas yang melacak siklus hidup koneksi aktif di dalam tabel sesi (*State Table*). Firewall mengenali fase *handshake* TCP (SYN, SYN-ACK, ACK), data transfer, hingga penutupan koneksi (FIN, RST). Paket balasan yang sah diizinkan masuk otomatis tanpa perlu rule izin terpisah. |
| **Proxy Firewall (Application Gateway)** | Layer 7 | Firewall yang memutus koneksi langsung antara klien dan server. Firewall bertindak sebagai perantara penuh: menerima sesi dari klien, membedah isi data aplikasi, lalu membuka sesi baru yang sepenuhnya terpisah ke server tujuan. Menjamin keamanan tertinggi namun memerlukan beban komputasi besar. |
| **Next-Generation Firewall (NGFW)** | Layer 2 s.d. 7 | Firewall modern yang menggabungkan SPI tradisional dengan inspeksi identitas aplikasi (*App-ID*), sistem pencegahan intrusi terintegrasi (*IPS*), kontrol berbasis identitas pengguna (*User-ID/SSO*), dan kemampuan dekripsi inspeksi lalu lintas SSL/TLS secara mendalam. |
| **Web Application Firewall (WAF)** | Layer 7 (HTTP/S) | Firewall khusus aplikasi web yang menganalisis lalu lintas HTTP/HTTPS dan memvalidasi permintaan web terhadap ancaman OWASP Top 10 (SQL Injection, Cross-Site Scripting/XSS, File Inclusion, Remote Code Execution). Berbeda dengan NGFW umum, WAF memahami sintaksis web dan cookies. |

### 2.2 Istilah Arsitektur Jaringan

| Istilah | Konsep Utama | Penjelasan Teknis & Rationale |
| :--- | :--- | :--- |
| **Zone-Based Architecture** | Segmentasi Keamanan | Pengelompokan satu atau beberapa antarmuka fisik/VLAN ke dalam wadah logis (*Security Zone*) yang memiliki tingkat kepercayaan seragam. Semua lalu lintas antar-zona (*inter-zone*) diblokir secara default kecuali secara eksplisit diizinkan oleh aturan kebijakan. |
| **Demilitarized Zone (DMZ)** | Perimeter Buffer | Subnet terisolasi yang menampung server yang dapat diakses dari publik (Web Server, Mail Gateway). Prinsip wajib DMZ: Server DMZ **TIDAK PERNAH** diizinkan memulai koneksi ke jaringan internal privat. Jika server DMZ diretas, peretas terisolasi dan tidak bisa merembes ke database internal. |
| **Bastion Host / Jump Box** | Gerbang Terisolasi | Komputer server tangguh yang sengaja ditempatkan di perimeter jaringan untuk menjadi satu-satunya titik masuk bagi administrator yang ingin mengelola perangkat internal dari jarak jauh. Dilengkapi dengan audit perekaman sesi, IP whitelist, dan autentikasi multi-faktor (MFA). |
| **Choke Point** | Pengendalian Arus | Titik penyempitan fisik atau logis di mana seluruh lalu lintas data antar-jaringan dipaksa melewati perangkat pemantau keamanan tunggal tanpa jalur pintas (*bypass*). Memastikan tidak ada jalan tikus yang luput dari pengawasan. |
| **Defense in Depth** | Pertahanan Berlapis | Strategi penempatan beberapa lapis kendali keamanan redundan di sepanjang jalur data (Border Router -> Perimeter Firewall -> WAF -> Internal Segmentation Firewall -> Host EDR -> Database Encryption). Kegagalan pada satu lapisan tidak mengakibatkan sistem runtuh seketika. |
| **Micro-segmentation** | Isolasi Granular | Praktik pembagian segmen jaringan internal menjadi kompartemen-kompartemen kecil hingga ke level beban kerja tunggal (*workload/VM/container*), mencegah lalu lintas lateral (*East-West*) yang tidak sah antar server di dalam subnet yang sama. |

### 2.3 Istilah Aturan & Kebijakan (Rule/ACL)

| Istilah | Penerapan Praktis | Penjelasan Teknis & Rationale |
| :--- | :--- | :--- |
| **Access Control List (ACL)** | Mekanisme Penyaringan | Kumpulan pernyataan aturan berurutan yang mendefinisikan izin (*permit/accept*) atau penolakan (*deny/drop/reject*) terhadap paket data berdasarkan atribut header jaringan yang ditentukan. |
| **Allow-List (Whitelist)** | Filosofi Keamanan Positif | Kebijakan yang hanya mencantumkan entitas, alamat IP, domain, atau aplikasi yang secara eksplisit disetujui dan dipercaya. Semua entitas lain yang tidak tercantum dianggap berbahaya dan langsung ditolak. |
| **Deny-List (Blacklist)** | Filosofi Keamanan Reaktif | Kebijakan yang mengizinkan semua lalu lintas mengalir secara bebas kecuali entitas yang secara spesifik tercantum di daftar larangan. Sangat berisiko di perimeter karena tidak mampu mendeteksi ancaman baru (*zero-day*). |
| **Implicit Deny** | Aturan Pamungkas | Aturan tak terlihat (*default fallback*) yang berada di baris paling bawah dari setiap tabel firewall, yang secara otomatis membuang semua paket yang tidak cocok dengan aturan-aturan di atasnya. |
| **Rule Ordering** | Hierarki Eksekusi | Urutan penataan baris aturan dari atas ke bawah. Karena firewall mengevaluasi paket dengan prinsip pencocokan pertama (*First-Match*), penataan urutan menentukan efektivitas dan logika keamanan sistem secara keseluruhan. |
| **NAT (Network Address Translation)** | Translasi Alamat L3 | Proses pengubahan alamat IP pada header paket saat melintasi firewall. Dibagi menjadi **SNAT** (mengubah IP sumber, digunakan agar LAN dapat mengakses Internet) dan **DNAT** (mengubah IP tujuan, digunakan untuk mempublikasikan server lokal ke publik). |
| **PAT (Port Address Translation / Overload)** | Pemetaan L4 Port | Varian SNAT yang memetakan ribuan alamat IP privat internal ke dalam satu alamat IP publik tunggal dengan membedakannya melalui nomor port sumber (*Source Port*) TCP/UDP yang unik dan dinamis. |

### 2.4 Istilah Ancaman, Deteksi & Mitigasi

| Istilah | Vektor Ancaman | Penjelasan Teknis & Rationale |
| :--- | :--- | :--- |
| **Intrusion Detection/Prevention (IDS/IPS)** | Analisis Eksploitasi | Mesin keamanan jaringan yang memantau paket secara real-time untuk mendeteksi (*IDS*) dan secara aktif memblokir (*IPS*) aktivitas mencurigakan, serangan eksploitasi kerentanan sistem operasi, *buffer overflow*, dan lalu lintas malware berbasis tanda tangan (*signatures*) atau anomali protokol. |
| **Deep Packet Inspection (DPI)** | Pembongkaran Payload | Metode pemeriksaan lanjutan yang membedah hingga ke bagian muatan data bersih (*data payload*) paket pada Layer 7, melampaui header IP dan port sederhana, untuk mengidentifikasi jenis konten, jenis file, atau perintah aplikasi tersembunyi. |
| **Rate Limiting & Traffic Shaping** | Kontrol Volume Paket | Kebijakan pembatasan jumlah maksimum paket atau bandwidth yang diizinkan mengalir dalam satuan waktu tertentu (misal maksimal 100 koneksi TCP SYN per detik per alamat IP). Digunakan untuk meredam serangan DoS flood dan pemindaian port (*port scan*). |
| **IP Spoofing & Anti-Spoofing** | Pemalsuan Identitas L3 | Teknik serangan di mana penyerang memanipulasi header IP untuk menyamarkan alamat IP sumber seolah-olah berasal dari alamat IP internal yang sah atau terpercaya. Dimigitasi menggunakan filter *Reverse Path Forwarding (uRPF)* dan pemblokiran paket *Martian / RFC 1918* pada antarmuka WAN. |
| **TCP SYN Flood** | Serangan Kehabisan Sumber Daya | Serangan DoS di mana penyerang membanjiri firewall/server dengan permintaan koneksi TCP SYN tanpa pernah menyelesaikan jabat tangan tiga arah (*three-way handshake*) dengan paket ACK. Akibatnya, tabel memori koneksi (*backlog queue*) penuh dan sistem menolak koneksi pengguna sah. |

### 2.5 Istilah Manajemen & Tata Kelola

| Istilah | Aspek Tata Kelola | Penjelasan Teknis & Rationale |
| :--- | :--- | :--- |
| **Audit Logging** | Akuntabilitas & Bukti | Rekaman kronologis terperinci dari setiap aktivitas administratif (login, perubahan konfigurasi, eksekusi perintah CLI) dan kejadian jaringan (sesi diizinkan, sesi ditolak, deteksi virus) yang disimpan untuk kebutuhan pembuktian forensik hukum. |
| **SIEM Integration** | Analitik Terpusat | Pengiriman log firewall secara real-time ke sistem *Security Information and Event Management* terpusat melalui protokol Syslog terenkripsi (TLS) untuk dikorelasikan dengan log dari server web, basis data, dan endpoint antivirus. |
| **Change Management / Change Control** | Pengendalian Risiko | Prosedur formal yang mengatur setiap rencana modifikasi pada konfigurasi firewall: mewajibkan adanya justifikasi bisnis, pengujian di laboratorium, persetujuan dewan penasihat perubahan (*Change Advisory Board - CAB*), serta rencana pengembalian darurat (*rollback plan*). |
| **Configuration Baseline** | Standar Minimum | Kumpulan parameter konfigurasi yang telah disepakati, diuji, dan dinyatakan aman sebagai patokan resmi operasional. Setiap deviasi dari baseline wajib terdeteksi secara otomatis melalui audit berkala. |

---

## 3. PRINSIP DASAR HARDENING FIREWALL

Sebelum mengeksekusi parameter teknis spesifik, arsitek jaringan dan administrator keamanan wajib menginternalisasi empat pilar filosofis pengerasan sistem keamanan perimeter berikut:

```
                  +-------------------------------------------------------------+
                  |               4 PILAR UTAMA HARDENING FIREWALL              |
                  +-------------------------------------------------------------+
                   |                             |                             |
                   v                             v                             v
  +---------------------------------+  +---------------------------------+  +---------------------------------+
  | 1. LEAST PRIVILEGE & DEF DENY   |  | 2. DEFENSE IN DEPTH             |  | 3. ATTACK SURFACE REDUCTION     |
  | - Explicit drop sebagai default |  | - Multi-layered inspection      |  | - Tutup seluruh port tak terpakai|
  | - Izinkan hanya IP/port esensial|  | - Antivirus + IPS + WAF + EDR   |  | - Matikan telnet, http, ping WAN|
  +---------------------------------+  +---------------------------------+  +---------------------------------+
                                                 |
                                                 v
                               +-----------------------------------+
                               | 4. ZERO-TRUST NETWORK SEGMENTATION|
                               | - Pisahkan Trust, Untrust, DMZ    |
                               | - Filter ketat arus East-West     |
                               +-----------------------------------+
```

### 3.1 Prinsip Least Privilege & Default Deny
* **Konsep:** Setiap entitas pengguna, aplikasi, dan perangkat sistem hanya diberikan hak akses seminimal mungkin yang mutlak diperlukan untuk menjalankan fungsi pekerjaannya, dan tidak lebih dari itu.
* **Implementasi:** Seluruh tabel kebijakan firewall harus mengadopsi postur **Default Deny (Deny-All by Default)**:
  1. Semua lalu lintas yang masuk ke antarmuka firewall dianggap ditolak secara otomatis.
  2. Akses hanya dibuka secara presisi dengan menentukan alamat IP sumber spesifik, alamat IP tujuan spesifik, dan nomor port aplikasi spesifik.
  3. **Haramkan penggunaan aturan "ANY-ANY-ANY ACCEPT":** Aturan yang mengizinkan *Source ANY*, *Destination ANY*, dan *Service ANY* pada aksi *Accept* adalah bentuk pelanggaran fatal tata kelola keamanan perimeter yang membuka seluruh sistem terhadap potensi kompromi total.

### 3.2 Prinsip Defense in Depth (Pertahanan Berlapis)
* **Konsep:** Keamanan tidak boleh bergantung pada satu pintu gerbang tunggal. Jika pertahanan terluar ditembus oleh penyerang, lapisan keamanan kedua dan ketiga harus mampu menghentikan atau mengisolasi ancaman tersebut.
* **Implementasi:**
  1. *Lapisan Perimeter:* NGFW menyaring lalu lintas IP, memblokir botnet, dan menerapkan inspeksi reputasi IP global.
  2. *Lapisan Aplikasi (DMZ):* WAF menyaring eksploitasi injeksi kode pada aplikasi web.
  3. *Lapisan Internal Segregasi:* Firewall internal membatasi komunikasi antar server di datacenter, memastikan server web tidak dapat berbicara langsung ke database di luar port SQL yang ditentukan.
  4. *Lapisan Endpoint:* Host server dilengkapi antivirus EDR lokal dan firewall berbasis host (seperti iptables/nftables atau Windows Defender Firewall).

### 3.3 Prinsip Minimalisasi Permukaan Serangan (Attack Surface Reduction)
* **Konsep:** Semakin sedikit layanan jaringan, port, dan antarmuka yang terekspos ke dunia luar, semakin kecil kemungkinan penyerang menemukan celah kerentanan yang dapat dieksploitasi.
* **Implementasi:**
  1. **Tutup Semua Port Terbuka yang Tidak Digunakan:** Nonaktifkan protokol manajemen warisan seperti Telnet (TCP 23) dan HTTP (TCP 80).
  2. **Hilangkan Respons ICMP di Interface Publik:** Nonaktifkan respons *Ping (ICMP Echo Request)* pada antarmuka WAN eksternal jika tidak diwajibkan oleh SLA pemantauan ISP, guna mencegah pemindai otomatis memetakan keberadaan perangkat (*host discovery*).
  3. **Hapus Layanan & Objek Yatim (Orphan Objects):** Bersihkan secara berkala definisi service, grup alamat, dan user lama yang sudah tidak lagi diasosiasikan dengan aturan firewall aktif.

### 3.4 Segmentasi Jaringan & Zonasi Berbasis Tingkat Kepercayaan (Trust Levels)
* **Konsep:** Mengelompokkan jaringan ke dalam kompartemen-kompartemen logis berdasarkan tingkat sensitivitas data dan profil risiko aset. Komunikasi lintas zona harus diatur secara ketat melalui aturan eksplisit.

```
[ UNTRUST ZONE ] (Internet Publik)
       |
       v (Filter L4/L7 + IPS + WAF)
[ DMZ ZONE ] (Public Portal / API Gateway / Mail Relay)
       |
       v (Strict Inter-Zone Rule: Hanya Port DB 5432 / 1433)
[ TRUST ZONE - DATACENTER ] (Server Farm, Database Clusters, SAN Storage)
       ^
       | (SAML SSO Auth + Full UTM: AV, App-Control, IPS)
[ TRUST ZONE - CAMPUS ] (Workstation Pegawai PUPR)
       |
       | (Total Dynamic Isolation)
[ ISOLATED ZONE ] (Guest Wi-Fi, Vendor CCTV, Smart Building IoT)
```

| Zona Keamanan | Tingkat Kepercayaan | Deskripsi & Aset di Dalamnya | Batasan Interaksi yang Diwajibkan |
| :--- | :---: | :--- | :--- |
| **WAN / Untrust** | Level 0 (Nol) | Internet publik, ISP eksternal, jaringan publik pihak ketiga. | Seluruh lalu lintas inbound diblokir secara default. Inbound hanya diizinkan melalui VIP port spesifik dengan perlindungan WAF/IPS. |
| **DMZ** | Level 1 (Rendah) | Server portal web publik, API reverse proxy, mail server gateway. | Terisolasi dari internal. **Dilarang memulai inisiasi koneksi ke arah Datacenter/LAN.** Respon data hanya diperbolehkan atas inisiasi dari database. |
| **Guest Wi-Fi** | Level 1 (Rendah) | Pengunjung kantor, perangkat pribadi tamu (*BYOD*). | Hanya memiliki akses keluar ke Internet via port 80/443. Diblokir 100% dari subnet pegawai dan server kantor. |
| **Campus LAN** | Level 2 (Menengah)| Laptop dinas pegawai, PC kantor, printer jaringan. | Wajib autentikasi pengguna (802.1X / SAML SSO). Akses ke Internet wajib melewati inspeksi Antivirus dan Web Filter. |
| **Datacenter** | Level 3 (Tinggi) | Server aplikasi produksi, database Oracle/Postgres, cluster SAN. | Sangat terisolasi. Hanya menerima koneksi dari aplikasi resmi. Tidak memiliki rute langsung ke Internet bebas (akses patch via proxy). |
| **OOB Mgmt** | Level 4 (Kritis) | Port konsol manajemen firewall, switch core, IPMI/iDRAC server. | Terpisah secara fisik (*air-gapped* atau VLAN non-routable). Akses hanya dapat dibuka dari Ruang Server/SOC terpercaya. |

---

## 4. CHECKLIST & PANDUAN HARDENING TEKNIS

Bagian ini menyajikan langkah-langkah implementasi teknis mendalam yang dikelompokkan ke dalam 7 kategori utama. Setiap butir panduan dilengkapi dengan **Rasional Keamanan (Alasan Teknis)** serta contoh implementasi atau acuan vendor.

---

### 4.1 Kategori A: Manajemen Akses Administratif (Management Plane)

Bidang manajemen adalah target utama penyerang karena pengambilalihan hak akses administrator memberikan kendali penuh terhadap seluruh arus data organisasi.

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST MANAJEMEN AKSES ADMINISTRATIF (MANAGEMENT PLANE)                                            |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Parameter Tindakan Pengerasan (Hardening Action)                            | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| A.1 | Nonaktifkan Protokol Komunikasi Manajemen Teks Terbuka (HTTP, Telnet)       | KRITIS  | [ ] OK  |
| A.2 | Terapkan Pembatasan Alamat IP Sumber Administrator (Trusted Hosts Whitelist)| KRITIS  | [ ] OK  |
| A.3 | Terapkan Autentikasi Multi-Faktor (MFA / 2FA) untuk Seluruh Akun Admin      | KRITIS  | [ ] OK  |
| A.4 | Ganti Akun dan Kata Sandi Bawaan Pabrik (Default Username "admin")          | TINGGI  | [ ] OK  |
| A.5 | Terapkan Kebijakan Kompleksitas Kata Sandi Administrator                    | TINGGI  | [ ] OK  |
| A.6 | Konfigurasikan Batas Waktu Ketidakaktifan Sesi (Admin Idle Session Timeout) | SEDANG  | [ ] OK  |
| A.7 | Aktifkan Proteksi Penguncian Akun dari Serangan Tebakan (Brute-Force Lockout)| TINGGI  | [ ] OK  |
| A.8 | Pisahkan Jalur Akses Administratif Menggunakan Port Out-of-Band (OOB) Khusus | TINGGI  | [ ] OK  |
| A.9 | Terapkan Pembagian Hak Akses Berbasis Peran (Role-Based Access Control / RBAC)| SEDANG | [ ] OK  |
| A.10| Pasang Pesan Peringatan Hukum Resmi pada Banner Halaman Login Masuk          | RENDAH  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

#### Detail Implementasi & Panduan Teknis:

* **A.1 Nonaktifkan Protokol Teks Terbuka (HTTP & Telnet):**
  * *Rasional:* Protokol HTTP (TCP 80) dan Telnet (TCP 23) mengirimkan kredensial kata sandi dalam bentuk teks biasa (*cleartext*). Siapa pun yang berada pada jalur jaringan yang sama (misal melalui *ARP Spoofing* atau penyadapan kabel) dapat membaca password administrator seketika menggunakan Wireshark.
  * *Rekomendasi:* Hanya izinkan protokol terenkripsi kuat: **HTTPS (TLS 1.2/1.3)** untuk antarmuka grafis Web GUI dan **SSH (SSHv2 dengan cipher modern AES-CTR/GCM)** untuk konsol CLI.
  * *Contoh Implementasi Vendor:*
    * *Fortinet (FortiOS):*
      ```fortios
      config system interface
          edit "port1"
              set allowaccess https ssh
          next
      end
      ```
    * *Palo Alto Networks (PAN-OS):* Pada *Device > Setup > Interfaces > Management*, nonaktifkan checkbox *Telnet* dan *HTTP*, hanya sisakan *HTTPS* dan *SSH*.
    * *pfSense / OPNsense:* Masuk ke *System > Advanced > Admin Access*, ubah protokol WebGUI menjadi *HTTPS*, dan aktifkan *Secure Shell (SSH)* dengan mematikan login root berbasis password.

* **A.2 Terapkan Pembatasan IP Administrator (Trusted Hosts / IP Whitelist):**
  * *Rasional:* Jika antarmuka manajemen dapat diakses dari alamat IP mana pun (`0.0.0.0/0`), maka firewall Anda rentan terhadap pemindaian botnet dan eksploitasi celah zero-day perangkat perimeter (seperti kerentanan SSL-VPN/Web GUI yang kerap terjadi).
  * *Rekomendasi:* Konfigurasikan firewall agar hanya merespons upaya koneksi administratif yang berasal dari blok subnet manajemen internal khusus (misal subnet SOC `10.10.2.0/24`) atau IP jump-host.
  * *Contoh Implementasi Vendor (FortiOS):*
    ```fortios
    config system admin
        edit "sec-admin"
            set trusthost1 10.10.2.0 255.255.255.0
            set trusthost2 192.168.100.5 255.255.255.255
        next
    end
    ```

* **A.3 Terapkan Autentikasi Multi-Faktor (MFA / 2FA):**
  * *Rasional:* Kata sandi statis rentan terhadap pencurian kredensial (*credential stuffing*, keylogger, phishing). MFA memastikan bahwa meskipun peretas mengetahui kata sandi admin, mereka tidak dapat login tanpa faktor kedua (*Time-based One-Time Password / TOTP* atau token fisik FIDO2).

* **A.6 Konfigurasikan Idle Session Timeout (Maksimal 10 Menit):**
  * *Rasional:* Mencegah pengambilalihan sesi administratif (*session hijacking*) saat laptop administrator ditinggalkan dalam keadaan login di meja kerja tanpa terkunci.
  * *Rekomendasi:* Tetapkan batas waktu timeout sesi GUI dan CLI maksimal **600 detik (10 menit)**.
  * *Contoh Implementasi Vendor (FortiOS):*
    ```fortios
    config system global
        set admintimeout 10
    end
    ```

---

### 4.2 Kategori B: Kebijakan Aturan Firewall (Rule / ACL Hardening)

Aturan firewall (*policies*) adalah inti dari pertahanan jaringan. Pengerasan pada level rule menjamin tidak ada celah lalu lintas gelap yang lolos tanpa teridentifikasi.

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST KEBIJAKAN ATURAN FIREWALL (RULE / ACL HARDENING)                                            |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Parameter Tindakan Pengerasan (Hardening Action)                            | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| B.1 | Pastikan Aturan Terakhir adalah Explicit Cleanup Deny-All dengan Log Aktif  | KRITIS  | [ ] OK  |
| B.2 | Hapus Seluruh Aturan yang Bersifat Permisif Luas (Any-to-Any Permit)        | KRITIS  | [ ] OK  |
| B.3 | Urutkan Aturan Mengikuti Hierarki Baku (Specific Inbound -> Broad Outbound) | TINGGI  | [ ] OK  |
| B.4 | Aktifkan Filter Anti-Spoofing (Blokir Martian IP & RFC 1918 di WAN)         | KRITIS  | [ ] OK  |
| B.5 | Pasang Integrasi Threat Intelligence (Feed Pemblokiran IP Botnet/C2/Tor)    | TINGGI  | [ ] OK  |
| B.6 | Validasi Ketiadaan Aturan Terbayang (Zero Policy Shadowing Check)           | TINGGI  | [ ] OK  |
| B.7 | Terapkan Penamaan Objek yang Terstandarisasi dan Deskripsi Lengkap          | SEDANG  | [ ] OK  |
| B.8 | Bersihkan Aturan yang Tidak Aktif / Memiliki Hit Count Nol (Zero Hit Pruning)| SEDANG | [ ] OK  |
| B.9 | Batasi Protokol Berisiko Tinggi Antar-Segmen Internal (Blokir SMB/RPC/RDP)  | TINGGI  | [ ] OK  |
| B.10| Terapkan Batas Waktu Kedaluwarsa pada Aturan Akses Sementara (Rule Expiry)   | SEDANG  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

#### Detail Implementasi & Panduan Teknis:

* **B.1 Aturan Pamungkas Explicit Cleanup Deny-All:**
  * *Rasional:* Meskipun beberapa firewall memiliki *implicit deny* bawaan, membuat aturan penolakan eksplisit (*explicit deny rule*) di baris terbawah memungkinkan administrator mengaktifkan opsi pencatatan log pada paket yang dibuang. Tanpa log penolakan ini, SOC tidak akan memiliki visibilitas terhadap aktivitas pemindaian port (*port scan*) atau serangan yang membentur firewall.
  * *Contoh Implementasi Universal:*
    * `Source Interface: ANY` | `Dest Interface: ANY` | `Source IP: ALL` | `Dest IP: ALL` | `Service: ALL` | `Action: DENY / DROP` | `Log: Enabled`.

* **B.3 Hierarki Baku Urutan Aturan (The Golden Rule Ordering):**
  * *Rasional:* Mesin firewall mengevaluasi paket dari atas ke bawah (*First-Match*). Urutan yang keliru dapat menyebabkan aturan keamanan dilewati.
  * *Urutan Wajib:*
    1. **Tingkat 1:** Aturan pembuangan paket cacat / *Anti-Spoofing / Drop Invalid TCP flags*.
    2. **Tingkat 2:** Pemblokiran reputasi ancaman (*Threat Feeds / Blacklist Botnet & Tor*).
    3. **Tingkat 3:** Akses masuk publik spesifik (*Inbound VIP / Reverse Proxy*) dengan filter ketat.
    4. **Tingkat 4:** Akses keluar pengguna terautentikasi (*Outbound LAN to WAN*) dengan inspeksi profil UTM lengkap.
    5. **Tingkat 5:** Akses antar-segmen internal terkontrol (*East-West Traffic*).
    6. **Tingkat 6:** Akses manajemen lokal (*Local-In Management Policies*).
    7. **Tingkat 7:** Aturan penolakan pembersihan akhir (*Explicit Cleanup Deny-All*).

* **B.4 Filter Anti-Spoofing (Martian IP Filtering):**
  * *Rasional:* Paket yang datang dari Internet publik (antarmuka WAN) tetapi memiliki alamat IP sumber privat (RFC 1918: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) atau IP localhost (`127.0.0.0/8`) adalah paket palsu (*spoofed*) yang dirancang untuk mengelabui filter internal.
  * *Rekomendasi:* Buat aturan penolakan di baris teratas antarmuka WAN untuk membuang seluruh paket yang memiliki IP sumber RFC 1918, atau aktifkan fitur *Unicast Reverse Path Forwarding (uRPF)* pada antarmuka L3.

---

### 4.3 Kategori C: Manajemen NAT & Port Forwarding yang Aman

Network Address Translation menjembatani alamat internal dengan jaringan publik. Konfigurasi yang ceroboh pada NAT dapat mengekspos jaringan privat secara tidak sengaja.

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST MANAJEMEN NAT & PORT FORWARDING                                                             |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Parameter Tindakan Pengerasan (Hardening Action)                            | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| C.1 | DILARANG Membuka Port Administrasi Perangkat (443/22) via VIP ke Publik     | KRITIS  | [ ] OK  |
| C.2 | Terapkan Port Translation (PAT) untuk Menyembunyikan Port Layanan Asli      | TINGGI  | [ ] OK  |
| C.3 | Batasi Alamat IP Sumber pada Seluruh Aturan Port Forwarding / Inbound VIP   | KRITIS  | [ ] OK  |
| C.4 | Lindungi Layanan Web Publik di Belakang WAF / Reverse Proxy Terisolasi      | TINGGI  | [ ] OK  |
| C.5 | Aktifkan Pengacakan Port Sumber (Port Randomization) pada Kebijakan SNAT    | SEDANG  | [ ] OK  |
| C.6 | Pisahkan Pool Alamat IP SNAT Pengguna Biasa dari IP Server Datacenter       | SEDANG  | [ ] OK  |
| C.7 | Pasang Rute Blackhole (Null0) untuk Mencegah Terjadinya Routing Loops       | TINGGI  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

#### Detail Implementasi & Panduan Teknis:

* **C.1 Larangan Keras Membuka Port Manajemen via Virtual IP (VIP):**
  * *Rasional:* Mempublikasikan port Web GUI (443) atau SSH (22) perangkat firewall langsung ke Internet publik dengan source `ANY` adalah penyebab utama jatuhnya ribuan firewall enterprise ke tangan kelompok peretas ransomware internasional (seperti eksploitasi CVE pada modul antarmuka web perimeter).
  * *Rekomendasi:* Akses manajemen **HANYA BOLEH** dibuka melalui koneksi VPN terenkripsi dengan autentikasi multi-faktor (MFA) atau jaringan OOB fisik.

* **C.7 Pemasangan Rute Blackhole (Null0 Routing):**
  * *Rasional:* Ketika firewall memiliki alokasi blok IP publik publik (misal `/28` atau `/29`) untuk NAT pool atau VIP, jika ada paket yang datang menuju alamat IP di dalam blok tersebut yang kebetulan sedang tidak aktif, paket tersebut berpotensi dipantulkan kembali (*routing loop*) antara firewall dan router gateway ISP, menghabiskan bandwidth antarmuka.
  * *Rekomendasi:* Pasang rute statis bertipe Blackhole untuk seluruh blok IP publik lokal dengan *Administrative Distance* tinggi (misal AD 254). Jika ada aturan VIP aktif, rute spesifik VIP akan menang. Jika VIP tidak aktif, paket langsung dibuang ke Blackhole tanpa memicu loop.

---

### 4.4 Kategori D: Logging, Monitoring & Telemetri

Tanpa pencatatan log yang tepat, administrator dan tim SOC buta terhadap apa yang sedang terjadi di dalam jaringan.

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST LOGGING, MONITORING & TELEMETRI                                                             |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Parameter Tindakan Pengerasan (Hardening Action)                            | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| D.1 | Konfigurasikan Pencatatan Log pada Akhir Sesi (Session-End / Close)         | TINGGI  | [ ] OK  |
| D.2 | Wajibkan Pengiriman Log Terenkripsi ke Server SIEM / Syslog Terpusat        | KRITIS  | [ ] OK  |
| D.3 | Terapkan Kebijakan Retensi Penyimpanan Log Minimal 180 Hari s.d. 365 Hari   | TINGGI  | [ ] OK  |
| D.4 | Sinkronisasikan Waktu Firewall Menggunakan Protokol NTP Tepercaya (Stratum 1/2)| TINGGI | [ ] OK  |
| D.5 | Aktifkan Log Perubahan Konfigurasi Administrator (Config Audit Log)        | KRITIS  | [ ] OK  |
| D.6 | Konfigurasikan Pemantauan Kinerja Perangkat via SNMPv3 (Nonaktifkan v1/v2c) | SEDANG  | [ ] OK  |
| D.7 | Tetapkan Ambang Batas Peringatan Dini Real-Time (CPU Spike, Link Down, Floods)| SEDANG | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

#### Detail Implementasi & Panduan Teknis:

* **D.1 Pencatatan Log pada Akhir Sesi (Session-End Logging):**
  * *Rasional:* Mencatat log pada awal sesi (*Session Start*) hanya memberi tahu bahwa sebuah koneksi dibuka, tetapi tidak mencatat berapa volume data yang dikirim (*bytes sent/received*), berapa lama koneksi berlangsung (*duration*), atau mengapa koneksi ditutup. Hal ini juga memicu pencatatan ganda (*duplicate logs*) yang memboroskan kapasitas disk.
  * *Rekomendasi:* Standarkan pencatatan log pada mode **Session-Close** pada seluruh aturan firewall yang mengizinkan lalu lintas.

* **D.4 Sinkronisasi Waktu Akurat via Network Time Protocol (NTP):**
  * *Rasional:* Log audit tanpa stempel waktu (*timestamp*) yang akurat tidak memiliki nilai hukum di pengadilan dan tidak dapat dikorelasikan oleh sistem SIEM. Jika jam firewall meleset 5 menit dari jam server web, tim analis insiden tidak akan pernah bisa merekonstruksi urutan kronologis terjadinya serangan.
  * *Rekomendasi:* Arahkan firewall ke minimal dua server NTP terpercaya (misal server NTP nasional atau Stratum-1/2 resmi).

---

### 4.5 Kategori E: Manajemen Patch, Firmware & Kerentanan

Firewall yang menjalankan versi firmware usang (*unpatched*) merupakan sasaran empuk peretas karena kode eksploitasinya biasanya sudah dipublikasikan secara luas di Internet (*Metasploit / GitHub*).

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST MANAJEMEN PATCH & KERENTANAN                                                                |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Parameter Tindakan Pengerasan (Hardening Action)                            | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| E.1 | Berlangganan Saluran Notifikasi Buletin Keamanan Vendor Resmi                | TINGGI  | [ ] OK  |
| E.2 | Terapkan Kebijakan Pembaruan Firmware Rutin (Siklus N-1 Mature Branch)      | TINGGI  | [ ] OK  |
| E.3 | Wajibkan Prosedur Pengujian Patch di Lingkungan Staging/Lab Non-Produksi   | TINGGI  | [ ] OK  |
| E.4 | Siapkan Dokumen Rencana Pengembalian Darurat (Rollback Plan) Sebelum Patch  | KRITIS  | [ ] OK  |
| E.5 | Lakukan Verifikasi Integritas File Firmware Menggunakan Hash SHA-256 Resmi  | TINGGI  | [ ] OK  |
| E.6 | Otomasikan Pembaruan Berkala Signatur Antivirus, IPS, dan WebFilter         | TINGGI  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

#### Detail Implementasi & Panduan Teknis:

* **E.2 Kebijakan Firmware N-1 (Mature Release):**
  * *Rasional:* Versi firmware paling mutakhir (*bleeding edge / .0 release*) sering kali masih mengandung *bug* fungsionalitas perangkat lunak yang belum terdeteksi. Sebaliknya, versi yang terlalu tua mengandung celah kerentanan yang sudah diketahui publik (*CVE*).
  * *Rekomendasi:* Terapkan versi firmware yang berada pada cabang stabil (*Mature / Long-Term Support / Patch rilis ke-4 ke atas*) yang telah diuji keandalannya oleh komunitas global.

* **E.5 Verifikasi Integritas File Firmware Menggunakan Hash SHA-256:**
  * *Rasional:* Memastikan file firmware yang diunduh dari situs vendor tidak rusak (*corrupt*) dan tidak disusupi oleh penyerang melalui serangan *supply chain* atau *Man-in-the-Middle*.
  * *Metode:* Selalu jalankan kalkulasi checksum sebelum proses instalasi:
    ```powershell
    Get-FileHash -Path "C:\firmware\image.out" -Algorithm SHA256
    ```
    Bandingkan hasilnya karakter per karakter dengan kode hash resmi yang tertera pada portal dukungan vendor.

---

### 4.6 Kategori F: Pengamanan Terhadap Serangan Umum

Firewall enterprise harus mampu secara otomatis mendeteksi dan meredam upaya penolakan layanan (*Denial of Service*) dan pemindaian jaringan (*Reconnaissance*).

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST PERTAHANAN TERHADAP SERANGAN UMUM                                                           |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Parameter Tindakan Pengerasan (Hardening Action)                            | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| F.1 | Aktifkan Proteksi Pertahanan TCP SYN Flood (SYN Proxy / SYN Cookie)         | KRITIS  | [ ] OK  |
| F.2 | Terapkan Pembatasan Laju Paket ICMP Flood (ICMP Rate Limiting)              | SEDANG  | [ ] OK  |
| F.3 | Terapkan Pembatasan Laju Koneksi Baru per Alamat IP Sumber (Connection Limit)| TINGGI  | [ ] OK  |
| F.4 | Aktifkan Fitur Pendeteksian dan Pemblokiran Port Scan Otomatis              | TINGGI  | [ ] OK  |
| F.5 | Aktifkan Fitur Drop Paket Berstatus Tidak Sah (Drop Invalid TCP State Flags)| TINGGI  | [ ] OK  |
| F.6 | Terapkan Sensor IPS dengan Tanda Tangan Perlindungan Exploitasi Kritis Aktif| KRITIS  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

#### Detail Implementasi & Panduan Teknis:

* **F.1 Proteksi TCP SYN Flood via SYN Proxy / SYN Cookies:**
  * *Rasional:* Menahan serangan banjir SYN yang menghabiskan memori firewall.
  * *Mekanisme Kerja:* Firewall mencegat paket SYN dari klien dan membalas dengan SYN-ACK menggunakan nomor urut kriptografis khusus (*SYN Cookie*) tanpa mengalokasikan ruang memori tabel sesi. Firewall baru membuka koneksi ke server tujuan setelah klien merespons dengan paket ACK yang sah.

* **F.5 Drop Invalid TCP State Flags:**
  * *Rasional:* Pemindai seperti Nmap sering mengirimkan kombinasi flag TCP yang ilegal secara spesifikasi RFC (misal *XMAS scan* yang mengaktifkan flag FIN, URG, dan PUSH secara bersamaan, atau *NULL scan* tanpa flag sama sekali) untuk memetakan jenis sistem operasi target.
  * *Rekomendasi:* Aktifkan opsi pembuangan paket otomatis terhadap paket dengan status TCP flag tidak sah di level inspeksi kernel firewall.

---

### 4.7 Kategori G: High Availability (HA) & Ketahanan Operasional

Ketersediaan tinggi (*High Availability*) memastikan bahwa kegagalan perangkat keras (*hardware failure*) pada satu unit tidak menyebabkan pemadaman jaringan secara keseluruhan (*zero downtime*).

```
+-------------------------------------------------------------------------------------------------------+
| CHECKLIST HIGH AVAILABILITY & KETAHANAN                                                               |
+-----+-----------------------------------------------------------------------------+---------+---------+
| No  | Parameter Tindakan Pengerasan (Hardening Action)                            | Tingkat | Status  |
+-----+-----------------------------------------------------------------------------+---------+---------+
| G.1 | Terapkan Kluster Redundansi Dua Unit Menggunakan Mode Active-Passive        | KRITIS  | [ ] OK  |
| G.2 | Hubungkan Minimal Dua Jalur Kabel Heartbeat Fisik Terpisah Tanpa Switch     | KRITIS  | [ ] OK  |
| G.3 | Aktifkan Sinkronisasi Sesi Aktif (Session State Synchronization)            | TINGGI  | [ ] OK  |
| G.4 | Konfigurasikan Pemantauan Antarmuka (Interface Link Monitoring / pmon)      | KRITIS  | [ ] OK  |
| G.5 | Jadwalkan Pencadangan Konfigurasi Otomatis Harian ke Repositori Terenkripsi | KRITIS  | [ ] OK  |
| G.6 | Lakukan Pengujian Simulasi Kegagalan Perangkat (Failover Test) Berkala      | TINGGI  | [ ] OK  |
+-----+-----------------------------------------------------------------------------+---------+---------+
```

#### Detail Implementasi & Panduan Teknis:

* **G.2 Redundansi Jalur Heartbeat Fisik (Mencegah Split-Brain):**
  * *Rasional:* Jika kabel komunikasi detak jantung (*heartbeat*) putus, unit sekunder akan mengira unit primer mati, sehingga kedua unit akan sama-sama mempromosikan diri menjadi unit aktif (**Kondisi Split-Brain**). Hal ini memicu benturan alamat IP (*IP conflict*) dan kelumpuhan jaringan total.
  * *Rekomendasi:* Wajib menggunakan minimal **2 link fisik terpisah** (misal port `HA1` dan port `HA2`) yang ditancapkan secara langsung (*direct cable*) antar kedua sasis firewall tanpa melalui switch perantara.

* **G.4 Pemantauan Antarmuka (Interface Link Monitoring / pmon):**
  * *Rasional:* Jika salah satu kabel uplink Internet atau kabel downlink ke Core Switch putus, unit firewall primer tidak mati secara listrik. Tanpa pemantauan antarmuka, kluster tidak akan melakukan failover, sehingga trafik tetap dialihkan ke unit primer yang kabelnya putus tersebut (*blackholing traffic*).
  * *Rekomendasi:* Masukkan seluruh port fisik penting ke dalam daftar antarmuka yang dipantau agar pengurangan prioritas (*priority penalty*) dieksekusi seketika saat ada kabel yang terputus.

---

## 5. PROSEDUR OPERASIONAL STANDAR (SOP)

Peralatan keamanan yang tangguh akan kehilangan keefektifannya jika tidak diimbangi dengan disiplin tata kelola manusia. Bagian ini mendefinisikan empat SOP formal yang wajib diintegrasikan ke dalam operasional IT organisasi.

---

### 5.1 SOP Manajemen Perubahan Konfigurasi (Change Control)

Setiap perubahan pada aturan, rute, atau objek firewall dilarang keras dilakukan secara spontan tanpa tiket resmi.

```
+---------------------------------------------------------------------------------------------------+
|                        ALUR KERJA MANAJEMEN PERUBAHAN FIREWALL (CHANGE CONTROL)                   |
+---------------------------------------------------------------------------------------------------+
  [ Pemohon ] -> Mengajukan Formulir Tiket Perubahan Firewall (CR Form)
       |
       v
  [ Security Engineer ] -> Melakukan Verifikasi Teknis, Analisis Risiko, & Uji Shadowing
       |
       v
  [ Change Advisory Board (CAB) ] -> Rapat Evaluasi & Pemberian Persetujuan Resmi (Approval)
       |
       v
  [ Pelaksanaan Maintenance ] -> Eksekusi Konfigurasi pada Jendela Waktu Perawatan (Maintenance Window)
       |
       v
  [ Verifikasi Pasca-Perubahan ] -> Uji Konektivitas Aplikasi & Validasi Tidak Ada Dampak Samping
       |
       +---> [ GAGAL ] -> Eksekusi Rencana Pengembalian Darurat (Rollback Plan) seketika
       |
       v [ BERHASIL ]
  [ Dokumentasi & Penutupan Tiket ] -> Pembaruan Inventaris Rule & Penutupan Tiket Resmi
```

1. **Pengajuan (Request Phase):** Pemohon mengisi formulir permohonan pembukaan akses dengan melampirkan justifikasi bisnis yang jelas, identitas sistem, dan tanggal kedaluwarsa akses.
2. **Review Keamanan (Security Review):** Tim Network Security memastikan prinsip least privilege terpenuhi (tidak ada port wildcard/ANY) dan tidak terjadi tumpang tindih aturan (*shadowing*).
3. **Persetujuan CAB (CAB Approval):** Dewan CAB meninjau dampak operasional dan memberikan persetujuan jadwal eksekusi pada jendela pemeliharaan (*Maintenance Window*).
4. **Pencadangan Pra-Eksekusi (Pre-Change Backup):** Administrator wajib melakukan *download backup configuration* beberapa menit sebelum perintah modifikasi pertama dieksekusi.
5. **Verifikasi & Rollback:** Jika dalam waktu 15 menit pasca-perubahan terdeteksi anomali performa atau keluhan konektivitas, konfigurasi cadangan wajib segera dipulihkan kembali (*Rollback*).

---

### 5.2 SOP Audit & Evaluasi Berkala

Audit firewall wajib diselenggarakan secara teratur untuk memastikan postur keamanan tidak mengalami penurunan kualitas (*configuration drift*).

| Frekuensi Audit | Pelaksana | Ruang Lingkup Pemeriksaan | Output Dokumen |
| :--- | :--- | :--- | :--- |
| **Mingguan (Weekly)** | Network Admin / NOC | Pemeriksaan utilisasi CPU/RAM, status sinkronisasi kluster HA, kegagalan login berulang, dan status update signatur AV/IPS. | Health Check Checklist |
| **Bulanan (Monthly)** | Security Engineer | Analisis top 10 aturan terbanyak digunakan, verifikasi log admin, pemeriksaan rute baru, dan evaluasi tiket perubahan yang kedaluwarsa. | Monthly Security Review |
| **Triwulanan (Quarterly)**| Tim SOC / Internal Auditor | Uji penetrasi eksternal, pembersihan aturan mati (*Zero Hit Count*), verifikasi kepatuhan ISO 27001 / PCI-DSS, dan simulasi failover. | Audit Compliance Report |
| **Tahunan (Annual)** | Auditor Eksternal Independen | Audit menyeluruh end-to-end arsitektur perimeter, evaluasi firmware lifecycle, dan review kebijakan tata kelola keamanan siber. | External Certification Audit |

---

### 5.3 SOP Resertifikasi & Review Aturan (Rule Recertification)

Aturan firewall yang dibiarkan hidup selamanya tanpa pemilik aset yang jelas adalah sumber utama kerapuhan perimeter.
1. Setiap aturan firewall yang dibuat **wajib memiliki masa berlaku maksimal 12 bulan** (atau maksimal 30 hari untuk akses proyek/kontraktor sementara).
2. Setiap 90 hari, sistem akan menghasilkan daftar aturan yang akan kedaluwarsa.
3. Tim Security mengirimkan konfirmasi kepada pemilik aplikasi (*Application Owner*):
   * *"Apakah server dan port pada aturan ini masih digunakan untuk operasional bisnis?"*
4. Jika pemilik aplikasi tidak memberikan konfirmasi balasan dalam waktu 14 hari kerja, aturan tersebut akan dinonaktifkan (*disabled*) selama 14 hari berikutnya.
5. Jika selama masa penonaktifan tidak ada laporan gangguan operasional bisnis yang sah, aturan tersebut akan **dihapus secara permanen** dari basis data firewall.

---

### 5.4 SOP Tanggap Insiden Keamanan Terkait Firewall

Panduan taktis bagi tim SOC/NOC saat terjadi insiden siber aktif (misal serangan DDoS masif atau indikasi peretasan sistem internal):

```
+---------------------------------------------------------------------------------------------------+
|                        ALUR TANGGAP INSIDEN KEAMANAN SIBER PERIMETER                              |
+---------------------------------------------------------------------------------------------------+
  [ 1. IDENTIFIKASI ] -> Deteksi anomali lalu lintas via SIEM/PRTG (Lonjakan bandwidth, botnet alert)
       |
       v
  [ 2. PEMBATASAN ]   -> Isolasi aset terdampak seketika (Karantina IP sumber, matikan VIP terkait)
       |
       v
  [ 3. ERADIKASI ]    -> Tambahkan IP penyerang ke Threat Feed Blacklist & perkuat sensor IPS
       |
       v
  [ 4. PEMULIHAN ]    -> Validasi bahwa lalu lintas berbahaya telah hilang & hidupkan kembali layanan
       |
       v
  [ 5. PASCA-INSIDEN ]-> Susun Laporan Post-Mortem & perbarui baseline konfigurasi agar tidak terulang
```

1. **Fase 1: Identifikasi (Detection & Identification):** Konfirmasikan kebenaran insiden melalui korelasi log SIEM, alamat IP sumber penyerang, pola serangan (L4 SYN Flood atau L7 HTTP Flood), dan port target.
2. **Fase 2: Pembatasan Darurat (Containment):**
   * Buat aturan *Emergency Blacklist* di baris teratas firewall untuk membuang seluruh paket dari alamat IP atau subnet penyerang.
   * Jika server web di DMZ terkompromi, putuskan rute inter-zone dari DMZ ke internal untuk mencegah pergerakan lateral.
3. **Fase 3: Eradikasi (Eradication):** Bersihkan artefak ancaman, tutup port yang dieksploitasi, dan mutakhirkan signatur IPS ke versi darurat terbaru.
4. **Fase 4: Pemulihan (Recovery):** Buka kembali layanan secara bertahap sambil memantau grafik utilisasi antarmuka secara ketat selama minimal 4 jam pengawasan intensif.
5. **Fase 5: Pembelajaran Pasca-Insiden (Post-Incident Review):** Dokumentasikan kronologi serangan, penyebab akar masalah (*root cause*), dan rekomendasi mitigasi permanen dalam laporan resmi *Post-Mortem*.

---

## 6. TEMPLATE & DOKUMEN KERJA PRAKTIS

Gunakan format baku berikut untuk standarisasi operasional sehari-hari:

### 6.1 Template Inventaris Aturan Firewall (Rule Inventory Matrix)

Setiap aturan yang ada di firewall wajib didokumentasikan ke dalam tabel matriks resmi berikut:

| ID Rule | Zona Asal | Zona Tujuan | IP Sumber / Objek | IP Tujuan / Objek | Port & Protokol | Aksi | Justifikasi Bisnis | Pemilik Aset (Owner) | No. Tiket CR | Tanggal Review |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- | :--- | :---: | :---: |
| **FW-001** | WAN | DMZ | `GRP_Whitelist_BKN` | `H_WebSSO_10.10.230.15` | `TCP/443 (HTTPS)` | ACCEPT | Integrasi API Kepegawaian BKN | Biro Kepegawaian | CR-2026-0891 | 01-Jan-2027 |
| **FW-002** | Campus | WAN | `NET_LAN_Pegawai` | `ALL (Internet)` | `TCP/80, 443, UDP/53` | ACCEPT | Akses browsing dinas pegawai | Divisi TI | CR-2026-0012 | 15-Jun-2027 |
| **FW-003** | DMZ | Datacenter| `H_WebSSO_10.10.230.15` | `H_DB_Prod_10.10.240.20` | `TCP/5432 (Postgres)` | ACCEPT | Koneksi web portal ke database | Tim Pengembang | CR-2026-0450 | 10-Mar-2027 |
| **FW-099** | ANY | ANY | `ALL` | `ALL` | `ALL` | DENY | Explicit Cleanup Rule | Security Team | POLICY-BASE | PERMANEN |

---

### 6.2 Template Formulir Permohonan Pembukaan Akses (Firewall Change Request)

```
====================================================================================================
               FORMULIR PERMOHONAN PEMBUKAAN AKSES FIREWALL (FIREWALL CHANGE REQUEST)
====================================================================================================
NOMOR TIKET        : CR-FW-________-________
TANGGAL PENGAJUAN  : [ DD / MM / YYYY ]
NAMA PEMOHON       : _______________________________________________________________________________
UNIT KERJA / DIVISI: _______________________________________________________________________________
KONTAK / EMAIL     : _______________________________________________________________________________
----------------------------------------------------------------------------------------------------
DETAIL TEKNIS PERMOHONAN:
1. Alamat IP Sumber (Source IP/Subnet)      : _______________________________________________________
2. Alamat IP Tujuan (Destination IP/Subnet) : _______________________________________________________
3. Protokol & Nomor Port Aplikasi           : [ ] TCP  [ ] UDP  Port: _______________________________
4. Arah Aliran Lalu Lintas                  : [ ] Inbound dari Internet  [ ] Outbound ke Internet
                                              [ ] Antar-Subnet Internal  [ ] VPN Site-to-Site
5. Durasi Kebutuhan Akses                   : [ ] Permanen (Review Tahunan)
                                              [ ] Sementara, s.d. Tanggal: [ DD / MM / YYYY ]
----------------------------------------------------------------------------------------------------
JUSTIFIKASI BISNIS & KEBUTUHAN OPERASIONAL:
(Jelaskan fungsi aplikasi, alasan mengapa port ini harus dibuka, dan dampak jika akses ditolak)
____________________________________________________________________________________________________
____________________________________________________________________________________________________
----------------------------------------------------------------------------------------------------
ANALISIS DAMPAK KEAMANAN (Diisi oleh Tim Security):
[ ] Risiko Rendah    [ ] Risiko Menengah    [ ] Risiko Tinggi (Wajib Persetujuan CISO)
Catatan Analisis     : _______________________________________________________________________________
Profil UTM Tambahan  : [ ] Antivirus  [ ] IPS Sensor  [ ] Web Filter  [ ] WAF Profile
----------------------------------------------------------------------------------------------------
PERSETUJUAN & OTORISASI:
Pemohon (Requester)            Lead Security Engineer           Ketua Dewan Perubahan (CAB Chair)

(____________________)         (____________________)           (____________________)
Tgl:                           Tgl:                             Tgl:
====================================================================================================
```

---

### 6.3 Template Lembar Hasil Audit Kepatuhan (Compliance Audit Sheet)

```
====================================================================================================
               LEMBAR HASIL AUDIT KEPATUHAN HARDENING FIREWALL (AUDIT REPORT)
====================================================================================================
NAMA PERANGKAT / HOSTNAME : ________________________________________________________________________
MODEL / VENDOR PERANGKAT  : ________________________________________________________________________
VERSI FIRMWARE            : ________________________________________________________________________
TANGGAL AUDIT             : [ DD / MM / YYYY ]
NAMA AUDITOR              : ________________________________________________________________________
----------------------------------------------------------------------------------------------------
RINGKASAN SKOR KEPATUHAN:
* Total Parameter Diperiksa : 50 Butir
* Parameter Memenuhi (PASS) : _____ Butir ( _____ % )
* Parameter Gagal (FAIL)    : _____ Butir ( _____ % )
* Kategori Tingkat Kematangan: [ ] SANGAT BAIK (>90%)   [ ] MEMADAI (75-90%)   [ ] KRITIS (<75%)
----------------------------------------------------------------------------------------------------
TEMUAN KETIDAKSESUAIAN UTAMA (NON-COMPLIANCE FINDINGS):
1. [ID Temuan: _______] : _________________________________________________________________________
   - Tingkat Risiko      : [ ] Rendah   [ ] Menengah   [ ] Tinggi   [ ] Kritis
   - Rencana Remediasi   : _________________________________________________________________________
   - Batas Waktu Perbaikan: [ DD / MM / YYYY ] | PIC: ______________________________________________

2. [ID Temuan: _______] : _________________________________________________________________________
   - Tingkat Risiko      : [ ] Rendah   [ ] Menengah   [ ] Tinggi   [ ] Kritis
   - Rencana Remediasi   : _________________________________________________________________________
   - Batas Waktu Perbaikan: [ DD / MM / YYYY ] | PIC: ______________________________________________
----------------------------------------------------------------------------------------------------
TANDA TANGAN PENGESAHAN:
Auditor Keamanan Siber                              Kepala Penanggung Jawab Infrastruktur

(_______________________________)                  (_______________________________)
====================================================================================================
```

---

## 7. REFERENSI & PENYELARASAN STANDAR

Panduan dokumentasi ini disusun dengan menyelaraskan klausul dan kontrol dari berbagai kerangka kerja standar keamanan siber internasional:

1. **Center for Internet Security (CIS) Controls v8 & CIS Firewall Benchmark v2.0:**
   * *CIS Safeguard 4.4:* Menetapkan dan memelihara arsitektur keamanan firewall yang membatasi akses antar-zona.
   * *CIS Safeguard 4.5:* Melakukan peninjauan aturan firewall (*automated/manual rule review*) secara berkala.
   * *CIS Safeguard 6.1 s.d. 6.8:* Penerapan kontrol manajemen akses administratif, penonaktifan protokol teks terbuka, dan pembatasan *trusted hosts*.
2. **National Institute of Standards and Technology (NIST) SP 800-41 Rev. 1:**
   * *Guidelines on Firewalls and Firewall Policy:* Rekomendasi penetapan kebijakan *default deny*, segregasi antarmuka DMZ, dan integrasi logging ke SIEM.
3. **ISO/IEC 27001:2022 (Sistem Manajemen Keamanan Informasi):**
   * *Kontrol A.8.20 (Network Security):* Pengamanan perangkat jaringan dan perlindungan informasi yang melintasinya.
   * *Kontrol A.8.22 (Segregation in Networks):* Pemisahan kelompok layanan informasi, pengguna, dan sistem informasi ke dalam zona jaringan yang berbeda.
   * *Kontrol A.8.24 (Use of Cryptography):* Penggunaan standar cipher enkripsi modern untuk melindungi kerahasiaan data.
4. **Payment Card Industry Data Security Standard (PCI-DSS) v4.0:**
   * *Requirement 1:* Membangun dan memelihara kontrol keamanan jaringan (firewall) untuk melindungi lingkungan data pemegang kartu (*Cardholder Data Environment - CDE*), termasuk larangan aturan any-any dan kewajiban dokumentasi justifikasi bisnis pada setiap rule aktif.
