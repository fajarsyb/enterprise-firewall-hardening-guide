# ENTERPRISE NEXT-GENERATION FIREWALL (NGFW) ARCHITECTURE & HARDENING GUIDE
**The Vendor-Agnostic Golden Baseline for Network Security Architects & Senior Engineers**
*Standard Compliance: CIS Controls v8, NIST SP 800-41 Rev. 1, ISO/IEC 27001:2022, PCI-DSS v4.0*

---

## Ringkasan Eksekutif & Prinsip Desain

Panduan ini disusun sebagai standar arsitektur dan konfigurasi firewall tingkat enterprise yang bersifat independen terhadap merk (*vendor-agnostic*). Prinsip-prinsip yang dirumuskan di sini berlaku universal baik untuk implementasi perangkat keras seperti **Fortinet FortiGate, Palo Alto Networks, Cisco Secure Firewall, Check Point Quantum, Juniper SRX**, maupun solusi open-source berstandar carrier seperti **pfSense / OPNsense**.

Tujuan dokumen ini adalah memberikan cetak biru (*blueprint*) praktis untuk merancang perimeter yang kokoh, mencegah pergerakan lateral ancaman, memaksimalkan throughput perangkat keras, dan memenuhi standar kepatuhan regulasi keamanan informasi internasional.

---


---

# Modul 01: Arsitektur Jaringan & Segmentasi Zero-Trust

## 1. Konsep Dasar Desain Zona Keamanan (Security Zones)

Firewall enterprise modern tidak boleh dioperasikan sebagai router sederhana dengan filter port statis. Arsitektur harus dirancang berbasis **Zona Keamanan (Security Zones)** yang memisahkan lalu lintas berdasarkan tingkat kepercayaan (*Trust Levels*) dan fungsi fungsional aset.

### Pembagian Zona Standar Enterprise:
1. **Untrust / WAN Zone:**
   * Menghubungkan firewall ke Internet publik, sirkuit ISP pihak ketiga, atau peering eksternal.
   * *Postur:* Nol kepercayaan (*Zero Trust*), seluruh trafik inbound diblokir secara default.
2. **DMZ (Demilitarized Zone):**
   * Menampung server publik yang memerlukan akses dari Internet (Web Portal, API Gateway, Mail Relay).
   * *Prinsip Isolasi:* Server di DMZ **TIDAK PERNAH** diizinkan memulai koneksi (*initiate connection*) ke jaringan privat internal (Server Farm / LAN). Jika DMZ disusupi, penyerang terperangkap di DMZ.
3. **Internal Server Farm / Core Datacenter Zone:**
   * Menampung basis data, server aplikasi internal, active directory, dan cluster penyimpanan (SAN/NAS).
   * Akses hanya dibuka dari segmen pengguna tertentu pada port aplikasi spesifik.
4. **Campus LAN / User Zone:**
   * Menampung perangkat kerja pegawai, laptop, dan telepon IP.
   * Wajib melalui autentikasi identitas (802.1X / SAML SSO) sebelum memperoleh hak akses ke luar atau ke server farm.
5. **Guest Wi-Fi Zone:**
   * Segmen dinamis untuk tamu dan pengunjung.
   * Dibatasi murni untuk browsing Internet (HTTP/HTTPS dan DNS). Terisolasi 100% dari subnet internal mana pun.
6. **Out-of-Band (OOB) Management Zone:**
   * Jaringan fisik/VLAN khusus yang tidak membawa trafik produksi. Digunakan untuk konsol IPMI, iLO/iDRAC, switch console, dan antarmuka manajemen firewall.
7. **Specialized IoT & Surveillance Zone:**
   * Segmen terisolasi untuk CCTV IP, sistem akses pintu (BMS), dan sensor gedung. Wajib memblokir lalu lintas SMB/RPC keluar untuk mencegah penyebaran worm/ransomware.

---

## 2. Micro-segmentation & East-West Traffic Filtering

Banyak organisasi hanya fokus pada keamanan perimeter (North-South: Internet <-> Jaringan Internal). Namun, jika satu workstation pegawai terinfeksi malware, ketiadaan filter **East-West** (antar-subnet internal) memungkinkan malware melakukan pergerakan lateral (*lateral movement*) tanpa hambatan.

### Pedoman Implementasi:
* Setiap antarmuka VLAN internal harus didefinisikan sebagai antarmuka L3 di firewall (atau diinspeksi via VRF / Inter-VDOM / Virtual Firewall).
* Blokir protokol berisiko tinggi antar-segmen internal:
  * **SMB (TCP 445 / 139):** Hanya diizinkan ke File Server resmi. Blokir antar workstation pegawai.
  * **RDP (TCP 3389) & SSH (TCP 22):** Hanya diizinkan melalui Jump Server / PAM (Privileged Access Management).
  * **NetBIOS (UDP 137/138, TCP 139):** Matikan atau blokir di seluruh segmen internal.

---

## 3. Hardware Offloading (ASIC) vs Software Architecture

