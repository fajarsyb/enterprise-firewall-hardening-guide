# Modul 11: Desain Arsitektur Jaringan Ideal & Aman: 3-Layer Hierarchical vs 2-Layer Spine-Leaf Clos Fabric

*Panduan Arsitektur & Standar Rekayasa Jaringan Enterprise untuk Campus Network dan Modern Datacenter.*

---

## DAFTAR ISI

1. [Eksekutif Summary: Evolusi Pola Trafik Jaringan](#1-eksekutif-summary-evolusi-pola-trafik-jaringan)
2. [Model 1: Arsitektur 3-Layer Hierarchical (Campus & Enterprise LAN)](#2-model-1-arsitektur-3-layer-hierarchical-campus--enterprise-lan)
   * 2.1 [Struktur Tiga Tingkat (Core, Distribution, Access)](#21-struktur-tiga-tingkat-core-distribution-access)
   * 2.2 [Batas Layer 2 / Layer 3 (L2/L3 Boundary)](#22-batas-layer-2--layer-3-l2l3-boundary)
   * 2.3 [Penempatan Perimeter Firewall & Internal Segmentation (ISFW)](#23-penempatan-perimeter-firewall--internal-segmentation-isfw)
   * 2.4 [Keamanan Lapisan Akses (Access Layer Hardening)](#24-keamanan-lapisan-akses-access-layer-hardening)
3. [Model 2: Arsitektur 2-Layer Spine-Leaf Clos Fabric (Modern Datacenter)](#3-model-2-arsitektur-2-layer-spine-leaf-clos-fabric-modern-datacenter)
   * 3.1 [Filosofi Desain Jaringan Clos Fabric](#31-filosofi-desain-jaringan-clos-fabric)
   * 3.2 [Underlay Routing (L3 ECMP) & Eliminasi Spanning Tree (STP)](#32-underlay-routing-l3-ecmp--eliminasi-spanning-tree-stp)
   * 3.3 [Overlay EVPN-VXLAN & Distributed Anycast Gateway](#33-overlay-evpn-vxlan--distributed-anycast-gateway)
   * 3.4 [Integrasi Firewall & Layanan Keamanan (Service Leaf Model)](#34-integrasi-firewall--layanan-keamanan-service-leaf-model)
   * 3.5 [Microsegmentation & Jaringan Storage Lossless (RoCEv2)](#35-microsegmentation--jaringan-storage-lossless-rocev2)
4. [Matriks Perbandingan Mendalam: 3-Layer vs Spine-Leaf](#4-matriks-perbandingan-mendalam-3-layer-vs-spine-leaf)
5. [Panduan Keputusan: Kapan Memilih 3-Layer vs Spine-Leaf](#5-panduan-keputusan-kapan-memilih-3-layer-vs-spine-leaf)

---

## 1. EKSEKUTIF SUMMARY: EVOLUSI POLA TRAFIK JARINGAN

Dua arsitektur jaringan yang paling dominan di dunia enterprise saat ini dirancang untuk menyelesaikan dua permasalahan fundamental yang sangat berbeda:

```
+-------------------------------------------------------------------------------------------------------+
|                                    PERBEDAAN KARAKTERISTIK POLA TRAFIK                                |
+-------------------------------------------------------------------+-----------------------------------+
| MODEL 1: 3-LAYER HIERARCHICAL (CAMPUS)                            | MODEL 2: 2-LAYER SPINE-LEAF (DC)  |
+-------------------------------------------------------------------+-----------------------------------+
| • Pola Dominan: NORTH - SOUTH (~80%)                              | • Pola Dominan: EAST - WEST (~80%)|
| • Trafik mengalir dari Pengguna (Access) ke Internet/Pusat        | • Trafik mengalir antar server, container & DB    |
| • Batas L2/L3 terpusat di Distribution Switch                     | • Fabric L3 murni dengan Overlay EVPN-VXLAN       |
| • Redundansi mengandalkan MC-LAG / Virtual Chassis                | • Redundansi mengandalkan ECMP (Equal-Cost Path)  |
| • Sangat ideal untuk: Gedung Perkantoran, Kampus, Cabang          | • Sangat ideal untuk: Private Cloud, VM Farm, K8s |
+-------------------------------------------------------------------+-----------------------------------+
```

Pergeseran dari aplikasi monolitik tradisional ke arsitektur *microservices*, virtualisasi server padat, dan komputasi awan telah mengubah arus data di dalam datacenter: server tidak lagi hanya melayani permintaan pengguna luar (North-South), melainkan ribuan kali saling berkomunikasi dengan server database, storage cluster, dan API internal (East-West).

---

## 2. MODEL 1: ARSITEKTUR 3-LAYER HIERARCHICAL (CAMPUS & ENTERPRISE LAN)

Arsitektur hierarki tiga lapis (Cisco Classic Hierarchical Model) adalah standar emas yang telah teruji selama puluhan tahun untuk jaringan kampus, gedung perkantoran, dan jaringan enterprise umum.

```
                      +---------------------------------------+
                      |         PUBLIC INTERNET / WAN         |
                      +---------------------------------------+
                                          |
                                          v
                      +---------------------------------------+
                      |   PERIMETER NGFW (ACTIVE-PASSIVE)     |
                      |   - NAT, Threat Feed, Deep SSL, IPS   |
                      +---------------------------------------+
                                          |
                                          | (100G Trunk Uplink)
                                          v
+-----------------------------------------------------------------------------------+
| CORE LAYER (Spines Tulang Punggung):                                              |
| - Dual High-Speed Switch (MC-LAG / Virtual Chassis)                               |
| - Murni High-Speed Packet Transport (Wire-Speed Switching) Tanpa ACL              |
+-----------------------------------------------------------------------------------+
                  |                                               |
                  | (Dual 40G/100G MC-LAG)                        | (Dual 40G/100G MC-LAG)
                  v                                               v
+---------------------------------------+       +---------------------------------------+
| DISTRIBUTION LAYER (Pod Barat):       |       | DISTRIBUTION LAYER (Datacenter Pod):  |
| - L3 SVI Default Gateway: VLAN 10     |       | - Internal Segmentation Firewall(ISFW)|
| - Inter-VLAN Routing & Policy ACL     |       | - SVI Default Gateway: VLAN 100       |
| - DHCP Snooping & Dynamic ARP Inspect |       | - Microsegmentation (Block SMB lateral|
+---------------------------------------+       +---------------------------------------+
                  |                                               |
                  | (Dual 10G/25G LACP)                           | (Dual 25G/40G LACP)
                  v                                               v
+---------------------------------------+       +---------------------------------------+
| ACCESS LAYER (Campus Access Stacks):  |       | ACCESS LAYER (Datacenter ToR Stacks): |
| - 48-Port PoE+ Gigabit Access Switches|       | - Dual 10G/25G Top-of-Rack Switches   |
| - 802.1X Port Security, BPDU Guard    |       | - Redundant Server Teaming / Bonding  |
| - PC Pegawai, Telepon IP, Ruckus APs  |       | - Production Hypervisors & DB Servers |
+---------------------------------------+       +---------------------------------------+
```

### 2.1 Struktur Tiga Tingkat (Core, Distribution, Access)

1. **Core Layer (Tulang Punggung Berkecepatan Tinggi):**
   * *Fungsi Utama:* Menghubungkan seluruh modul distribusi dengan kecepatan kawat (*wire-speed*), latensi rendah, dan ketersediaan tinggi (99.999%).
   * *Aturan Keras Desain:* **JANGAN PERNAH menaruh Access Control List (ACL) yang rumit, filter paket mendalam, atau koneksi perangkat pengguna langsung pada Core Switch.** Core switch harus murni bertindak sebagai jalan tol bebas hambatan.
   * *Redundansi:* Sepasang Core Switch yang digabungkan menggunakan teknologi *Multi-Chassis Link Aggregation (MC-LAG)* atau *Virtual Chassis* untuk menjamin nol kegagalan titik tunggal (*no Single Point of Failure - SPOF*).

2. **Distribution Layer (Agregasi & Penegakan Kebijakan):**
   * *Fungsi Utama:* Menjadi titik temu agregasi dari puluhan switch akses di setiap lantai atau gedung.
   * *Peran Kritis:* Menjadi **L3 Boundary** tempat beradanya *Switched Virtual Interface (SVI)* yang berfungsi sebagai *Default Gateway* bagi pengguna.
   * *Penegakan Keamanan:* Menjalankan perutean antar-VLAN (*Inter-VLAN Routing*), QoS marking, mitigasi badai broadcast (*Storm Control*), dan penyaringan akses antar-departemen.

3. **Access Layer (Lapisan Akses Pengguna & Perangkat Tepi):**
   * *Fungsi Utama:* Menyediakan port konektivitas fisik bagi perangkat pengguna akhir: Workstation PC, laptop, telepon VoIP, kamera CCTV, dan Access Point Wi-Fi 6.
   * *Fitur Wajib:* Power over Ethernet (PoE+ / PoE++ 802.3bt), penumpukan switch (*Stacking Virtual Chassis*), dan pengendalian port fisik.

---

### 2.2 Batas Layer 2 / Layer 3 (L2/L3 Boundary)

Dalam arsitektur 3-Layer modern yang aman, **Batas L2/L3 WAJIB diletakkan di Distribution Layer, BUKAN di Core Layer dan BUKAN di Access Layer**:
* **Mengapa bukan di Core?** Jika VLAN dibiarkan membentang hingga ke Core, wilayah siaran broadcast (*Broadcast Domain*) menjadi terlalu luas. Kegagalan *loop* Spanning Tree di satu gedung dapat melumpuhkan seluruh jaringan kampus.
* **Mengapa di Distribution?** Setiap gedung/lantai memiliki segmen VLAN lokal yang terisolasi. Lalu lintas broadcast terhenti di Distribution Switch. Rute dari Distribution menuju Core Switch murni menggunakan routing Layer 3 (OSPF / IS-IS) yang stabil dan konvergen cepat.

---

### 2.3 Penempatan Perimeter Firewall & Internal Segmentation (ISFW)

* **Perimeter NGFW:**
  * Ditempatkan di puncak jaringan, menghubungkan Core Switch dengan Border Router ISP.
  * Berfungsi sebagai gerbang pengamanan North-South: menjalankan SNAT Pool, mitigasi serangan DDoS, inspeksi reputasi IP botnet, dan dekripsi Deep SSL untuk seluruh trafik Internet keluar masuk.
* **Demilitarized Zone (DMZ):**
  * Terhubung langsung ke antarmuka terdedikasi pada Perimeter Firewall.
  * Terisolasi dari LAN internal: server DMZ menerapkan kebijakan *Zero Trust* di mana server DMZ dilarang menginisiasi koneksi ke jaringan internal.
* **Internal Segmentation Firewall (ISFW):**
  * Ditempatkan di sisi Distribution Pod Datacenter.
  * Berfungsi memeriksa lalu lintas internal: memisahkan segmen PC Pegawai dari server database produksi, serta memblokir protokol rentan seperti SMB (TCP 445) dan RDP (TCP 3389) agar ransomware tidak dapat menyebar lateral.

---

### 2.4 Keamanan Lapisan Akses (Access Layer Hardening)

Switch akses langsung bersentuhan dengan pengguna, sehingga wajib dibekali 4 kendali keamanan perangkat tepi:
1. **IEEE 802.1X Network Access Control (NAC):** Port switch terkunci secara default. Perangkat yang dicolok wajib diautentikasi identitasnya melalui server RADIUS/Active Directory sebelum diberikan hak akses ke VLAN resmi.
2. **DHCP Snooping:** Mencegah serangan *Rogue DHCP Server* dengan menetapkan hanya port uplink menuju distribution switch yang berstatus *Trusted*.
3. **Dynamic ARP Inspection (DAI):** Mencegah serangan penyadapan *Man-in-the-Middle* berbasis *ARP Spoofing/Poisoning* dengan memvalidasi paket ARP terhadap tabel database DHCP Snooping.
4. **BPDU Guard & Root Guard:** Mencegah pengguna menyambungkan switch liar di bawah meja yang dapat merusak topologi Spanning Tree.

---

## 3. MODEL 2: ARSITEKTUR 2-LAYER SPINE-LEAF CLOS FABRIC (MODERN DATACENTER)

Arsitektur Spine-Leaf (diciptakan oleh Charles Clos pada teori jaringan telepon) adalah arsitektur baku mutlak untuk pusat data modern, private cloud (OpenStack, VMware Cloud Foundation), dan cluster kontainer berskala besar (Kubernetes).

```
                      +---------------------------------------+
                      |         WAN / MULTI-CLOUD EDGE        |
                      +---------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
| NORTH-SOUTH BORDER & SERVICES LEAF:                                               |
| - Dual Border Leafs (eBGP WAN & Cloud Direct Connect)                             |
| - Security Services Cluster (Cluster NGFW + WAF + ADC Load Balancer)              |
| - Service Chaining Inter-VRF Transit (VRF-Web <-> Firewall <-> VRF-DB)            |
+-----------------------------------------------------------------------------------+
         |                                                                   |
         | (100G Uplink)                                                     | (100G Uplink)
         v                                                                   v
+-----------------------------------------------------------------------------------+
| SPINE LAYER (L3 Pure Routing Underlay Fabric):                                    |
| - 4x Spine Switches (Non-Blocking High Density 100G/400G QSFP-DD)                 |
| - BGP EVPN Route Reflector | Equal-Cost Multi-Path (ECMP) | ZERO SPANNING TREE    |
+-----------------------------------------------------------------------------------+
   |        |        |        |        |        |        |        |        |        |
   | (Setiap Leaf Switch TERHUBUNG PENUH ke SEMUA Spine Switch - Full-Mesh Fabric)  |
   v        v        v        v        v        v        v        v        v        v
+-----------------------------------------------------------------------------------+
| LEAF LAYER (Top-of-Rack Switches / VTEP Distributed Anycast Gateways):            |
| - Leaf 01/02 (Web Pod)   - Leaf 03/04 (App Pod)   - Leaf 05/06 (Database Pod)    |
| - Leaf 07/08 (Lossless NVMe Storage Fabric / RoCEv2 RDMA over Converged Ethernet) |
+-----------------------------------------------------------------------------------+
         |                        |                        |                |
         v                        v                        v                v
+-----------------+      +-----------------+      +-----------------+  +------------+
| Kubernetes Pods |      | Hypervisor ESXi |      | Oracle RAC / DB |  | All-Flash  |
| Web Containers  |      | Application VMs |      | Postgres Clust. |  | SAN NVMe   |
+-----------------+      +-----------------+      +-----------------+  +------------+
```

### 3.1 Filosofi Desain Jaringan Clos Fabric

Dua hukum arsitektural mutlak pada Spine-Leaf Fabric:
1. **Setiap Leaf switch terhubung ke SETIAP Spine switch.**
2. **TIDAK PERNAH ada koneksi antar sesama Spine switch, dan TIDAK ADA koneksi antar sesama Leaf switch (kecuali link MC-LAG lokal jika tanpa EVPN).**

**Keunggulan Utama:**
* **Jarak Komunikasi Selalu 1-Hop (Predictable Latency):** Untuk berkomunikasi dari server di Rack A ke server di Rack Z, paket data selalu melintasi tepat 3 lompatan: `Leaf A -> Spine -> Leaf Z`. Latensi dapat diprediksi secara matematis dan bernilai sub-mikrodetik (<800 nanodetik).
* **Skalabilitas Horizontal Bebas Hambatan (Scale-Out):** Jika bandwidth kurang, cukup tambahkan 1 switch Spine baru dan hubungkan ke semua Leaf. Kapasitas fabric bertambah secara linier tanpa mengganggu konfigurasi yang sudah berjalan.

---

### 3.2 Underlay Routing (L3 ECMP) & Eliminasi Spanning Tree (STP)

Dalam arsitektur 3-layer lama, protokol *Spanning Tree Protocol (STP)* mematikan 50% jalur kabel cadangan untuk mencegah looping. Ini adalah pemborosan investasi perangkat keras yang sangat besar.
* **Underlay L3 Murni:** Di Spine-Leaf, seluruh link antar Spine dan Leaf adalah link Layer 3 yang dirutekan menggunakan protokol **eBGP** atau **OSPF**.
* **Equal-Cost Multi-Pathing (ECMP):** Jika terdapat 4 switch Spine 100G, maka setiap Leaf memiliki bandwidth agregat **400 Gbps yang 100% aktif secara bersamaan**. Beban paket data didistribusikan secara merata menggunakan algoritma *hash L4 (IP Asal, IP Tujuan, Port Asal, Port Tujuan)*. Tidak ada kabel yang diblokir oleh Spanning Tree.

---

### 3.3 Overlay EVPN-VXLAN & Distributed Anycast Gateway

Meskipun jaringan fisik di bawahnya (*underlay*) adalah Layer 3 murni, datacenter modern tetap membutuhkan kemampuan Layer 2 (misal untuk fitur live migration VM / vMotion antar-rak tanpa mengubah alamat IP). Solusinya adalah **EVPN-VXLAN**:
1. **VXLAN Encapsulation:** Frame Layer 2 Ethernet dibungkus ke dalam paket UDP Layer 3 (Port 4789). Jaringan L2 dibentangkan (*stretched*) di atas fabric L3.
2. **BGP EVPN Control Plane:** Bertindak sebagai sistem kontrol cerdas yang mendistribusikan informasi alamat MAC dan IP antar-Leaf tanpa memerlukan banjir siaran (*broadcast flood-and-learn*).
3. **Distributed Anycast Gateway:** Setiap switch Leaf memiliki alamat IP gateway dan alamat MAC virtual yang **persis sama** untuk setiap subnet VLAN. Default gateway server selalu berada tepat di switch Top-of-Rack di atas kepalanya. Perutean inter-VLAN terjadi lokal di rak tersebut tanpa perlu bolak-balik naik ke core switch (*Local Routing Optimization*).

---

### 3.4 Integrasi Firewall & Layanan Keamanan (Service Leaf Model)

Bagaimana cara mengamankan datacenter Spine-Leaf tanpa merusak performa latensi tinggi?
Jawabannya adalah **Border Leaf / Service Leaf Architecture**:
* **Dedicated Service Leaf Pod:** Sepasang switch Leaf khusus didedikasikan untuk menampung kluster peralatan keamanan: Cluster Next-Generation Firewall (NGFW), Web Application Firewall (WAF), dan Load Balancer (ADC).
* **Pemisahan Tenant via VRF (Virtual Routing and Forwarding):** Jaringan dibagi menjadi beberapa tabel rute terisolasi: `VRF-Web`, `VRF-App`, dan `VRF-Database`.
* **Service Chaining (Pembelokan Trafik Terkendali):**
  * Komunikasi antar-server di dalam satu tier yang sama (`Web ke Web`) diproses lokal di Leaf pada kecepatan 100 Gbps.
  * Komunikasi lintas tier (`Web ke Database`) secara otomatis diarahkan (*leaked*) melalui BGP EVPN menuju cluster Firewall di Service Leaf untuk diinspeksi secara mendalam sebelum diteruskan ke database tujuan.

---

### 3.5 Microsegmentation & Jaringan Storage Lossless (RoCEv2)

* **Microsegmentation di Level Host:**
  * Di samping firewall perimeter di Service Leaf, pengamanan antar-workload di rak yang sama diperkuat menggunakan *Distributed Virtual Firewall* (seperti Cilium eBPF pada Kubernetes atau VMware NSX pada hypervisor). Jika satu container web diretas, ia tidak bisa menyerang container web di sebelahnya meskipun berada di subnet yang sama.
* **Storage Lossless via RoCEv2 (RDMA over Converged Ethernet):**
  * Datacenter modern menggunakan protokol RoCEv2 untuk menghubungkan server komputasi dengan array penyimpanan NVMe All-Flash.
  * *Fitur Wajib Fabric:* Mengaktifkan **Priority Flow Control (PFC - 802.1Qbb)** dan **Explicit Congestion Notification (ECN - RFC 3168)** pada switch Spine-Leaf guna menjamin nol kehilangan paket (*Zero Packet Drop*) untuk lalu lintas storage berlatensi ultra-rendah.

---

## 4. MATRIKS PERBANDINGAN MENDALAM: 3-LAYER VS SPINE-LEAF

| Dimensi Parameter | Arsitektur 3-Layer Hierarchical | Arsitektur 2-Layer Spine-Leaf Clos Fabric |
| :--- | :--- | :--- |
| **Konteks Penggunaan Ideal** | **Campus Network**, Gedung Kantor, Jaringan Kampus Universitas, Cabang Enterprise. | **Modern Datacenter**, Cloud Infrastructure, High-Density Virtualization, AI/ML Clusters. |
| **Dominasi Aliran Trafik** | **North - South** (80% trafik menuju Internet / Wan Gateway). | **East - West** (80% trafik berkomunikasi antar server / VM / database lokal). |
| **Protokol Pencegahan Loop** | Spanning Tree Protocol (STP / RSTP / MSTP) dengan MC-LAG. | **Zero STP**. Pure L3 Dynamic Routing (eBGP) dengan ECMP Multipathing. |
| **Efisiensi Pemanfaatan Link** | **50% - 75%**. Beberapa link diblokir STP atau siaga pasif. | **100% Aktif**. Seluruh link Spine-Leaf membawa trafik simultan via ECMP. |
| **Karakteristik Latensi** | Bervariasi (2 - 5 hop tergantung letak switch distribution/core). | **Konsisten & Deterministik**. Selalu berjarak tepat **1-Hop** antar Leaf mana pun. |
| **Batas Default Gateway** | Terpusat di Distribution Switch (SVI / VRRP). | Terdistribusi di setiap Leaf Switch (**Distributed Anycast Gateway**). |
| **Teknologi Segmentasi** | VLAN IEEE 802.1Q Tradisional (Maksimal 4.094 VLAN). | **Overlay EVPN-VXLAN** (Mendukung hingga 16 Juta Network VNI). |
| **Metode Penambahan Kapasitas** | *Scale-Up* (Mengganti sasis switch Core dengan modul lebih besar). | *Scale-Out* (Cukup menambahkan 1 switch Spine/Leaf baru secara horizontal). |
| **Tingkat Kompleksitas Desain** | Menengah. Sangat umum dipahami oleh seluruh network engineer. | Tinggi. Membutuhkan keahlian otomasi BGP EVPN, underlay/overlay, dan telemetry. |

---

## 5. PANDUAN KEPUTUSAN: KAPAN MEMILIH 3-LAYER VS SPINE-LEAF

Gunakan diagram pohon keputusan berikut saat merancang jaringan baru:

```
[ IDENTIFIKASI KEBUTUHAN UTAMA JARINGAN ]
                   |
                   v
Apakah jaringan diperuntukkan bagi Gedung Kantor / Pengguna Manusia / PC Pegawai?
                   |
     +-------------+-------------+
     | YA                        | TIDAK (Untuk Pusat Data / Server Farm)
     v                           v
Gunakan MODEL 1:            Apakah terdapat ribuan VM, microservices Kubernetes, atau storage NVMe-oF?
3-LAYER HIERARCHICAL                     |
(Core - Dist - Access)       +-----------+-----------+
                             | YA                    | TIDAK (Hanya 1-2 Rak Server Kecil)
                             v                       v
                        Gunakan MODEL 2:        Gunakan MODEL 1 Sederhana:
                        2-LAYER SPINE-LEAF      Collapsed Core / Two-Tier Campus
                        (EVPN-VXLAN Clos)       (Core/Dist Gabung + Access)
```

### Kesimpulan Rekayasa:
* **Gunakan 3-Layer Hierarchical** jika fokus utama Anda adalah menghubungkan ribuan perangkat pengguna (*end-user devices*), menyediakan daya listrik PoE untuk telepon/AP Wi-Fi, mengontrol kebijakan akses berbasis departemen, dan sebagian besar trafik mengalir keluar menuju Internet.
* **Wajib Menggunakan 2-Layer Spine-Leaf** jika Anda sedang membangun lingkungan *Core Datacenter* modern di mana server-server saling bertukar data dalam volume masif (East-West), membutuhkan replikasi basis data berkecepatan tinggi tanpa hambatan Spanning Tree, serta menuntut skalabilitas komputasi awan yang modular dan tangguh.
