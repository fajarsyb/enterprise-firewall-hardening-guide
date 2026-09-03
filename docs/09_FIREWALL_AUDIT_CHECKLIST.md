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
