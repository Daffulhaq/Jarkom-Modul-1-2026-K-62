# Jarkom-Modul-1-2026-K-62

## =========== SEMENTARA =============

### 1. 
prefix: 192.242.xx
netmask subnet: /24 (255.255.255.0)

Sehingga:
switch1: 192.242.1.0/24
switch2: ........2.....
switch3: ........3.....

Gateway menjadi:
eth1: 192.242.1.1
eth2: ........2..
eth3: ........3..

aturan alokasi Ip host pada subnet /24 di rentang .2 - .254
switch1: Alice 192.242.1.2/24 -> host addr pertama setelah gateway 192.242.1.1
         Mika  ..........3.... gateway sama
switch2: Chisa ........2.2.... gateway 19.242.2.1
switch3: Knights ......3.2.... gateway .......3..
         Eiri  ..........3.... gateway sama

iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.242.0.0/16
/16 subnet mask mencakup seluruh rentang 192.242.0.0 hingga 192.242.255.255
semua client diizinka melewati router Lain untuk mengakses jaringan luar/internet. 

### 2.
intinya no 2 itu hubungin interface eth0 Lain ke adapter NAT, conf eth0 agar dapat alamat IP dan default route yang mengarah ke gateway virtual GNS3, dan mastiin Lain  isa konek ke luar.

```bash
# 1. Mengaktifkan interface eth0
ip link set eth0 up

# 2. Menetapkan alamat IP pada eth0
ip addr add 192.168.122.100/24 dev eth0 2>/dev/null || true

# 3. Menetapkan Default Route ke Gateway Luar
ip route replace default via 192.168.122.1 dev eth0
```
ini ada di script ```router_config.sh```

kok IP 192.168.122.100 gateway 192.168.122.1? 
node NAt bawaan GNS3 umumnya pakai libvirt/virbr0 yang jalan pada subnet default 192.168.122.0/24
IP 192.168.122.1 ini gateway yang kasih jalur keluar ke internet asli PC
192.168.122.100 jadi IP statis royter Lain yang masih se subnet degan gateway itu. agar tidak IP conflict.
pake IP statis karena soal sebut NAT/DHCP
```2>/dev/null || true``` biar pas script dijalanan berkali-kali, terminal tidak mengeluarkan pesan error. 
```ip route replace``` memstikan rute default nimpa mengarah ke gateway luar 19.2.168.122.1

testing:
```bash
# Cek IP dan route sudah terpasang
ip -br a show eth0
ip route show
# Output: eth0 UP 192.168.122.100/24


# Cek konektivitas ke gateway host dan ke internet publik
ping -c 2 192.168.122.1
ping -c 2 8.8.8.8
# Output: 2 packets transmitted, 2 received, 0% packet loss
```

### 3. 

```bash
# Mengaktifkan IP Forwarding di Linux kernel
sysctl -w net.ipv4.ip_forward=1
```
ini ada di router_config.sh

di tiap client (contoh si Alice)
```bash
# Menentukan gateway agar paket keluar switch dilempar ke router
ip route add default via 192.242.1.1
```

router directly connected ke 3 subnet via eth1-3. otomatis kernel Linux memiliki tabel rute lokal untuk ke3 segemn itu
tapi setelah itu dimatikan fungsi meneruskan paket demi keamanan. @

testing: 
run router_config.sh nanti outputnya =1
buka terminal Alice, lalu command ```ping -c 2 192.242.2.2``` (Chisa) ourputnya 0% packet loss tanda berhasil.

### 4. 

``` bash
# mengosongkan rule firewall lama agar tidak coflict atau duplicate aturan
iptables -F
iptables -t nat -F

# pasang aturan pada tabel NAT di tahap post-routing (paket harus keluar lewat interface mana)
iptables -t nat -A POSTROUTING 
# lewat sini
-o eth0 
# ganti Ip private client pengirim jadi Ip interface eth0 router secara dinamis
-j MASQUERADE 
# nentuin sumber jaringan yang diizinkan (tadi di nomor 1 mentioned)
-s 192.242.0.0/16
```
yak masih di router_config.sh

ntr tunjukin ss an aja. for double verification: ```ping -c 2 google.com```

### 5. 

cek_status.sh
``` bash
#!/bin/bash

# bried addr: ringkasan eth0-3, status link, dn alamat IP
ip -br a

# buka tabel NAT, tampilin semua rule, liatin informasi lalu lintas paket
# memastikan aturan masquerade di chain postroitng untuk subnet IP 0.0 tetap aktif.
iptables -t nat -L -v -n
```

tinggal run aja /root/cek_status.sh 
aman? harusnya sih

### 6.

Pertama, praktikan perlu mengunduh file yang disediakan asisten, kemudian menyalin isinya ke file `traffic_protocol7.sh` yang ada di `terminal node Mika`.

Kemudian, pada topologi jaringan yang telah dibuat di GNS3, kita perlu mengaktifkan "Start Capture" pada kabel yang menghubungkan Switch dengan node Mika. Jika sudah, maka  logo kaca pembesar akan muncul.
<img width="976" height="672" alt="Screenshot 2026-09-18 235134" src="https://github.com/user-attachments/assets/f5099a7e-2e43-4e82-85b2-1329c8c35adf" />

Kemudian, jalankan `traffic_protocol.sh` untuk membuat traffic.

<img width="881" height="957" alt="Screenshot 2026-09-18 235000" src="https://github.com/user-attachments/assets/ec843c4c-6371-4c46-86f5-64df81aa05d8" />

<img width="787" height="946" alt="Screenshot 2026-09-18 235025" src="https://github.com/user-attachments/assets/4da18753-8041-432e-8030-a05af958104b" />

Di Wireshark, gunakan filter `dns || icmp` untuk melacak package yang diminta sesuai soal.

<img width="1917" height="931" alt="Screenshot 2026-09-17 190702" src="https://github.com/user-attachments/assets/4aa272e2-96cf-4e28-bd81-011d86ef709b" />

<img width="1917" height="927" alt="Screenshot 2026-09-17 190717" src="https://github.com/user-attachments/assets/6ef9c206-23a3-4c21-b35d-909852540f13" />

### 14.
Fitur endpoints mengagregasi ribuan paket menjadi daftar alamat host unik (L3/IPv4) dan port layanan (L4/TCP).
menyajikan file web, script, atau gambar ke klien dan kebalikannya, minta request.
