# Enterprise Firewall Architecture & Hardening Guide

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Standards: CIS & NIST](https://img.shields.io/badge/Security_Standards-CIS_Benchmark_%7C_NIST_SP_800--41-green.svg)](#standards-alignment)
[![Architecture: Zero Trust](https://img.shields.io/badge/Architecture-Zero_Trust_Ready-orange.svg)](#architecture-philosophy)
[![Status: Production Baseline](https://img.shields.io/badge/Baseline-v1.0.0-purple.svg)](#)

> **Panduan Arsitektur, Desain Kebijakan, dan Pengerasan Keamanan (Hardening) Firewall Enterprise Berstandar Global.**
> Bersifat *vendor-agnostic* (dapat diterapkan di Fortinet, Palo Alto Networks, Cisco Secure Firewall, Check Point, Juniper SRX, maupun pfSense/OPNsense) dan dirancang sebagai acuan referensi tetap bagi Network Security Engineer, Security Operations Center (SOC), dan Network Architect.

---

## Daftar Isi

1. [Filosofi Desain & 10 Aturan Emas](#10-aturan-emas-firewall-enterprise)
2. [Modul Panduan Teknis (Chapters)](#modul-panduan-teknis-chapters)
   * [Modul 01: Arsitektur Jaringan & Segmentasi Zero-Trust](docs/01_ARCHITECTURE_AND_SEGMENTATION.md)
   * [Modul 02: Siklus Hidup & Urutan Aturan Kebijakan (Rule Ordering)](docs/02_POLICY_LIFECYCLE_AND_ORDERING.md)
   * [Modul 03: Standar Kriptografi VPN & IPsec Modern](docs/03_VPN_AND_CRYPTOGRAPHY_STANDARDS.md)
   * [Modul 04: Arsitektur NAT & Keamanan Virtual IP (VIP/DNAT)](docs/04_NAT_AND_VIP_SECURITY.md)
   * [Modul 05: Next-Generation Security Profiles (UTM & SSL Inspection)](docs/05_UTM_AND_DEEP_PACKET_INSPECTION.md)
   * [Modul 06: High Availability (HA) & Ketahanan Operasional](docs/06_HIGH_AVAILABILITY_AND_RESILIENCY.md)
   * [Modul 07: Strategi Pencatatan Log, SIEM & Telemetri](docs/07_LOGGING_MONITORING_AND_SIEM.md)
   * [Modul 08: Pengerasan Bidang Manajemen (Management Plane)](docs/08_MANAGEMENT_PLANE_HARDENING.md)
   * [Modul 09: Checklist Audit Kepatuhan (50-Point Compliance Checklist)](docs/09_FIREWALL_AUDIT_CHECKLIST.md)
   * [Modul 10: Glosarium & Terminologi Lengkap (Firewall, Cyber, Network, Cloud, VPN, Routing, Switching)](docs/10_COMPREHENSIVE_TERMINOLOGY_GUIDE.md)
3. [Dokumen Kompilasi Tunggal (Single-File Guide)](#dokumen-kompilasi-tunggal)
4. [Standar Kepatuhan Internasional](#standar-kepatuhan-internasional)
5. [Kontribusi](#kontribusi)

---

## 10 Aturan Emas Firewall Enterprise

Prinsip dasar yang wajib dipatuhi dalam setiap perancangan dan operasional firewall produksi:

```
[1] Default Deny All  -> Tolak semua lalu lintas secara eksplisit; izinkan hanya yang dibutuhkan.
[2] Strict Ordering   -> Evaluasi dari paling spesifik (Drop/Threat Feed) ke paling umum.
[3] No Any-to-Any     -> Haramkan rule 'ANY Source to ANY Destination with Service ALL'.
[4] Segment by Zone   -> Pisahkan Trust, Untrust, DMZ, Server Farm, Pegawai, Tamu, dan OOB Mgmt.
[5] Protect Mgmt      -> Jangan pernah mengekspos Web GUI/SSH ke Internet tanpa IP Whitelist & MFA.
[6] Modern Crypto     -> Gunakan AES-GCM & DH14+; buang MD5, 3DES, DES, dan DH Group 1/2/5.
[7] Log Accountability-> Aktifkan pencatatan log pada setiap rule izin dan arahkan ke SIEM terpusat.
[8] MSS Clamping      -> Pasang TCP MSS 1350/1360 pada tunnel IPsec untuk mencegah fragmentasi paket.
[9] Inspect Content   -> Port 443 tanpa SSL Inspection adalah celah buta (blind spot) bagi malware.
[10] Hygiene Routine  -> Audit bulanan: bersihkan rule bayangan (shadowed) dan objek tak terpakai.
```

---

## Modul Panduan Teknis (Chapters)

### 📘 [Modul 01: Arsitektur Jaringan & Segmentasi Zero-Trust](docs/01_ARCHITECTURE_AND_SEGMENTATION.md)
* Desain Security Zones (Untrust WAN, Edge DMZ, Server Farm, Campus LAN, Isolated IoT/CCTV, OOB Management).
* Prinsip Micro-segmentation & East-West traffic filtering antar server internal.
* Komparasi Hardware Acceleration (ASIC/NP/CP) vs Software CPU Processing.

### 📘 [Modul 02: Siklus Hidup & Urutan Aturan Kebijakan (Rule Ordering)](docs/02_POLICY_LIFECYCLE_AND_ORDERING.md)
* Hierarki penataan urutan policy dari baris teratas:
  1. *Anti-Spoofing & Invalid State Drop*
  2. *Threat Intelligence Feeds (Botnet, C2, Tor, Scanner IP Blocklist)*
  3. *Inbound Specific Services (VIP / Reverse Proxy)*
  4. *Outbound Authenticated User Traffic with Full UTM*
  5. *Inter-Zone Controlled East-West Flow*
  6. *Explicit Cleanup Deny-All with Logging*
* Strategi penamaan objek dan grup (Naming Conventions).
* Tata kelola pembersihan objek mati (Dormant Object Hygiene).

### 📘 [Modul 03: Standar Kriptografi VPN & IPsec Modern](docs/03_VPN_AND_CRYPTOGRAPHY_STANDARDS.md)
* Mengapa **AES-GCM (Single-Pass AEAD)** jauh lebih cepat dan aman daripada **AES-CBC + HMAC (Double-Pass)**.
* Standar baseline Phase 1: IKEv2, AES-256-GCM / PRF-SHA384, DH Group 19 (Curve25519) / 20 / 14.
* Standar baseline Phase 2: AES-GCM dengan Perfect Forward Secrecy (PFS) aktif.
* Panduan pencegahan fragmentasi paket melalui **TCP MSS Clamping (1350/1360 byte)**.
* Standar Remote Access VPN (MFA wajib, Device Posture Check, pembatasan split-tunneling).

### 📘 [Modul 04: Arsitektur NAT & Keamanan Virtual IP (VIP/DNAT)](docs/04_NAT_AND_VIP_SECURITY.md)
* Source NAT (SNAT): Dynamic Pool vs Overload PAT; mitigasi port exhaustion.
* Destination NAT (DNAT/VIP): Larangan mempublikasikan port manajemen ke publik (`0.0.0.0/0`).
* Best practice port translation, reverse proxy, dan WAF front-ending.
* Blackhole routing untuk mencegah kebocoran paket (*routing leaks*).

### 📘 [Modul 05: Next-Generation Security Profiles (UTM & SSL Inspection)](docs/05_UTM_AND_DEEP_PACKET_INSPECTION.md)
* 5 Lapisan inspeksi konten: Antivirus, IPS (Botnet/Exploits), Application Control, Web Filtering, DNS Filtering.
* Deep SSL/TLS Inspection: Strategi distribusi Certificate Authority (CA) internal dan pengecualian (Whitelisting Finansial/Kesehatan).
* Pengendalian aplikasi Layer 7 vs port Layer 4 tradisional.

### 📘 [Modul 06: High Availability (HA) & Ketahanan Operasional](docs/06_HIGH_AVAILABILITY_AND_RESILIENCY.md)
* Active-Passive vs Active-Active: Kapan harus memilih failover deterministik.
* Redundansi kabel Heartbeat (Dual physical direct-attach connections).
* Sinkronisasi sesi (*Session Synchronization*) dan mitigasi *Asymmetric Routing*.
* Pemantauan link (*Interface Monitoring*) dan gateway health checks.

### 📘 [Modul 07: Strategi Pencatatan Log, SIEM & Telemetri](docs/07_LOGGING_MONITORING_AND_SIEM.md)
* Kebijakan pencatatan log: Kapan mencatat *Log at Session Start* vs *Log at Session Close*.
* Integrasi Syslog terpusat, SIEM, dan Security Analytics (FortiAnalyzer, Splunk, Elastic, Sentinel).
* Sinkronisasi waktu akurat melalui Network Time Protocol (NTP Stratum 1/2).
* Ambang batas peringatan dini (*Real-Time Alerting*) untuk deteksi serangan brute-force dan pemindaian port.

### 📘 [Modul 08: Pengerasan Bidang Manajemen (Management Plane)](docs/08_MANAGEMENT_PLANE_HARDENING.md)
* Pemisahan jalur Out-of-Band (OOB) fisik terisolasi dari jalur data.
* Penerapan **Trusted Hosts / Admin IP Whitelist** mutlak bagi seluruh akun privilege.
* Peniadaan protokol teks terbuka (Matikan HTTP, Telnet, SNMPv1/v2c; wajibkan HTTPS, SSH, SNMPv3).
* Integrasi Multi-Factor Authentication (MFA), SAML SSO enterprise, dan audit rekaman sesi admin.

### 📘 [Modul 09: Checklist Audit Kepatuhan (50-Point Checklist)](docs/09_FIREWALL_AUDIT_CHECKLIST.md)
* Lembar kerja praktis berisi 50 poin evaluasi siap pakai untuk audit berkala tim keamanan internal maupun persiapan sertifikasi kepatuhan ISO 27001.

### 📘 [Modul 10: Glosarium & Terminologi Lengkap](docs/10_COMPREHENSIVE_TERMINOLOGY_GUIDE.md)
* Kamus istilah komprehensif 7 domain: **Firewall, Cybersecurity, Networking, Cloud, Tunneling, Routing, dan Switching**.
* Setiap istilah dilengkapi dengan **Definisi Teknis**, **Analogi Sederhana**, dan **Contoh Kasus Riil**.

---

## Dokumen Kompilasi Tunggal

Untuk kemudahan cetak atau pembacaan offline dalam satu file utuh:
* 📄 **[ENTERPRISE_FIREWALL_GOLDEN_BASELINE.md](ENTERPRISE_FIREWALL_GOLDEN_BASELINE.md)** (Seluruh bab digabungkan ke dalam 1 dokumen komprehensif).

---

## Standar Kepatuhan Internasional

Panduan ini diselaraskan dengan kerangka kerja keamanan siber global:
* **CIS Controls v8:** Safeguard 4.4 (Firewall Architecture), Safeguard 4.5 (Automated Rule Review).
* **NIST Special Publication 800-41 Rev. 1:** *Guidelines on Firewalls and Firewall Policy*.
* **PCI-DSS v4.0:** Requirement 1 (*Install and Maintain Network Security Controls*).
* **ISO/IEC 27001:2022:** Control A.8.20 (*Network Security*), Control A.8.22 (*Segregation in Networks*).

---

## Lisensi & Kontribusi

Dokumen ini didistribusikan di bawah lisensi **[MIT License](LICENSE)**. Anda bebas menggunakan, memodifikasi, dan menyebarkannya untuk kebutuhan institusi, perusahaan, maupun pembelajaran pribadi.
