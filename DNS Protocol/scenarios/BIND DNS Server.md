# 🚀 راهنمای جامع و صفر تا صد: پروژه راه‌اندازی و مدیریت BIND DNS Server در لینوکس

این داکیومنت مرجع کامل پروژه راه‌اندازی سرویس‌دهنده BIND9 در لینوکس است. هدف این پروژه، پیاده‌سازی یک DNS Server عملیاتی، امن و بهینه برای شبکه داخلی و سرویس‌های سازمان است.

---

## 📋 فهرست مطالب
1. [مفاهیم پایه: DNS چیست و چگونه کار می‌کند؟](#1-مفاهیم-پایه-dns-چیست-و-چگونه-کار-می‌کند)
2. [سناریو و مشخصات پروژه](#2-سناریو-و-مشخصات-پروژه)
3. [گام اول: نصب و آماده‌سازی BIND9 در لینوکس](#3-گام-اول-نصب-و-آماده‌سازی-bind9-در-لینوکس)
4. [گام دوم: پیکربندی فایل‌های اصلی BIND9 (`named.conf`)](#4-گام-دوم-پیکربندی-فایل‌های-اصلی-bind9-namedconf)
5. [گام سوم: ساخت فایل‌های Zone (Direct & Reverse)](#5-گام-سوم-ساخت-فایل‌های-zone-direct--reverse)
6. [گام چهارم: پیکربندی امنیت، ACL و دسترسی‌ها](#6-گام-چهارم-پیکربندی-امنیت-acl-و-دسترسی‌ها)
7. [گام پنجم: تست، اعتبارسنجی و عیب‌یابی با ابزارها](#7-گام-پنجم-تست-اعتبارسنجی-و-عیب‌یابی-با-ابزارها)
8. [نکات کلیدی نگهداری و بروزرسانی پروژه](#8-نکات-کلیدی-نگهداری-و-بروزرسانی-پروژه)

---

## 1️⃣ مفاهیم پایه: DNS چیست و چگونه کار می‌کند؟

سیستم نام دامنه (DNS - Domain Name System) مثل دفترچه تلفن شبکه است. انسان‌ها نام‌ها را راحت‌تر به خاطر می‌سپارند (مثل `lab.local`) اما کامپیوترها و تجهیزات شبکه با آدرس‌های IP کار می‌کنند (مثل `192.168.10.10`).

### 💡 انواع مدکار DNS Server:
1. **Authoritative DNS:** سروری که مسئولیت رسمی یک دامنه خاص را بر عهده دارد و پاسخ نهایی رکوردهای آن دامنه را می‌دهد.
2. **Recursive / Caching DNS:** سروری که درخواست‌های کلاینت‌ها را می‌گیرد، در صورت عدم وجود در کش، از بقیه سرورهای جهان می‌پرسد و پاسخ را به کلاینت برمی‌گرداند.

---

## 2️⃣ سناریو و مشخصات پروژه

در این پروژه می‌خواهیم یک سرور DNS برای شبکه داخلی یک شرکت راه‌اندازی کنیم:

* **نام دامنه پروژه:** `lab.local`
* **محدوده شبکه (Subnet):** `192.168.10.0/24`
* **آدرس IP سرور DNS:** `192.168.10.10`
* **میزبان‌های موجود در شبکه (Hosts):**
  * ا `ns1.lab.local` ➔ `192.168.10.10` (خود سرور DNS)
  * ا `web.lab.local` ➔ `192.168.10.50` (سرور وب)
  * ا `mail.lab.local` ➔ `192.168.10.60` (سرور ایمیل)
  * ا `app.lab.local` ➔ `192.168.10.70` (سرور برنامه)

---

## 3️⃣ گام اول: نصب و آماده‌سازی BIND9 در لینوکس

در توزیع‌های مبتنی بر دبیان/اوبونتو، پکیج BIND با نام `bind9` و ابزارهای آن با `bind9-utils` شناخته می‌شوند.

### 🔹 دستورات نصب:
```bash
sudo apt update
sudo apt install bind9 bind9-utils bind9-doc -y
```

### 🔹 بررسی وضعیت سرویس:

```
sudo systemctl status bind9
sudo systemctl enable bind9
```

### 4️⃣ گام دوم: پیکربندی فایل‌های اصلی BIND9 (``named.conf``)

ساختار فایل‌های پیکربندی BIND در مسیر ``/etc/bind/`` به شرح زیر است:

* ا ``named.conf``: فایل ریشه که فایل‌های دیگر را فراخوانی می‌کند.
* ا ``named.conf.options``: تنظیمات عمومی (پورت، دسترسی، Forwarderها).
* ا ``named.conf.local``: محل تعریف Zoneهای اختصاصی پروژه.

### 💡 ۱. تنظیمات عمومی (``/etc/bind/named.conf.options``)

فایل را باز کرده و پیکربندی زیر را اعمال کنید:

``sudo nano /etc/bind/named.conf.options``

```
// تعریف لیست دسترسی شبکه داخلی
acl "trusted_network" {
    127.0.0.1;          // خود سرور
    192.168.10.0/24;    // کل شبکه داخلی
};

options {
    directory "/var/cache/bind";

    // امنیت: فقط سیستم‌های شبکه داخلی حق ارسال پرس‌وجو دارند
    allow-query { trusted_network; };

    // اجازه پرس‌وجوی بازگشتی (Recursion) فقط برای شبکه داخلی
    recursion yes;
    allow-recursion { trusted_network; };

    // سرورهای بالادستی جهت حل دامنه‌های اینترنتی (مثل گوگل و کلادفلر)
    forwarders {
        1.1.1.1;
        8.8.8.8;
    };

    // گوش دادن روی پورت 53 بر روی کارت‌های شبکه تعیین شده
    listen-on port 53 { 127.0.0.1; 192.168.10.10; };
    listen-on-v6 { none; }; // غیرفعال کردن IPv6 در صورت عدم نیاز

    dnssec-validation auto;
    
    // مخفی کردن نسخه BIND جهت افزایش امنیت
    version "Not Disclosed";
};
```

### 💡 ۲. تعریف زون‌ها (``/etc/bind/named.conf.local``)

در این فایل، مشخص می‌کنیم سرور ما مسئول چه دامنه‌هایی است.

``sudo nano /etc/bind/named.conf.local``

```
// تعریف Forward Zone (ترجمه نام به آدرس IP)
zone "lab.local" {
    type master;
    file "/etc/bind/zones/db.lab.local";
    allow-transfer { none; }; // عدم اجازه دانلود کامل زون به سیستم‌های دیگر
};

// تعریف Reverse Zone (ترجمه آدرس IP به نام)
zone "10.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168.10";
    allow-transfer { none; };
};
```

### 5️⃣ گام سوم: ساخت فایل‌های Zone (Direct & Reverse)

پوشه‌ای برای نگهداری فایل‌های زون می‌سازیم:

``sudo mkdir -p /etc/bind/zones``

### 🔹 ۱. ساخت فایل Forward Zone (``/etc/bind/zones/db.lab.local``)

این فایل وظیفه نگهداری رکوردهای مستقیم دامنه ``lab.local`` را دارد.

``sudo nano /etc/bind/zones/db.lab.local``

```
$TTL    86400
$ORIGIN lab.local.
@       IN      SOA     ns1.lab.local. admin.lab.local. (
                              2026091501 ; Serial (تاریخ + شماره تغییر)
                                   604800 ; Refresh
                                    86400 ; Retry
                                  2419200 ; Expire
                                    86400 ) ; Minimum TTL

; --- Name Server Records ---
@       IN      NS      ns1.lab.local.

; --- Mail Exchange Records ---
@       IN      MX  10  mail.lab.local.

; --- A Records (IP v4) ---
ns1     IN      A       192.168.10.10
web     IN      A       192.168.10.50
mail    IN      A       192.168.10.60
app     IN      A       192.168.10.70

; --- CNAME Records (Alias) ---
www     IN      CNAME   web
portal  IN      CNAME   app

; --- TXT Records ---
@       IN      TXT     "v=spf1 mx ~all"
```

نکته بسیار مهم (Trailing Dot): حتماً در انتهای نام‌های کامل مانند ``ns1.lab.local.`` علامت نقطه (``.``) بگذارید، در غیر این صورت BIND نام زون را به انتهای آن اضافه می‌کند!

### 🔹 ۲. ساخت فایل Reverse Zone (``/etc/bind/zones/db.192.168.10``)

این فایل آدرس‌های IP را به نام‌های دامنه نگاشت می‌کند.

``sudo nano /etc/bind/zones/db.192.168.10``

```
$TTL    86400
@       IN      SOA     ns1.lab.local. admin.lab.local. (
                              2026091501 ; Serial
                                   604800 ; Refresh
                                    86400 ; Retry
                                  2419200 ; Expire
                                    86400 ) ; Minimum TTL

; --- Name Server Records ---
@       IN      NS      ns1.lab.local.

; --- PTR Records (Pointer) ---
10      IN      PTR     ns1.lab.local.
50      IN      PTR     web.lab.local.
60      IN      PTR     mail.lab.local.
70      IN      PTR     app.lab.local.
```

### 6️⃣ گام چهارم: پیکربندی امنیت، ACL و دسترسی‌ها

برای ایمن‌سازی سرور در محیط تولید (Production)، اقدامات زیر ضروری است:

### 🛡️ ۱. تنظیم مجوزهای دسترسی فایل‌ها (Permissions)

```
sudo chown -R bind:bind /etc/bind/zones
sudo chmod 755 /etc/bind/zones
sudo chmod 644 /etc/bind/zones/*
```

### 🛡️ ۲. پیکربندی فایروال (UFW)

پورت 53 باید هم در پروتکل UDP (برای پرس‌وجوهای عادی) و هم TCP (برای پاسخ‌های بزرگ و همگام‌سازی) باز باشد:

```
sudo ufw allow 53/tcp
sudo ufw allow 53/udp
sudo ufw reload
```

### 7️⃣ گام پنجم: تست، اعتبارسنجی و عیب‌یابی با ابزارها

پیش از فعال‌سازی نهایی، باید تمام کانفیگ‌ها تست شوند.

### 🧪 ۱. تست صحت ساختار فایل‌های اصلی (``named-checkconf``)

``sudo named-checkconf``

(اگر سینتکس فایل‌ها درست باشد، هیچ خروجی یا خطایی نمایش داده نمی‌شود).

### 🧪 ۲. تست صحت فایل‌های زون (named-checkzone)

```
# بررسی Forward Zone
sudo named-checkzone lab.local /etc/bind/zones/db.lab.local

# بررسی Reverse Zone
sudo named-checkzone 10.168.192.in-addr.arpa /etc/bind/zones/db.192.168.10
```

خروجی موفق:

```
zone lab.local/IN: loaded serial 2026091501
OK
```

### 🧪 ۳. ری‌استارت سرویس BIND9

پس از تایید سلامت فایل‌ها:

```
sudo systemctl restart bind9
```

### 🧪 ۴. تست عملی عملکرد با دستور dig

* تست ترجمه نام به IP (A Record):

``dig @192.168.10.10 web.lab.local +short``

پاسخ متناظر: ``192.168.10.50``

* تست ترجمه CNAME:

``dig @192.168.10.10 www.lab.local``

* تست ترجمه معکوس IP به نام (PTR Record):

``dig @192.168.10.10 -x 192.168.10.70 +short``

پاسخ متناظر: ``app.lab.local.``

* تست عملکرد Forwarder (حل دامنه‌های عمومی اینترنت):

``dig @192.168.10.10 google.com +short``

### 8️⃣ نکات کلیدی نگهداری و بروزرسانی پروژه

1. قانون Serial Number: هر زمان که تغییراتی در فایل‌های زون ایجاد کردید، حتماً مقدار ``Serial`` را در رکورد SOA افزایش دهید (مثلاً از ``2026091501`` به ``2026091502``).
2. اعمال تغییرات بدون قطع سرویس: به جای ``restart`` کردن سرویس BIND، از دستور زیر استفاده کنید:

``sudo rndc reload``

3. بررسی لاگ‌های سرویس جهت عیب‌یابی:

``sudo journalctl -u bind9 -f``

### 🎯 جمع‌بندی

در این پروژه، ساختار جامع و ایمن یک BIND DNS Server روی لینوکس پیاده‌سازی شد. سیستم به‌گونه‌ای پیکربندی شده است که علاوه بر پاسخ‌دهی دقیق به نام‌ها و آدرس‌های شبکه داخلی (``lab.local``)، درخواست‌های خارجی را نیز به‌صورت امن از طریق Forwarderها حل کرده و بالاترین سطح پایداری و امنیت را ارائه می‌دهد.

