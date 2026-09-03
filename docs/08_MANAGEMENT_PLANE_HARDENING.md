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
