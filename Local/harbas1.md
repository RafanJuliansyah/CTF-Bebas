# Menyelesaikan soal ctf dari sekolah 

---

### target: menemukan sebuah file.txt
---
**target mesinya `10.1.10.66`**
---
## 1. Scan dengan nmap 
```
nmap -sV -T4 10.1.10.66
```
![Screenshot 2025-06-02 101335](https://github.com/user-attachments/assets/7333e03d-f607-4edd-b276-8c5eeae0af61)

maka hasilnya akan seperti pada gambar di atas 

---

## 2. lalu cari sub domain 
```
gobuster dir -u http://10.1.10.66 -w /usr/share/wordlists/dirb/common.txt
```
![Screenshot 2025-06-02 102003](https://github.com/user-attachments/assets/f91c2faa-ec35-4dec-bb7d-afd893f5d4a2)

maka hasilnya seperti pada gambar di atas

---

## 3. masuk pada 
