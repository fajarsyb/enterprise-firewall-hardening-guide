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
