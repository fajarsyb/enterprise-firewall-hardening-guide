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
