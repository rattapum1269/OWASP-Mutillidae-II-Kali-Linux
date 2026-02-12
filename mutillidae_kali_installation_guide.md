# คู่มือการติดตั้ง OWASP Mutillidae II + Kali Linux แบบละเอียด (Step-by-Step)

เอกสารนี้อธิบายตั้งแต่เริ่มต้นจนสามารถใช้งาน Pentest Lab ได้จริง
ประกอบด้วย:

- การติดตั้ง Docker
- การติดตั้ง OWASP Mutillidae II (Version 2.12.5)
- การตั้งค่า Network ให้ Kali เชื่อมถึง
- ขั้นตอนการเปิด–ปิด Kali และ Mutillidae

---

# ส่วนที่ 1: การติดตั้ง Docker บน Windows

Step 1: ดาวน์โหลด Docker Desktop

- ไปที่ [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
- ดาวน์โหลดและติดตั้ง

Step 2: เปิด Docker Desktop

- รอจนสถานะขึ้นว่า Docker is running

Step 3: ตรวจสอบการทำงาน
เปิด PowerShell แล้วพิมพ์:

docker --version

docker ps

---

# ส่วนที่ 2: ติดตั้ง OWASP Mutillidae II (Version 2.12.5)

Step 1: Clone โปรเจกต์

cd C: git clone [https://github.com/webpwnized/mutillidae-docker.git](https://github.com/webpwnized/mutillidae-docker.git)
cd mutillidae-docker

Step 2: แก้ docker-compose.yml ให้ใช้ network ภายนอก (external network)
เพิ่ม network เช่น:

networks:
hacking-lab:
external: true

Step 3: สร้าง Network สำหรับ Lab

docker network create hacking-lab

ตรวจสอบ:

docker network ls

Step 4: Build Image

docker compose build

Step 5: เปิดระบบ

docker compose up -d

ตรวจสอบ container:

docker ps

Step 6: เข้าใช้งานผ่าน Browser

[http://localhost](http://localhost)

ถ้าขึ้นหน้า Database Offline ให้กด "Click here to setup the database"

---

# ส่วนที่ 3: ติดตั้ง Kali Linux (VM)

Step 1: ติดตั้ง VMware หรือ VirtualBox
Step 2: ดาวน์โหลด Kali Linux ISO
[https://www.kali.org/get-kali/](https://www.kali.org/get-kali/)

Step 3: สร้าง VM ใหม่

- RAM อย่างน้อย 4GB
- Network: เลือก "Bridged Adapter"

Step 4: ติดตั้ง Kali ตามขั้นตอนปกติ

---

# ส่วนที่ 4: การตั้งค่า Network ให้ Kali เชื่อม Mutillidae

โครงสร้าง:
Kali VM → Windows Host → Docker (Mutillidae)

Step 1: ดู IP Windows
เปิด Command Prompt:

ipconfig

ดูค่า IPv4 เช่น:
172.31.240.1

Step 2: จาก Kali ทดสอบ ping

ping 172.31.240.1

Step 3: เปิดเว็บจาก Kali

[http://172.31.240.1](http://172.31.240.1)

ถ้าเข้าได้ แสดงว่า network ถูกต้อง

---

# ส่วนที่ 5: ขั้นตอนการเปิดใช้งาน (Start Lab)

## เปิด Docker + Mutillidae

1. เปิด Docker Desktop
2. เข้าโฟลเดอร์ mutillidae-docker
3. รัน:

docker compose up -d

4. ตรวจสอบ:

docker ps

## เปิด Kali

1. เปิด VMware/VirtualBox
2. Start Kali VM
3. ตรวจสอบ IP:

ip a

4. เข้าเว็บเป้าหมาย

http\://

---

# ส่วนที่ 6: ขั้นตอนการปิดระบบ (Shutdown Lab)

## ปิด Mutillidae

docker compose down

ถ้าต้องการหยุดเฉย ๆ (ไม่ลบ network/volume):

docker compose stop

## ปิด Kali

ใน Kali terminal:

sudo poweroff

หรือปิดจากปุ่มใน VMware

---

# ส่วนที่ 7: คำสั่ง Network สำคัญ

ดู network ทั้งหมด:

docker network ls

ดูรายละเอียด network:

docker network inspect hacking-lab

ดู container ทั้งหมด:

docker ps -a

---

# สรุปโครงสร้าง Lab

Kali Linux (VM)
↓
Windows Host
↓
Docker Network (hacking-lab)
↓
Mutillidae Containers

ตอนนี้ระบบพร้อมสำหรับ:

- Nmap Scan
- SQL Injection
- XSS
- Burp Suite
- sqlmap

---

จบคู่มือการติดตั้ง

