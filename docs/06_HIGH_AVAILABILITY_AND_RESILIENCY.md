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
