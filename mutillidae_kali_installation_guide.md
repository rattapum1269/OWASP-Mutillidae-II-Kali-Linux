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
```c
docker --version

docker ps
```
---

# ส่วนที่ 2: ติดตั้ง OWASP Mutillidae II (Version 2.12.5)

Step 1: Clone โปรเจกต์
```c
cd C: git clone [https://github.com/webpwnized/mutillidae-docker.git](https://github.com/webpwnized/mutillidae-docker.git)
cd mutillidae-docker
```
Step 2: วาง docker-compose.yml 

```c
name: mutillidae-ii

services:

  # 1️⃣ Database
  database:
    container_name: database
    image: webpwnized/mutillidae:database
    build:
      context: ./.build/database
      dockerfile: Dockerfile
    networks:
      - datanet

  # 2️⃣ Database Admin (phpMyAdmin)
  database_admin:
    container_name: database_admin
    depends_on:
      - database
    image: webpwnized/mutillidae:database_admin
    build:
      context: ./.build/database_admin
      dockerfile: Dockerfile
    ports:
      - 127.0.0.1:81:80
    networks:
      - datanet

  # 3️⃣ Web Application (Mutillidae)
  www:
    platform: linux/amd64
    container_name: mutillidae-app
    hostname: mutillidae
    depends_on:
      - database
      - directory
    image: webpwnized/mutillidae:www
    build:
      context: ./.build/www
      dockerfile: Dockerfile
    ports:
      - "80:80"
      - "443:443"
    networks:
      datanet:
        aliases:
          - mutillidae
      ldapnet:
        aliases:
          - mutillidae
      pentest-net:
        aliases:
          - mutillidae

  # 4️⃣ LDAP
  directory:
    container_name: directory
    image: webpwnized/mutillidae:ldap
    build:
      context: ./.build/ldap
      dockerfile: Dockerfile
    volumes:
      - ldap_data:/var/lib/ldap
      - ldap_config:/etc/ldap/slapd.d
    ports:
      - 127.0.0.1:389:389
    networks:
      - ldapnet

  # 5️⃣ LDAP Admin
  directory_admin:
    container_name: directory_admin
    depends_on:
      - directory
    image: webpwnized/mutillidae:ldap_admin
    build:
      context: ./.build/ldap_admin
      dockerfile: Dockerfile
    ports:
      - 127.0.0.1:82:80
    networks:
      - ldapnet

# Volumes
volumes:
  ldap_data:
  ldap_config:

# Networks
networks:
  datanet:
  ldapnet:
  pentest-net:
    external: true
```

Step 3 : แก้ไข dockerfile