Dalam memilih dan mengonfigurasi firewall enterprise, pahami jalur pemrosesan paket (*data plane*):
* **ASIC / Hardware-Accelerated Path:**
  * Protokol standar seperti AES-GCM, perutean L4 IPv4/IPv6, dan TCP SYN handling langsung diolah oleh chip prosesor jaringan (misal Fortinet NP, Palo Alto SP3, dsb). Latensi berada di level sub-mikrodetik.
* **CPU / Software Slow-Path:**
  * Fitur canggih seperti Deep SSL Decryption, Antivirus stream inspection, dan protokol usang yang tidak didukung ASIC akan dilempar ke CPU host.
  * *Rekomendasi:* Selalu monitor utilisasi CPU saat mengaktifkan modul UTM, dan pastikan cipher VPN yang dipilih kompatibel dengan hardware offloading vendor.


---

# Modul 02: Siklus Hidup & Urutan Aturan Kebijakan (Rule Ordering)

## 1. Mekanisme Evaluasi Kebijakan (Top-Down First-Match)

Sebagian besar mesin firewall mengevaluasi kebijakan dari **atas ke bawah (Top-Down)**. Paket yang datang akan dicocokkan dengan rule pertama yang memenuhi seluruh kriteria (Interface Asal, Interface Tujuan, Alamat Asal, Alamat Tujuan, Service, Schedule). Begitu cocok, aksi (Accept / Deny) langsung dieksekusi dan pencocokan berhenti.

> **PENTING:** Aturan yang salah urutan (*Rule Order Inversion*) dapat melumpuhkan seluruh sistem pertahanan keamanan Anda.

---

## 2. Hierarki Baku Urutan Aturan (The Golden Rule Ordering)

Susun firewall policy Anda mengikuti 7 tingkatan baku berikut:

```
+-------------------------------------------------------------------------------+
| TINGKAT 1: ANTI-SPOOFING & INVALID STATE DROP                                 |
| - Drop paket invalid TCP flags, IP RFC 1918 di interface publik (Martians)    |
+-------------------------------------------------------------------------------+
                                       |
                                       v
+-------------------------------------------------------------------------------+
| TINGKAT 2: THREAT INTELLIGENCE & REPUTATION IP BLOCKLIST                     |
| - Drop IP pemindai (Scanner), C2 Botnet, Tor Exit Nodes, AbuseIPDB/BIN Feeds |
+-------------------------------------------------------------------------------+
                                       |
                                       v
+-------------------------------------------------------------------------------+
| TINGKAT 3: SPECIFIC INBOUND SERVICES (VIP / DNAT)                             |
| - Port Forwarding ke DMZ / Server dengan filter IP sumber seketat mungkin     |
+-------------------------------------------------------------------------------+
                                       |
                                       v
+-------------------------------------------------------------------------------+
| TINGKAT 4: OUTBOUND USER INTERNET TRAFFIC (WITH UTM & IDENTITY)               |
| - Akses browsing pegawai yang diautentikasi (SSO/802.1X) + AV + Web Filter    |
+-------------------------------------------------------------------------------+
                                       |
                                       v
+-------------------------------------------------------------------------------+
| TINGKAT 5: INTER-ZONE CONTROLLED EAST-WEST FLOW                               |
| - Komunikasi terkontrol User ke Server Farm, atau antar-server database       |
+-------------------------------------------------------------------------------+
                                       |
                                       v
+-------------------------------------------------------------------------------+
| TINGKAT 6: MANAGEMENT & CONTROL PLANE LOCAL-IN POLICIES                       |
| - Akses administratif terisolasi hanya dari subnet SOC / Admin Whitelist     |
+-------------------------------------------------------------------------------+
                                       |
                                       v
+-------------------------------------------------------------------------------+
| TINGKAT 7: EXPLICIT CLEANUP DENY-ALL & LOG                                    |
| - Menolak seluruh sisa paket yang tidak cocok dengan pencatatan log audit     |
+-------------------------------------------------------------------------------+
```

---

## 3. Bahaya Rule Shadowing & Redundansi

* **Rule Shadowing (Aturan Terbayang):** Terjadi jika aturan yang lebih longgar/luas diletakkan **di atas** aturan yang spesifik/ketat. Akibatnya, aturan ketat tidak akan pernah tersentuh (*zero hits*).
  * *Contoh Fatal:* Meletakkan rule `Permit Inbound ALL` di atas rule `Deny Botnet Scanner`. Botnet scanner akan lolos karena rule permit dievaluasi lebih awal.
* **Overly Broad Rules:** Hindari menggunakan alamat `all` atau `0.0.0.0/0` bersamaan dengan service `ALL` pada aksi `ACCEPT`. Batasi service hanya pada port aplikasi yang sah.

---

## 4. Standar Penamaan Objek & Grup (Naming Conventions)

Standarisasi nama mutlak diperlukan agar konfigurasi mudah diaudit oleh tim pihak ketiga:

