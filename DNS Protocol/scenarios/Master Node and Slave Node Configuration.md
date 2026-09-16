# 🔄 راهنمای جامع و روان: پیاده‌سازی عملی Master Node و Slave Node با BIND9 در لینوکس

این داکیومنت راهنمای کامل پروژه برای ساخت یک ساختار **Redundant (مبتنی بر پشتیبان)** در شبکه است. در این معماری، یک سرور اصلی (**Master Node**) تمام رکوردهای دامنه را نگهداری می‌کند و یک سرور پشتیبان (**Slave Node**) به‌صورت خودکار تمام تغییرات و فایل‌های زون را از Master دریافت کرده و همگام‌سازی (Zone Transfer) می‌کند.

---

## 📋 فهرست مطالب
1. [چرا به معماری Master / Slave نیاز داریم؟](#1-چرا-به-معماری-master--slave-نیاز-داریم)
2. [سناریو و مشخصات شبکه پروژه](#2-سناریو-و-مشخصات-شبکه-پروژه)
3. [گام اول: پیکربندی کامل Master Node](#3-گام-اول-پیکربندی-کامل-master-node)
4. [گام دوم: پیکربندی کامل Slave Node](#4-گام-دوم-پیکربندی-کامل-slave-node)
5. [گام سوم: تست عملی همگام‌سازی (Zone Transfer & NOTIFY)](#5-گام-سوم-تست-عملی-همگام‌سازی-zone-transfer--notify)
6. [گام چهارم: امنیت در همگام‌سازی با استفاده از کلید TSIG](#6-گام-چهارم-امنیت-در-همگام‌سازی-با-استفاده-از-کلید-tsig)
7. [تست و عیب‌یابی خطاهای رایج](#7-تست-و-عیب‌یابی-خطاهای-رایج)

---

## 1️⃣ چرا به معماری Master / Slave نیاز داریم؟

در شبکه‌های عملیاتی، اتکا به یک سرور DNS ریسک بالایی دارد:
* **پایداری بالا (High Availability):** اگر سرور Master به هر دلیلی (قطعی برق، بروزرسانی یا خرابی سخت‌افزار) از دسترس خارج شود، سرور Slave پاسخگوی تمام کلاینت‌ها خواهد بود.
* **توزیع بار (Load Balancing):** کلاینت‌ها می‌توانند درخواست‌های ترجمه آدرس خود را بین دو سرور تقسیم کنند.
* **کاهش خطای انسانی:** شما فقط فایل‌های زون را روی **Master Node** دستکاری می‌کنید؛ سرور Slave خودکار و بدون دخالت دست تمام تغییرات را دانلود می‌کند.

---

## 2️⃣ سناریو و مشخصات شبکه پروژه

برای سادگی فهم موضوع، فرض می‌کنیم دو سرور لینوکس در شبکه داخلی داریم:

* **نام دامنه پروژه:** `network.local`
* **محدوده شبکه:** `192.168.10.0/24`
* **مشخصات Master Node:**
  * **Hostname:** `ns1.network.local`
  * **IP Address:** `192.168.10.10`
* **مشخصات Slave Node:**
  * **Hostname:** `ns2.network.local`
  * **IP Address:** `192.168.10.11`

---

## 3️⃣ گام اول: پیکربندی کامل Master Node (`192.168.10.10`)

ابتدا پکیج‌های BIND9 را روی سرور Master نصب می‌کنیم:
```bash
sudo apt update
sudo apt install bind9 bind9-utils -y
```

### 🔹 ۱. تنظیمات عمومی (``/etc/bind/named.conf.options``)

``sudo nano /etc/bind/named.conf.options``

```
acl "trusted" {
    127.0.0.1;
    192.168.10.0/24; // شبکه داخلی
};

options {
    directory "/var/cache/bind";

    allow-query { trusted; };
    recursion yes;

    forwarders {
        8.8.8.8;
        1.1.1.1;
    };

    listen-on port 53 { 127.0.0.1; 192.168.10.10; };
    dnssec-validation auto;
};
```

### 🔹 ۲. تعریف زون‌ها با قابلیت ارسال به Slave (``/etc/bind/named.conf.local``)

در فایل کانفیگ local، تعیین می‌کنیم که فقط سرور Slave اجازه دریافت اطلاعات زون را دارد:

``sudo nano /etc/bind/named.conf.local``

```
// لیست آی‌پی سرور Slave
acl "slaves" {
    192.168.10.11;
};

// تعریف Forward Zone
zone "network.local" {
    type master;
    file "/etc/bind/zones/db.network.local";
    
    // ۱. فقط اجازه انتقال زون به Slave تعیین‌شده
    allow-transfer { slaves; };
    
    // ۲. اطلاع‌رسانی آنی تغییرات به Slave
    notify yes;
    also-notify { 192.168.10.11; };
};

// تعریف Reverse Zone
zone "10.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168.10";
    allow-transfer { slaves; };
    notify yes;
    also-notify { 192.168.10.11; };
};
```

### 🔹 ۳. ایجاد فایل Forward Zone روی Master (``/etc/bind/zones/db.network.local``)

```
sudo mkdir -p /etc/bind/zones
sudo nano /etc/bind/zones/db.network.local
```

```
$TTL    86400
$ORIGIN network.local.
@       IN      SOA     ns1.network.local. admin.network.local. (
                              2026091601 ; Serial (بسیار مهم!)
                                   604800 ; Refresh
                                    86400 ; Retry
                                  2419200 ; Expire
                                    86400 ) ; Minimum TTL

; --- هر دو سرور به‌عنوان NS معرفی می‌شوند ---
@       IN      NS      ns1.network.local.
@       IN      NS      ns2.network.local.

; --- رکوردهای A سرورها ---
ns1     IN      A       192.168.10.10
ns2     IN      A       192.168.10.11

; --- سایر سرویس‌ها ---
web     IN      A       192.168.10.50
app     IN      A       192.168.10.60
www     IN      CNAME   web
```

### 🔹 ۴. ایجاد فایل Reverse Zone روی Master (``/etc/bind/zones/db.192.168.10``)

``sudo nano /etc/bind/zones/db.192.168.10``

```
$TTL    86400
@       IN      SOA     ns1.network.local. admin.network.local. (
                              2026091601 ; Serial
                                   604800 ; Refresh
                                    86400 ; Retry
                                  2419200 ; Expire
                                    86400 ) ; Minimum TTL

@       IN      NS      ns1.network.local.
@       IN      NS      ns2.network.local.

10      IN      PTR     ns1.network.local.
11      IN      PTR     ns2.network.local.
50      IN      PTR     web.network.local.
60      IN      PTR     app.network.local.
```

تست و ری‌استارت سرور Master:

```
sudo named-checkconf
sudo named-checkzone network.local /etc/bind/zones/db.network.local
sudo systemctl restart bind9
```

### 4️⃣ گام دوم: پیکربندی کامل Slave Node (``192.168.10.11``)

روی سرور دوم لینوکس، BIND9 را نصب کنید:

```
sudo apt update
sudo apt install bind9 bind9-utils -y
```

### 🔹 ۱. تنظیمات عمومی (``/etc/bind/named.conf.options``)

``sudo nano /etc/bind/named.conf.options``

```
acl "trusted" {
    127.0.0.1;
    192.168.10.0/24;
};

options {
    directory "/var/cache/bind";

    allow-query { trusted; };
    recursion yes;

    forwarders {
        8.8.8.8;
        1.1.1.1;
    };

    listen-on port 53 { 127.0.0.1; 192.168.10.11; };
    dnssec-validation auto;
};
```

### 🔹 ۲. تعریف زون‌ها به‌صورت Slave در (``/etc/bind/named.conf.local``)

در سرور Slave، هیچ نیازی به ساخت دستی فایل‌های زون نیست! سرور خود به Master متصل شده و آن‌ها را در مسیر مشخص‌شده دانلود می‌کند.

``sudo nano /etc/bind/named.conf.local``

```
// Forward Zone به‌صورت Slave
zone "network.local" {
    type slave;
    // فایل دریافت شده در این مسیر ذخیره می‌شود (مسیر cache دسترسی نوشتن دارد)
    file "/var/cache/bind/db.network.local";
    
    // آدرس سرور Master
    masters { 192.168.10.10; };
    
    // هیچکس نباید زون را از Slave دانلود کند
    allow-transfer { none; };
};

// Reverse Zone به‌صورت Slave
zone "10.168.192.in-addr.arpa" {
    type slave;
    file "/var/cache/bind/db.192.168.10";
    masters { 192.168.10.10; };
    allow-transfer { none; };
};
```

تست و ری‌استارت سرور Slave:

```
sudo named-checkconf
sudo systemctl restart bind9
```

### 5️⃣ گام سوم: تست عملی همگام‌سازی (Zone Transfer & NOTIFY)

پس از ری‌استارت BIND روی Slave، بررسی کنید که آیا فایل‌ها به درستی منتقل شده‌اند یا خیر.

### 🧪 ۱. بررسی لاگ‌های سرور Slave

``sudo journalctl -u bind9 -n 20 --no-pager``

خروجی موفقیت‌آمیز:

```
transfer of 'network.local/IN' from 192.168.10.10#53: Transfer completed: 1 messages, 8 records
transfer of '10.168.192.in-addr.arpa/IN' from 192.168.10.10#53: Transfer completed
```

### 🧪 ۲. تست سناریوی بروزرسانی (تغییر رکورد روی Master)

برای اطمینان از کارکرد سیستم همگام‌سازی، مراحل زیر را انجام دهید:

1. فایل زون روی Master را باز کنید:

``sudo nano /etc/bind/zones/db.network.local``

2. یک رکورد جدید اضافه کرده و حتماً Serial را افزایش دهید:

```
; تغییر سریال از 2026091601 به 2026091602
2026091602 ; Serial

; اضافه کردن رکورد جدید
db  IN  A   192.168.10.70
```

3. روی Master دستور ری‌لود را بزنید:

 ``sudo rndc reload``

4. حالا روی Slave (بدون دست زدن به فایل‌ها)، رکورد جدید را تست کنید:

 ``dig @192.168.10.11 db.network.local +short``
  
 پاسخ: ``192.168.10.70`` (نشان‌دهنده همگام‌سازی موفق در چند ثانیه است!).
  
### 6️⃣ گام چهارم: امنیت در همگام‌سازی با استفاده از کلید TSIG
  
در محیط‌های حساس، تکیه بر آدرس IP برای allow-transfer کافی نیست؛ زیرا آی‌پی قابل Spoof شدن است. استفاده از TSIG (Transaction Signature) باعث می‌شود انتقال زون رمزشده و فقط با احراز هویت کلید مشترک انجام شود.

### 🔹 ۱. تولید کلید روی Master Node
  
``tsig-keygen -a hmac-sha256 transfer-key``

خروجی متناظر:

```
key "transfer-key" {
	algorithm hmac-sha256;
	secret "aB3xD7qL9+kF1zM5P...==";
};
```

### 🔹 ۲. قرار دادن کلید در هر دو سرور (Master & Slave)

یک فایل به اسم ``/etc/bind/tsig.key`` روی هر دو سرور ایجاد کرده و بلوک کلید بالا را در آن قرار دهید:

``sudo nano /etc/bind/tsig.key``

مجوز فایل را محدود کنید:

```
sudo chown root:bind /etc/bind/tsig.key
sudo chmod 640 /etc/bind/tsig.key
```

فایل کلید را در ``/etc/bind/named.conf`` هر دو سرور include کنید:

``include "/etc/bind/tsig.key";``

### 🔹 ۳. به‌روزرسانی پیکربندی Master (``named.conf.local``)

```
zone "network.local" {
    type master;
    file "/etc/bind/zones/db.network.local";
    
    // فقط در صورت داشتن کلید اجازه انتقال داده شود
    allow-transfer { key "transfer-key"; };
    
    notify yes;
    also-notify { 192.168.10.11; };
};
```

### 🔹 ۴. به‌روزرسانی پیکربندی Slave (``named.conf.local``)

در سرور Slave تعیین می‌کنیم که موقع ارتباط با Master از این کلید استفاده کند:

```
server 192.168.10.10 {
    keys { "transfer-key"; };
};

zone "network.local" {
    type slave;
    file "/var/cache/bind/db.network.local";
    masters { 192.168.10.10; };
    allow-transfer { none; };
};
```

### 7️⃣ تست و عیب‌یابی خطاهای رایج

| نوع خطا / مشکل | علت احتمالی | روش حل |
| :--- | :--- | :--- |
|``zone transfer failed: REFUSED`` | عدم تطابق IP یا کلید TSIG در ``allow-transfer`` | بررسی ACLهای سرور Master و صحت کلید TSIG |
| Slave بروزرسانی نمیشود | عدم افزایش Serial Number روی Master | چک کردن مقدار Serial و افزایش آن، سپس اجرا ``rndc reload`` |
| ``permission denied`` روی Slave | ذخیره فایل زون در مسافتی غیر از ``/var/cache/bind`` | در سیستم‌عامل‌های دبیانی فقط پوشه ``/var/cache/bind`` مجوز نوشتن برای BIND را دارد
| عدم دریافت NOTIFY | مسدود بودن پورت 53 UDP/TCP توسط فایروال | اجرای دستور ``sudo ufw allow 53`` روی هر دو سرور |

در این داکیومنت، یک ساختار استاندارد و عملیاتی DNS Master / Slave با BIND9 پیاده‌سازی شد. با به کارگیری فرایند همگام‌سازی خودکار و ایمن‌سازی آن با TSIG Key، سیستم DNS شبکه شما علاوه بر خطاپذیری (Fault Tolerance)، از امنیت بالا در تبادل داده‌ها نیز برخوردار است.