```c
FROM php:apache

ARG DATABASE_HOST="database"
ARG DATABASE_USERNAME="root"
ARG DATABASE_PASSWORD="mutillidae"
ARG DATABASE_NAME="mutillidae"
ARG DATABASE_PORT="3306"
ARG ENABLE_ERROR_REPORTING=true

RUN apt-get update && \
    apt-get install --no-install-recommends -y \
        libldap2-dev \
        libxml2-dev \
        libonig-dev \
        libcurl4-openssl-dev \
        dnsutils \
        git \
        iputils-ping \
        ntpsec && \
    docker-php-ext-install ldap xml mbstring curl mysqli && \
    cd /tmp && \
    git clone https://github.com/webpwnized/mutillidae.git mutillidae && \
    cp -r mutillidae/src /var/www/mutillidae && \
    rm -rf /tmp/mutillidae && \
    mkdir -p /var/www/mutillidae/data /var/www/mutillidae/passwords && \
    chown -R www-data:www-data /var/www/mutillidae/data /var/www/mutillidae/passwords && \
    chmod -R 775 /var/www/mutillidae/data /var/www/mutillidae/passwords && \
    apt-get remove --no-install-recommends -y git && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    useradd -M phinius

# PHP Config
RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini && \
    sed -i 's/allow_url_include = Off/allow_url_include = On/g' /usr/local/etc/php/php.ini && \
    sed -i 's/allow_url_fopen = Off/allow_url_fopen = On/g' /usr/local/etc/php/php.ini && \
    sed -i 's/expose_php = Off/expose_php = On/g' /usr/local/etc/php/php.ini

RUN if [ "$ENABLE_ERROR_REPORTING" = "true" ]; then \
    sed -i 's/^error_reporting = .*/error_reporting = E_ALL/' /usr/local/etc/php/php.ini && \
    sed -i 's/^display_errors = .*/display_errors = On/' /usr/local/etc/php/php.ini; \
    fi

# App Config
RUN rm /var/www/mutillidae/.htaccess && \
    sed -i "s/define('DB_HOST', '127.0.0.1');/define('DB_HOST', '$DATABASE_HOST');/" /var/www/mutillidae/includes/database-config.inc && \
    sed -i "s/define('DB_USERNAME', 'root');/define('DB_USERNAME', '$DATABASE_USERNAME');/" /var/www/mutillidae/includes/database-config.inc && \
    sed -i "s/define('DB_PASSWORD', 'mutillidae');/define('DB_PASSWORD', '$DATABASE_PASSWORD');/" /var/www/mutillidae/includes/database-config.inc && \
    sed -i "s/define('DB_NAME', 'mutillidae');/define('DB_NAME', '$DATABASE_NAME');/" /var/www/mutillidae/includes/database-config.inc && \
    sed -i "s/define('DB_PORT', 3306);/define('DB_PORT', $DATABASE_PORT);/" /var/www/mutillidae/includes/database-config.inc && \
    sed -i 's/127.0.0.1/directory/' /var/www/mutillidae/includes/ldap-config.inc

# Apache
RUN a2enmod ssl && \
    a2dissite 000-default

EXPOSE 80
EXPOSE 443
```
Step 4: สร้าง Network สำหรับ Lab
```c
docker network create hacking-lab
```
ตรวจสอบ:
```c
docker network ls
```
Step 5: Build Image
```c
docker compose build
```
Step 6: เปิดระบบ
```c
docker compose up -d
```
ตรวจสอบ container:
```c
docker ps
```
Step 7: เข้าใช้งานผ่าน Browser

[http://localhost](http://localhost)

ถ้าขึ้นหน้า Database Offline ให้กด "Click here to setup the database"

---

# ส่วนที่ 3: ติดตั้ง Kali Linux (VM)
 ใช้  powershell ในการรันนะ
```c
docker run -d `
  --name kali-gui `
  --network pentest-net `
  --security-opt seccomp=unconfined `
  -e PUID=1000 `
  -e PGID=1000 `
  -e TZ=Asia/Bangkok `
  -p 3500:3000 `
  --shm-size=1gb `
  --restart unless-stopped `
  lscr.io/linuxserver/kali-linux:latest
```
เข้า Kali GUI

เปิด browser Windows:

http://localhost:3500


จะเห็น Desktop ของ Kali
---

### ขั้นตอนการเปิด-ปิด

# ส่วนที่ 1: ขั้นตอนการเปิดใช้งาน (Start Lab)

## เปิด Docker + Mutillidae

1. เปิด Docker Desktop
2. เข้าโฟลเดอร์ mutillidae-docker
3. รัน:

docker compose up -d

4. ตรวจสอบ:

docker ps

## เปิด Kali

```c
docker start kali-gui
```
เข้า kali

http://localhost:3500


---

# ส่วนที่ 6: ขั้นตอนการปิดระบบ (Shutdown Lab)

## ปิด Mutillidae
```c
docker compose down
```
ถ้าต้องการหยุดเฉย ๆ (ไม่ลบ network/volume):
```c
docker compose stop
```
## ปิด Kali
```c
docker stop kali-gui
```


---

# ส่วนที่ 7: คำสั่ง Network สำคัญ

ดู network ทั้งหมด:
```c
docker network ls
```
ดูรายละเอียด network:
```c
docker network inspect hacking-lab
```
ดู container ทั้งหมด:
```c
docker ps -a
```
---

# สรุปโครงสร้าง Lab

Kali Linux (docker)
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

