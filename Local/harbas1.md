# Menyelesaikan soal ctf dari sekolah 

---

### target: menemukan sebuah file.txt
---
#### target mesinya `10.1.10.66`
---
## 1. Scan dengan nmap 
```
nmap -sV -T4 10.1.10.66
```
![Screenshot 2025-06-02 101335](https://github.com/user-attachments/assets/7333e03d-f607-4edd-b276-8c5eeae0af61)


---

## 2. lalu cari sub domain 
```
gobuster dir -u http://10.1.10.66 -w /usr/share/wordlists/dirb/common.txt
```
![Screenshot 2025-06-02 102003](https://github.com/user-attachments/assets/f91c2faa-ec35-4dec-bb7d-afd893f5d4a2)

---

## 3. masuk pada `/admin`
```
http://10.1.10.66/admin
```
![Screenshot 2025-06-02 102028](https://github.com/user-attachments/assets/df12b664-fe5b-4fe7-8f6e-eab5ca1e5f07)

#### lalu klik pada bagian notes.txt
![Screenshot 2025-06-02 102040](https://github.com/user-attachments/assets/8c3ba66d-834a-4014-b8ae-714bff55b85d)


---

## 4. 