| Kategori Objek | Format Penamaan Standar | Contoh |
| :--- | :--- | :--- |
| **Host Tunggal** | `H_<Deskripsi>_<IP>` | `H_WebProd01_10.10.230.15` |
| **Network / Subnet** | `NET_<VLAN/Deskripsi>_<Prefix>` | `NET_ServerFarm_10.10.230.0_24` |
| **IP Range / Pool** | `RNG_<Deskripsi>_<Start>_<End>` | `RNG_DHCP_Pool_10.10.1.100_200` |
| **Address Group** | `GRP_<Fungsi/Zona>` | `GRP_Production_Web_Servers` |
| **Custom Service** | `SVC_<Protokol>_<Port>` | `SVC_TCP_8443_SmartZone` |
| **Firewall Policy** | `<Aksi>_<ZonaAsal>_to_<ZonaTujuan>_<Service>` | `ALLOW_LAN_to_WAN_Internet_Browsing` |

---

## 5. Tata Kelola Pembersihan Objek (Hygiene Routine)

* Lakukan audit berkala setiap 90 hari.
* Identifikasi objek yang memiliki **0 hit count / zero references** di kebijakan mana pun.
* Hapus objek alamat dan custom service yang sudah tidak digunakan untuk mencegah penumpukan konfigurasi (*configuration clutter*).


---

# Modul 03: Standar Kriptografi VPN & IPsec Modern

## 1. Evolusi Algoritma: Mengapa Harus Beralih ke AEAD (AES-GCM)?

Dalam konfigurasi IPsec tradisional, keamanan paket mengandalkan dua proses terpisah:
1. **Enkripsi Simetris:** Menggunakan AES-CBC, 3DES, atau DES.
2. **Otentikasi Integritas:** Menggunakan HMAC-SHA256, HMAC-SHA1, atau MD5.

Proses dua langkah (*Double-Pass*) ini membebani prosesor karena setiap bit data harus diolah dua kali.

### Keunggulan AES-GCM (Authenticated Encryption with Associated Data):
* Menggabungkan enkripsi dan verifikasi integritas dalam **satu rumus matematika tunggal (Single-Pass)**.
* **40% – 50% lebih efisien** dalam komputasi hardware ASIC/CPU modern.
* **Throughput jauh lebih tinggi dan latensi lebih rendah.**
* Kebal terhadap serangan klasik seperti *Padding Oracle Attacks* yang sering menghantui mode CBC.

---

## 2. Matriks Standar Kriptografi Golden Baseline

Gunakan matriks berikut saat menegosiasikan tunnel Site-to-Site maupun Remote Access VPN:

| Komponen VPN | Standar Aman (Golden Baseline) | Dapat Diterima (Transisi) | DILARANG KERAS (Vulnerable) |
| :--- | :--- | :--- | :--- |
| **IKE Version** | **IKEv2** (Efisiensi tinggi, EAP, NAT-T native) | IKEv1 (Jika peer lama) | IKEv1 Aggressive Mode dengan PSK |
| **Phase 1 Cipher** | **AES-256-GCM** atau **AES-128-GCM** | AES-256-CBC + SHA256 | **MD5, SHA-1, DES, 3DES** |
| **Phase 1 PRF / Hash** | **PRF-SHA384** atau **PRF-SHA256** | SHA-256 | **MD5, SHA-1** |
| **Diffie-Hellman Group** | **Group 19 (Curve25519) / 20 / 21** | Group 14 (2048-bit), Group 15 | **Group 1 (768b), Group 2 (1024b), Group 5** |
| **Phase 2 Encryption** | **AES-256-GCM** atau **AES-128-GCM** | AES-256-CBC (SHA-256) | **MD5, SHA-1, DES, 3DES, NULL** |
| **PFS (Perfect Forward Secrecy)**| **Aktif (PFS Enable)**, DH Group 19/14 | Aktif (PFS Enable), DH 14 | Nonaktif (PFS Disable) |
| **Dead Peer Detection (DPD)** | **Aktif (On-Demand / Periodic)** | Aktif | Nonaktif |

---

## 3. Masalah Klasik: MTU & TCP MSS Clamping

Paket IPsec menambahkan header enkripsi berukuran 50–70 byte pada setiap paket IP. Jika MTU interface fisik WAN adalah 1500 byte, maka paket data 1500 byte dari klien internal akan membengkak menjadi ~1560 byte saat dienkripsi.

