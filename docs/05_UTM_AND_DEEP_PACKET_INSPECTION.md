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
