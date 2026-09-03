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
