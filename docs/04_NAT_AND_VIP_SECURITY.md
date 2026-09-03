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