### Akibat Fragmentasi:
* Jika flag DF (*Don't Fragment*) aktif: Paket di-drop oleh ISP, aplikasi web mengalami *loading hang* atau *timeout*.
* Jika terfragmentasi: Beban reassembly di router lawan meningkat, memicu packet loss.

### Solusi Wajib: TCP MSS Clamping
Paksa klien TCP untuk menegosiasikan ukuran segmen maksimum yang lebih kecil pada fase 3-way handshake:
$$\text{MSS Ideal} = \text{MTU WAN (1500)} - \text{IP/TCP Header (40)} - \text{IPsec Overhead (70-100)} \approx \mathbf{1350 \text{ atau } 1360 \text{ Byte}}$$

* Terapkan nilai **MSS 1350** pada policy atau virtual interface tunnel IPsec.

---

## 4. Standar Keamanan Remote Access VPN

Untuk pengguna jarak jauh (Teleworkers / Mobile Users):
1. **Wajib Multi-Factor Authentication (MFA):** Jangan izinkan autentikasi berbasis kata sandi statis tunggal.
2. **Device Posture Assessment:** Periksa apakah laptop klien memiliki antivirus aktif, OS ter-update, dan hard drive terenkripsi (BitLocker/FileVault).
3. **Split-Tunneling Policy:** Tinjau secara ketat. Jika split-tunneling aktif, pastikan ada endpoint inspection untuk mencegah laptop menjadi jembatan (*pivot point*) dari Internet ke jaringan kantor.


---

# Modul 04: Arsitektur NAT & Keamanan Virtual IP (VIP/DNAT)

## 1. Arsitektur Source NAT (SNAT)

Source NAT digunakan untuk mentranslasikan alamat IP privat (RFC 1918) ke alamat IP publik agar dapat berkomunikasi di Internet.

### Mode SNAT Utama:
1. **Interface PAT (Port Address Translation / Overload):**
   * Mentranslasikan ribuan koneksi internal ke 1 IP publik interface WAN.
   * *Risiko:* Kehabisan port sumber (*Source Port Exhaustion*) jika lalu lintas simultan sangat padat (maksimal ~60.000 sesi TCP/UDP per IP publik).
2. **Dynamic IP Pool (SNAT Pool):**
   * Menggunakan blok subnet IP publik (misal /28 atau /29) untuk mendistribusikan beban translasi secara seimbang.
   * *Best Practice:* Pisahkan pool IP untuk segmen Pengguna Umum dengan pool IP untuk Server Datacenter agar reputasi IP server tidak tercemar oleh aktivitas browsing pengguna.
3. **Port Preservation:**
   * Jangan aktifkan *port-preservation* kecuali jika aplikasi spesifik mewajibkannya. Pengacakan port sumber (*Port Randomization*) membantu mencegah serangan *Blind TCP Spoofing*.

---

## 2. Keamanan Destination NAT (DNAT / Virtual IP / Port Forwarding)

Destination NAT digunakan untuk mempublikasikan layanan internal (seperti Web Server di DMZ) ke Internet publik.

### 🚫 Aturan Keras Keamanan DNAT:
1. **Haram Membuka Antarmuka Manajemen ke Publik:**
   * Jangan pernah membuat VIP yang memetakan port manajemen firewall (HTTPS 443 / SSH 22) ke Internet publik dengan source `ANY` atau `0.0.0.0/0`. Celah ini adalah penyebab nomor satu serangan eksploitasi zero-day pada peralatan perimeter.
2. **Terapkan Port Translation (Port Redirection):**
   * Jika layanan internal berjalan pada port standar, gunakan port eksternal non-standar bila memungkinkan untuk menghindari pemindai botnet otomatis (kecuali untuk portal web resmi publik).
3. **Batasi IP Sumber (Source Whitelisting):**
   * Untuk layanan administratif (seperti Controller Wi-Fi, Server Monitoring PRTG, SSH Server), batasi alamat IP sumber pada firewall policy hanya untuk IP publik kantor cabang atau rekanan resmi.
4. **Wajib Front-Ending dengan WAF / Reverse Proxy:**
   * Layanan web berbasis HTTP/HTTPS di DMZ harus melewati profil WAF (*Web Application Firewall*) untuk menyaring serangan SQL Injection, Cross-Site Scripting (XSS), dan proteksi bot.

---

## 3. Blackhole Routing untuk Menghindari Routing Loops

Jika firewall mengumumkan rute publik atau memiliki subnet VPN, pastikan ada **Blackhole Route** (atau rute Null0) dengan *Administrative Distance* tinggi (misal AD 254):
* Saat tunnel VPN turun (*down*), paket untuk subnet tujuan tidak akan berputar kembali (*routing loop*) ke default gateway Internet, melainkan langsung dibuang ke Blackhole.


---

# Modul 05: Next-Generation Security Profiles (UTM & SSL Inspection)

## 1. Mengapa Port-Based Firewalling Sudah Usang?

Firewall tradisional Layer 4 hanya melihat alamat IP dan nomor Port (TCP/UDP). Metode ini sudah tidak efektif karena:
* Lebih dari 90% lalu lintas web dan aplikasi saat ini dibungkus di dalam protokol enkripsi HTTPS (Port 443).
* Berbagai aplikasi modern (termasuk malware, C2 channels, torrent, dan tunneling tools) dapat berkamuflase menggunakan port 443 atau 80 untuk menembus filter tradisional.

---

## 2. 5 Pilar Next-Generation Inspection

Setiap kebijakan akses pengguna ke Internet (*Outbound Policy*) wajib menerapkan kombinasi profil keamanan berikut:

```
                  +----------------------------------------------+
                  |         INCOMING PACKET STREAM (L7)          |
                  +----------------------------------------------+
                                         |
                                         v
                  [1] DNS Filter (Cek reputasi domain sebelum IP)
                                         |
                                         v
                  [2] SSL/TLS Inspection (Dekripsi & Buka Payload)
                                         |
                                         v
                  [3] Antivirus Engine (Cek virus & Cloud Sandboxing)
                                         |
                                         v
                  [4] IPS Engine (Deteksi Eksploitasi & C2 Botnet)
                                         |
                                         v
                  [5] App-Control & Web Filter (Kategori & Lisensi)
                                         |
                                         v
                  +----------------------------------------------+
                  |      CLEAN PACKET FORWARDED TO DESTINATION   |
                  +----------------------------------------------+
```

1. **DNS Filtering:**
   * Menyaring permintaan domain berbahaya pada lapisan DNS sebelum koneksi IP terbentuk. Mencegah malware berkomunikasi dengan server DGA (*Domain Generation Algorithm*).
2. **Antivirus & Anti-Malware Engine:**
   * Menggunakan pendeteksian berbasis tanda tangan (*signature-based*) dan integrasi hash intelijen ancaman eksternal untuk memblokir malware seketika.
3. **Intrusion Prevention System (IPS):**
   * Menganalisis anomali paket, eksploitasi kerentanan perangkat lunak, serangan DoS flood, dan komunikasi botnet.
4. **Application Control (L7 App-ID):**
   * Mengidentifikasi aplikasi secara presisi tanpa memandang port yang digunakan (misal memblokir BitTorrent, Ultrasurf, atau membatasi transfer file pada aplikasi cloud storage).
5. **Web Filtering:**
   * Memblokir kategori situs tidak produktif atau berbahaya (Malware, Phishing, Proxy Avoidance, Judi, Pornografi).

---

## 3. Deep SSL/TLS Inspection (SSL Decryption)

Jika firewall tidak melakukan dekripsi SSL, firewall Anda buta (*blind*) terhadap isi paket di dalam port 443.

### Strategi Penerapan SSL Inspection:
* **Certificate Inspection (SNI Only):**
  * Firewall hanya membaca Server Name Indication (SNI) pada tahap handshake awal. Tidak membaca payload data. Ringan, tetapi malware yang terenkripsi tetap bisa lolos.
* **Full Deep SSL Inspection (Man-in-the-Middle):**
  * Firewall mendekripsi trafik, melakukan inspeksi antivirus/IPS, lalu mengenkripsi ulang sebelum dikirim ke klien.
  * *Syarat Wajib:* Sertifikat Root CA internal firewall harus didistribusikan ke seluruh workstation klien melalui Group Policy Object (GPO) atau MDM agar browser tidak menampilkan peringatan keamanan.
* **Daftar Pengecualian Wajib (Exclusion List):**
  * Demi privasi dan kepatuhan hukum, kecualikan kategori sensitif dari inspeksi dekripsi:
    * Perbankan & Layanan Finansial (*Finance & Banking*)
    * Layanan Medis & Kesehatan (*Health & Medicine*)
    * Layanan Update Resmi Sistem Operasi (Microsoft Update, Apple Update)


---

# Modul 06: High Availability (HA) & Ketahanan Operasional

## 1. Pemilihan Mode HA: Active-Passive vs Active-Active

| Parameter | Mode Active-Passive (Direkomendasikan) | Mode Active-Active |
| :--- | :--- | :--- |
| **Prinsip Kerja** | 1 unit menangani 100% trafik, unit kedua siaga penuh (*hot standby*). | Beban trafik dibagi antar kedua unit secara simultan. |
| **Deterministik** | **Tinggi.** Saat unit 1 gagal, performa tetap 100% sama di unit 2. | **Rendah.** Jika kapasitas kedua unit di atas 50%, saat 1 unit mati, unit tersisa akan mengalami *overload* (50%+ drop). |
| **Kompleksitas Routing**| Sangat sederhana, bebas dari risiko routing asimetris. | Kompleks, rawan pemisahan state sesi (*session desync*). |
| **Penggunaan Ideal** | Standar Datacenter Enterprise & Perbankan. | Skenario khusus dengan isolasi VDOM terpisah. |

---

## 2. Redundansi Kabel Heartbeat (HA Control Link)

Jalur Heartbeat adalah kabel komunikasi antar node kluster yang menyinkronkan tabel status sesi (*session state table*) dan sinyal detak jantung (*keepalive*):
* **Wajib Menggunakan Minimal 2 Link Fisik Terpisah:**
  * Jangan pernah hanya mengandalkan 1 kabel heartbeat. Jika kabel tersebut putus, kedua unit akan menganggap rekannya mati dan keduanya mempromosikan diri menjadi Master (**Split-Brain Scenario**), menyebabkan benturan IP (*IP conflict*) dan pemadaman jaringan total.
* **Koneksi Langsung (Direct Cable):**
  * Sambungkan kabel heartbeat langsung port-to-port antar unit tanpa melalui switch perantara.
* **Prioritaskan Antarmuka Dedicated:**
  * Gunakan port yang didedikasikan pabrikan untuk HA (misal port `ha1` dan `ha2`).

---

## 3. Pemantauan Antarmuka (Interface Monitoring / Link Failover)

Secara default, jika sebuah unit firewall tidak mengalami mati daya, kluster tidak akan melakukan failover meskipun kabel uplink Internet ke switch putus!
* **Solusi Wajib:** Konfigurasikan **Interface Monitoring (pmon)** pada seluruh port kritis:
  * Port Trunk Core Switch (Downlink)
  * Port Uplink ISP Utama (Uplink)
* Jika salah satu antarmuka yang dipantau putus, prioritas unit tersebut akan diturunkan secara otomatis (*priority penalty*), memicu failover instan ke unit standby yang link-nya masih sehat.


---

# Modul 07: Strategi Pencatatan Log, SIEM & Telemetri

## 1. Kebijakan Pencatatan Log (Logging Policy)

Pencatatan log adalah fondasi utama bagi visibilitas jaringan, audit kepatuhan, dan investigasi forensik insiden siber.

### Aturan Pencatatan:
1. **Rule Izinkan (Accept):**
   * Pilih **Log at Session Close (Session-End)**.
   * *Alasan:* Mencatat log di awal sesi (*Session Start*) tidak mencatat durasi sesi dan total volume data (byte sent / byte received), serta menghasilkan volume log ganda yang memboroskan memori.
2. **Rule Tolak (Deny / Drop):**
   * Aktifkan pencatatan log pada seluruh rule tolak, termasuk rule *Cleanup Deny-All*.
   * Sangat krusial untuk mendeteksi pemindaian port (*port scanning*), upaya eksploitasi, dan anomali jaringan internal.
3. **Pengecualian Log Bising (High-Frequency Noise):**
   * Untuk lalu lintas broadcast internal berfrekuensi tinggi (misal NetBIOS Name Service, MDNS, SSDP), buat rule drop eksplisit tanpa log (*deny without logging*) agar ruang penyimpanan log tidak dipenuhi artefak yang tidak berguna.

---

## 2. Integrasi SIEM Terpusat & Sinkronisasi Waktu

* **Penyimpanan Lokal Firewall Terbatas:**
  * Firewall appliance memiliki penyimpanan disk lokal yang terbatas. Log lokal dapat terhapus atau tertimpa (*overwrite*) dalam hitungan minggu atau hari saat terjadi lonjakan trafik.
* **Wajib Integrasi Syslog / SIEM Terpusat:**
  * Arahkan log secara real-time ke SIEM terpusat (misal FortiAnalyzer, Splunk, Elastic, Sentinel, atau Graylog) menggunakan protokol Syslog terenkripsi (TLS) atau protokol agen aman.
  * Terapkan kebijakan retensi log minimal **180 hari hingga 365 hari** sesuai standar kepatuhan regulasi.
* **Sinkronisasi Waktu Akurat (NTP):**
  * Pastikan firewall disinkronkan ke server NTP terpercaya (Stratum 1 atau 2).
  * Tanpa sinkronisasi waktu yang presisi, korelasi log forensik antara firewall, server web, dan database saat terjadi insiden keamanan siber menjadi mustahil dilakukan.


---

# Modul 08: Pengerasan Bidang Manajemen (Management Plane)

## 1. Pemisahan Jaringan Out-of-Band (OOB)

Bidang manajemen (*Management Plane*) adalah jalur kontrol yang digunakan administrator untuk mengonfigurasi perangkat.
* **Isolasi Fisik:** Hubungkan port manajemen firewall (misal port `mgmt` atau port1) ke jaringan terisolasi fisik yang terpisah dari jaringan data produksi.
* **Nonaktifkan Akses Manajemen pada Interface Publik:**
  * Pada interface WAN yang menghadap Internet, **HARAM** mengaktifkan opsi akses:
    * `HTTP` (Teks terbuka, kredensial dapat disadap)
    * `TELNET` (Teks terbuka)
    * `PING` (Kecuali jika dibutuhkan monitoring SLA ISP spesifik)
    * `SNMP` (Kecuali via VPN terenkripsi)
    * `HTTPS` & `SSH` (Kecuali dibatasi ketat dengan IP Whitelist)

---

## 2. Kebijakan Trusted Hosts (IP Whitelisting Administrator)

Setiap akun administrator yang dibuat pada firewall **WAJIB** dikonfigurasi dengan batasan **Trusted Hosts**:
* Hanya izinkan akses login GUI/CLI dari subnet manajemen terpercaya (misal subnet SOC atau jump server).
* Akun super-administrator tanpa batasan trusted host (terbuka untuk sembarang IP) adalah celah keamanan berperingkat **KRITIS** yang melanggar standar CIS Benchmark dan ISO 27001.

---

## 3. Autentikasi Kuat & Kontrol Akses Administratif

1. **Wajibkan Multi-Factor Authentication (MFA):**
   * Integrasikan autentikasi admin dengan identity provider enterprise (SAML SSO, RADIUS dengan token OTP, atau FIDO2 hardware token).
2. **Prinsip Least Privilege (RBAC):**
   * Pisahkan peran administrator:
     * *Super Admin:* Hak penuh untuk perubahan arsitektur utama.
     * *Network Operator:* Hanya memiliki hak mengubah rute dan interface.
     * *Security Auditor / Read-Only:* Hanya hak melihat konfigurasi dan log untuk tim audit.
3. **Session Timeout & Lockout Protection:**
   * Atur *Idle Timeout* antarmuka GUI/CLI maksimal **10 menit** (600 detik).
   * Aktifkan proteksi *Brute-Force Lockout* (misal blokir akun setelah 3 kali salah memasukkan password selama 15 menit).
4. **Pemberitahuan Login Banner:**
   * Tampilkan pesan peringatan hukum resmi pada halaman login awal untuk menegaskan bahwa akses tidak sah akan dituntut sesuai peraturan perundang-undangan.


---

# Modul 09: Checklist Audit Kepatuhan (50-Point Compliance Checklist)

Gunakan lembar kerja evaluasi mandiri ini untuk mengaudit postur keamanan firewall produksi Anda secara berkala.

| No | Parameter Pemeriksaan | Target Standar Kepatuhan | Status Evaluasi | Catatan Tindakan |
| :--- | :--- | :--- | :---: | :--- |
| **A** | **ARSITEKTUR & SEGMENTASI** | | | |
| 1 | Pemisahan Zona Keamanan | Terbagi minimal Untrust, DMZ, Server, User, OOB Mgmt | [ ] PASS / [ ] FAIL | |
| 2 | Ketiadaan Akses Langsung DMZ ke Internal | Server DMZ tidak dapat memulai sesi ke database internal | [ ] PASS / [ ] FAIL | |
| 3 | Isolasi Jaringan Tamu (Guest Wi-Fi) | Tidak ada rute atau izin akses dari Tamu ke segmen kantor | [ ] PASS / [ ] FAIL | |
| 4 | Segmentasi IoT & Kamera CCTV | Port SMB (445) dan NetBIOS diblokir keluar dari segmen IoT | [ ] PASS / [ ] FAIL | |
| 5 | Anti-Spoofing Filter | Memblokir paket IP martians & RFC1918 yang datang dari WAN | [ ] PASS / [ ] FAIL | |
| **B** | **KEBIJAKAN & TATA KELOLA ATURAN** | | | |
| 6 | Kebijakan Default Deny All | Aturan terakhir adalah Explicit Deny All dengan Log aktif | [ ] PASS / [ ] FAIL | |
| 7 | Ketiadaan Rule Any-to-Any | Tidak ada rule Accept dengan Source ALL, Dest ALL, Service ALL | [ ] PASS / [ ] FAIL | |
| 8 | Urutan Aturan (Rule Ordering) | Rule blokir scanner/botnet diletakkan di atas rule izin | [ ] PASS / [ ] FAIL | |
| 9 | Ketiadaan Rule Bayangan (Shadowed) | Tidak ada aturan spesifik yang terblokir oleh aturan luas | [ ] PASS / [ ] FAIL | |
| 10 | Standarisasi Penamaan Objek | Seluruh objek memiliki awalan standar (H_, NET_, GRP_) | [ ] PASS / [ ] FAIL | |
| 11 | Pembersihan Objek Tak Terpakai | Zero hit count objects dibersihkan setiap 90 hari | [ ] PASS / [ ] FAIL | |
| 12 | Deskripsi pada Setiap Kebijakan | Setiap rule mencantumkan keterangan PIC dan nomor tiket CR | [ ] PASS / [ ] FAIL | |
| **C** | **KRIPTOGRAFI & KEAMANAN VPN** | | | |
| 13 | Peniadaan Algoritma Usang | Tidak ada tunnel yang memakai MD5, SHA-1, DES, atau 3DES | [ ] PASS / [ ] FAIL | |
| 14 | Penggunaan Cipher Modern | Proposal menggunakan AES-GCM (128 atau 256) | [ ] PASS / [ ] FAIL | |
| 15 | Kekuatan Diffie-Hellman Group | Tidak menggunakan DH Group 1, 2, atau 5 (Wajib DH 14/19/20) | [ ] PASS / [ ] FAIL | |
| 16 | Perfect Forward Secrecy (PFS) | PFS diaktifkan pada seluruh konfigurasi Phase 2 | [ ] PASS / [ ] FAIL | |
| 17 | TCP MSS Clamping pada Tunnel | Dikonfigurasi pada nilai 1350/1360 untuk mencegah fragmentasi | [ ] PASS / [ ] FAIL | |
| 18 | Multi-Factor Authentication VPN | Seluruh remote access VPN wajib MFA (OTP / Hardware Token) | [ ] PASS / [ ] FAIL | |
| 19 | Dead Peer Detection (DPD) | DPD aktif untuk pemulihan cepat saat rute terputus | [ ] PASS / [ ] FAIL | |
| 20 | Keamanan Pre-Shared Key (PSK) | PSK minimal 32 karakter acak (kombinasi huruf, angka, simbol) | [ ] PASS / [ ] FAIL | |
| **D** | **NAT & PUBLIKASI LAYANAN (VIP)** | | | |
| 21 | Proteksi Port Manajemen | Port GUI/SSH tidak dipublikasikan ke publik via VIP (0.0.0.0/0) | [ ] PASS / [ ] FAIL | |
| 22 | IP Whitelist pada Inbound Admin | Akses administratif dibatasi hanya untuk IP statis SOC | [ ] PASS / [ ] FAIL | |
| 23 | Port Randomization pada SNAT | Port sumber diacak untuk mencegah blind TCP spoofing | [ ] PASS / [ ] FAIL | |
| 24 | Pemisahan Pool SNAT | Pool IP publik user terpisah dari pool IP server produksi | [ ] PASS / [ ] FAIL | |
| 25 | Blackhole Routing | Rute Blackhole (AD 254) aktif untuk subnet internal publik | [ ] PASS / [ ] FAIL | |
| **E** | **NEXT-GEN SECURITY INSPECTION (UTM)** | | | |
| 26 | Antivirus pada Akses Internet | Profil AV aktif pada seluruh policy outbound pengguna | [ ] PASS / [ ] FAIL | |
| 27 | IPS Sensor Terupdate | IPS aktif menyaring serangan botnet, exploit, dan scanning | [ ] PASS / [ ] FAIL | |
| 28 | Web Filtering Kategori Berbahaya | Memblokir kategori Malware, Phishing, Proxy, dan Judi | [ ] PASS / [ ] FAIL | |
| 29 | DNS Filtering | Memblokir domain botnet C2 pada lapisan resolusi DNS | [ ] PASS / [ ] FAIL | |
| 30 | Application Control | Memblokir aplikasi tunneling (Tor, Ultrasurf, P2P Torrent) | [ ] PASS / [ ] FAIL | |
| 31 | Penerapan SSL Inspection | Deep SSL aktif dengan pengecualian domain perbankan/medis | [ ] PASS / [ ] FAIL | |
| 32 | Otomasi Update Signatur | Database IPS, AV, dan WebFilter ter-update otomatis berkala | [ ] PASS / [ ] FAIL | |
| **F** | **PENCATATAN LOG & MONITORING** | | | |
| 33 | Pencatatan Log pada Rule Izin | Log diatur pada mode `Session-End / Session Close` | [ ] PASS / [ ] FAIL | |
| 34 | Pencatatan Log pada Rule Tolak | Rule deny mencatat paket yang dibuang untuk audit serangan | [ ] PASS / [ ] FAIL | |
| 35 | Integrasi SIEM Terpusat | Log dikirimkan ke server SIEM / Syslog terenkripsi | [ ] PASS / [ ] FAIL | |
| 36 | Retensi Log Memadai | Penyimpanan log dijamin minimal 180 hari | [ ] PASS / [ ] FAIL | |
| 37 | Sinkronisasi Waktu (NTP) | Menggunakan server NTP terpercaya (Stratum 1/2) | [ ] PASS / [ ] FAIL | |
| 38 | Peringatan Real-Time Aktif | Peringatan aktif untuk kegagalan login, link down, CPU spike | [ ] PASS / [ ] FAIL | |
| **G** | **HIGH AVAILABILITY (HA) & KETAHANAN** | | | |
| 39 | Konfigurasi Redundan (Dual Node) | Menggunakan kluster Active-Passive deterministik | [ ] PASS / [ ] FAIL | |
| 40 | Dual Link Heartbeat Fisik | Menggunakan minimal 2 kabel heartbeat langsung tanpa switch | [ ] PASS / [ ] FAIL | |
| 41 | Pemantauan Antarmuka (pmon) | Interface monitoring aktif pada uplink ISP dan link Core Switch | [ ] PASS / [ ] FAIL | |
| 42 | Sinkronisasi Sesi | Session sync aktif untuk menjamin zero-drop saat failover | [ ] PASS / [ ] FAIL | |
| **H** | **PENGURUSAN BIDANG MANAJEMEN** | | | |
| 43 | Isolasi Out-of-Band (OOB) | Akses admin hanya tersedia melalui port fisik manajemen | [ ] PASS / [ ] FAIL | |
| 44 | Trusted Hosts pada Semua Akun | Tidak ada akun admin yang memiliki izin akses dari sembarang IP | [ ] PASS / [ ] FAIL | |
| 45 | Penonaktifan Protokol Tidak Aman | HTTP, Telnet, SNMPv1/v2c dimatikan (hanya HTTPS, SSH, SNMPv3) | [ ] PASS / [ ] FAIL | |
| 46 | Multi-Factor Authentication Admin | Akses admin dilindungi token MFA / SAML SSO terpusat | [ ] PASS / [ ] FAIL | |
| 47 | Pengaturan Idle Timeout | Timeout sesi admin diatur maksimal 10 menit | [ ] PASS / [ ] FAIL | |
| 48 | Proteksi Brute-Force Lockout | Akun terkunci otomatis setelah 3-5 kali gagal login | [ ] PASS / [ ] FAIL | |
| 49 | Penyesuaian Login Banner | Menampilkan pesan peringatan hukum resmi | [ ] PASS / [ ] FAIL | |
| 50 | Backup Konfigurasi Otomatis | Backup otomatis terjadwal dan terenkripsi ke repositori aman | [ ] PASS / [ ] FAIL | |
