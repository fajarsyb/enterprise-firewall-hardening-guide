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
