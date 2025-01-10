# **Panduan Konfigurasi MikroTik untuk Jaringan Lokal dengan DHCP, NAT, Firewall, dan Manajemen Bandwidth Menggunakan AntiX Linux**

## **Soal 1: Konfigurasi IP Address dan DHCP Server**

### **Langkah 1.1: Pengaturan Adapter di VirtualBox**
- **MikroTik VM:**
  - **Adapter 1 (ether1):**  
    - Pilih *Bridged Adapter* supaya MikroTik dapat terhubung ke jaringan yang sama dengan laptop kamu (Wi-Fi atau Ethernet).
    - Fungsi: Menghubungkan MikroTik ke jaringan lokal untuk distribusi IP.
  
  - **Adapter 2 (ether2):**  
    - Pilih *NAT* atau *Bridged Adapter*, tergantung pengaturan internet kamu. Kalau pakai *NAT*, MikroTik akan berbagi koneksi internet dengan perangkat lain.
    - Fungsi: Menghubungkan MikroTik ke internet.

- **AntiX VM:**
  - **Adapter 1:**  
    - Pilih *Bridged Adapter* supaya AntiX dapat terhubung langsung ke jaringan lokal yang sama dengan MikroTik.
    - Fungsi: Agar AntiX bisa mendapatkan IP dari MikroTik.

### **Langkah 1.2: Konfigurasi di MikroTik**
1. **Login ke MikroTik** dan akses terminal.
   
2. **Atur IP untuk ether1 (local network):**  
   - IP **192.168.0.45** dipilih karena diambil dari dua angka terakhir NPM (2213020145). Jadi, alamat IP ini dipilih supaya mudah diingat.
   - Perintah:
     ```bash
     /ip address add address=192.168.0.45/24 interface=ether1
     ```
   
3. **Aktifkan ether1:**
   - Perintah:
     ```bash
     /interface enable ether1
     ```
   
4. **Buat Pool IP DHCP:**  
   - Misalnya, kita buat pool dari **192.168.0.50** sampai **192.168.0.59** untuk 10 perangkat.
   - Perintah:
     ```bash
     /ip pool add name=dhcp_pool ranges=192.168.0.50-192.168.0.59
     ```

5. **Buat DHCP Server:**  
   - Perintah:
     ```bash
     /ip dhcp-server add name=dhcp1 interface=ether1 address-pool=dhcp_pool disabled=no
     ```

6. **Atur Jaringan DHCP:**  
   - Perintah:
     ```bash
     /ip dhcp-server network add address=192.168.0.0/24 gateway=192.168.0.45 dns-server=8.8.8.8
     ```

### **Langkah 1.3: Pengujian DHCP**
1. **Di AntiX, lepas IP lama dan minta IP baru** dengan perintah:
   ```bash
   sudo dhclient -r eth0
   sudo dhclient eth0
   ```
   - Perintah `sudo dhclient -r eth0` digunakan untuk melepaskan IP yang lama.
   - Perintah `sudo dhclient eth0` digunakan untuk meminta IP baru dari MikroTik.

2. **Cek IP yang diterima** di AntiX:
   ```bash
   ip a
   ```
   Pastikan IP yang diterima berada di rentang **192.168.0.50 hingga 192.168.0.59**.

3. **Ping ke gateway MikroTik** untuk cek koneksi:
   ```bash
   ping 192.168.0.45
   ```
---

## **Soal 2: NAT dan Internet Sharing**

### **Langkah 2.1: Konfigurasi di MikroTik**
1. **Atur IP untuk ether2 (WAN):**  
   - Perintah:
     ```bash
     /ip address add address=192.168.1.2/24 interface=ether2
     ```
   
2. **Atur Gateway ISP:**  
   - Perintah:
     ```bash
     /ip route add gateway=192.168.1.1
     ```

3. **Tambahkan NAT Masquerade untuk akses internet:**  
   - Perintah:
     ```bash
     /ip firewall nat add chain=srcnat action=masquerade out-interface=ether2
     ```

### **Langkah 2.2: Pengujian NAT**
1. **Di MikroTik, ping internet:**
   ```bash
   ping 8.8.8.8
   ```
   
2. **Di AntiX, ping internet juga:**
   ```bash
   ping 8.8.8.8
   ```

---

## **Soal 3: Firewall Rules untuk Memblokir Situs Tertentu**

### **Langkah 3.1: Konfigurasi di MikroTik**
1. **Buat Layer7 Protocol untuk blokir `facebook.com`:**  
   - Perintah:
     ```bash
     /ip firewall layer7-protocol add name=block_facebook regexp="^.+(facebook.com).*\$"
     ```

2. **Buat Aturan Firewall untuk blokir akses ke `facebook.com`:**  
   - Perintah:
     ```bash
     /ip firewall filter add chain=forward protocol=tcp layer7-protocol=block_facebook action=drop
     ```

### **Langkah 3.2: Pengujian Firewall**
1. **Di AntiX, coba akses `facebook.com`:**  
   - Facebook akan terblokir.

2. **Gunakan terminal di AntiX:**
   ```bash
   curl facebook.com
   ```
   - Jika berhasil, akan ada pesan error karena koneksi gagal.

---

## **Soal 4: Bandwidth Management (Simple Queue)**

### **Langkah 4.1: Konfigurasi di MikroTik**
1. **Buat Simple Queue untuk IP klien (contoh: 192.168.0.50):**  
   - Perintah:
     ```bash
     /queue simple add name=client1 target=192.168.0.50/32 max-limit=2M/1M
     ```
   - **Alasan:** Kecepatan download dibatasi 2 Mbps dan upload 1 Mbps.

2. **Buat Simple Queue untuk semua klien (opsional):**  
   - Perintah:
     ```bash
     /queue simple add name=all_clients target=192.168.0.0/24 max-limit=2M/1M
     ```

### **Langkah 4.2: Pengujian Bandwidth**
1. **Di AntiX, install speedtest-cli:**
   Sebelum install, disarankan untuk update dulu:
   ```bash
   sudo apt update
   sudo apt install speedtest-cli
   speedtest-cli
   ```

   - Pastikan kecepatan download dan upload sesuai dengan yang sudah diatur di MikroTik.

---
