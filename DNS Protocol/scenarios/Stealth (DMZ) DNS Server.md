# 🛡️ راهنمای جامع و ساده: راه‌اندازی و پیاده‌سازی Stealth (DMZ) DNS Server در لینوکس

این داکیومنت یک راهنمای عملی برای معماری **Stealth DNS Server** (که گاهی به آن **Hidden Master DNS** نیز می‌گویند) در منطقه **DMZ** است. هدف این معماری، مخفی کردن سرور اصلی اطلاعات دامنه‌ها (Master Node) از دید اینترنت و افزایش حداکثری امنیت شبکه سازمان است.

---

## 📋 فهرست مطالب
1. [معماری Stealth DNS Server چیست و چرا به آن نیاز داریم؟](#1-معماری-stealth-dns-server-چیست-و-چرا-به-آن-نیاز-داریم)
2. [طراحی سناریو و توپولوژی شبکه (DMZ & LAN)](#2-طراحی-سناریو-و-توپولوژی-شبکه-dmz--lan)
3. [گام اول: پیکربندی Stealth Master (سرور مخفی در LAN)](#3-گام-اول-پیکربندی-stealth-master-سرور-مخفی-در-lan)
4. [گام دوم: پیکربندی Public Slave DNS (سرور عمومی در DMZ)](#4-گام-دوم-پیکربندی-public-slave-dns-سرور-عمومی-در-dmz)
5. [تست سناریو، عیب‌یابی و اثبات مخفی بودن Master](#5-تست-سناریو-عیب‌یابی-و-اثبات-مخفی-بودن-master)

---

## 1️⃣ معماری Stealth DNS Server چیست و چرا به آن نیاز داریم؟

در حالت عادی، اگر یک سرور DNS عمومی (Public) داشته باشید، هم اینترنت و هم هکرها می‌توانند مستقیماً به آن متصل شوند. اگر هکری بتواند به این سرور نفوذ کند یا آن را از دسترس خارج کند (DDoS)، تمام اطلاعات زون‌های شما دستکاری شده یا کل سرویس‌های سازمان از دست می‌روند.

### 💡 ایده Stealth DNS چیست؟
در معماری **Stealth (Hidden Master)**، ما مدیریت زون‌ها را از پاسخ‌دهی به کاربران تفکیک می‌کنیم:

* ا **Stealth Master:** سرور اصلی نگهداری اطلاعات (Master) در یک منطقه امن (مثل شبکه داخلی یا LAN) قرار می‌گیرد و **هیچ دسترسی مستقیمی از اینترنت به آن وجود ندارد**. حتی در رکوردهای NS عمومی هم اسم این سرور آورده نمی‌شود!
* ا **Public Slave (در DMZ):** سرورهای Slave در منطقه DMZ (محدوده قابل دسترس از اینترنت) قرار می‌گیرند. این سرورها فقط یک کپی خواندنی (Read-Only) از Stealth Master می‌گیرند و تمام پاسخ‌های کاربران اینترنتی را میدهند.

### 🌟 مزایای این روش:
1. **امنیت بالا:** اگر سرور DMZ هک شود، هکر فقط یک کپی خواندنی دستش آمده و نمی‌تواند رکوردهای اصلی را دستکاری کند.
2. **پایداری (High Availability):** حتی اگر سرور داخل DMZ زیر حمله DDoS از کار بیفتد، سرور اصلی (Master) کاملاً سالم و امن در شبکه داخلی باقی می‌ماند.
3. **عدم افشای ساختار شبکه:** هکرها حتی IP سرور اصلی شما را نمی‌دانند.

---

## 2️⃣ طراحی سناریو و توپولوژی شبکه (DMZ & LAN)

برای درک بهتر، سناریوی زیر را در نظر بگیرید:
```
[ INET / Internet Users ]
│
▼ (فقط پورت 53 به DMZ باز است)
┌─────────────────────────────────────────┐
│              DMZ Zone                   │
│                                         │
│   ┌─────────────────────────────────┐   │
│   │ Public Slave DNS (BIND9)        │   │
│   │ IP: 192.168.20.50               │   │
│   │ Public IP: 203.0.113.50         │   │
│   └─────────────────────────────────┘   │
└────────────────────▲────────────────────┘
│ (فقط Zone Transfer از LAN به DMZ)
│ (پورت 53 TCP/UDP بین Master و Slave)
┌────────────────────┴────────────────────┐
│             Internal LAN                │
│                                         │
│   ┌─────────────────────────────────┐   │
│   │ Stealth Master DNS (BIND9)      │   │
│   │ IP: 10.10.10.10 (Hidden!)       │   │
│   └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

### 📐 مشخصات سناریو:
* **دامنه سازمان:** `example.com`
* **Stealth Master (مخفی در LAN):** `10.10.10.10`
* **Public Slave (در DMZ):** `192.168.20.50` (که روی اینترنت با `203.0.113.50` شناخته می‌شود)

---

## 3️⃣ گام اول: پیکربندی Stealth Master (سرور مخفی در LAN)

سرور Master در شبکه داخلی نصب می‌شود. این سرور نباید به درخواست‌های عمومی پاسخ دهد و **فقط و فقط** باید اجازه انتقال زون (Zone Transfer) به سرور Slave در DMZ را بدهد.

### 🔹 مرحله ۱: نصب BIND9
```bash
sudo apt update
sudo apt install bind9 bind9-utils -y
```

### 🔹 مرحله ۲: تنظیمات عمومی (``/etc/bind/named.conf.options``)

فایل کانفیگ را باز کنید:

``sudo nano /etc/bind/named.conf.options``

تنظیمات:

```
acl "dmz_slave" {
    192.168.20.50; // آدرس IP سرور Slave در DMZ
};

options {
    directory "/var/cache/bind";

    // امنیت مهم: این سرور نباید به کلاینت‌های ناشناس پاسخ بدهد
    allow-query { localhost; 10.10.10.0/24; };

    // غیرفعال کردن Recursion برای جلوگیری از سوءاستفاده
    recursion no;

    // خاموش کردن پاسخ‌دهی عمومی
    listen-on port 53 { 127.0.0.1; 10.10.10.10; };

    dnssec-validation auto;
};
```

### 🔹 مرحله ۳: تعریف زون و مجوز همگام‌سازی (``/etc/bind/named.conf.local``)

``sudo nano /etc/bind/named.conf.local``

تنظیمات:

```
zone "example.com" {
    type master;
    file "/etc/bind/zones/db.example.com";
    
    // ۱. فقط اجازه ارسال زون به سرور DMZ داده می‌شود
    allow-transfer { dmz_slave; };

    // ۲. به محض تغییر زون، به سرور DMZ اطلاع بده (NOTIFY)
    notify yes;
    also-notify { 192.168.20.50; };
};
```

### 🔹 مرحله ۴: ایجاد فایل زون (``/etc/bind/zones/db.example.com``)

نکته حیاتی در Stealth DNS: در رکوردهای ``NS`` داخل فایل زون، نباید نام یا IP سرور Stealth Master نوشته شود! فقط نام سرور عمومی Slave قرار می‌گیرد.

```
sudo mkdir -p /etc/bind/zones
sudo nano /etc/bind/zones/db.example.com
```

محتوای فایل زون:

```
$TTL    86400
@       IN      SOA     ns1.example.com. admin.example.com. (
                              2026091501 ; Serial
                                   604800 ; Refresh
                                    86400 ; Retry
                                  2419200 ; Expire
                                    86400 ) ; Minimum TTL
;
; Name Server Record (فقط سرور عمومی DMZ معرفی می‌شود!)
@       IN      NS      ns1.example.com.

; A Record برای خود Name Server عمومی
ns1     IN      A       203.0.113.50   ; آدرس عمومی سرور DMZ

; رکوردهای وب‌سایت و سرویس‌ها
@       IN      A       203.0.113.100  ; آدرس وب‌سایت
www     IN      CNAME   example.com.
mail    IN      A       203.0.113.110
```

### 🔹 مرحله ۵: تست و راه‌اندازی Stealth Master

```
# بررسی صحت کانفیگ‌ها
sudo named-checkconf
sudo named-checkzone example.com /etc/bind/zones/db.example.com

# ری‌استارت سرویس
sudo systemctl restart bind9
sudo systemctl enable bind9
```

### 4️⃣ گام دوم: پیکربندی Public Slave DNS (سرور عمومی در DMZ)

این سرور در DMZ قرار دارد و آدرس IP عمومی آن (``203.0.113.50``) به عنوان ``NS`` رسمی دامنه در ثبت‌کننده (Registrar) معرفی می‌شود.

### 🔹 مرحله ۱: نصب BIND9

```
sudo apt update
sudo apt install bind9 bind9-utils -y
```

### 🔹 مرحله ۲: تنظیمات عمومی (``/etc/bind/named.conf.options``)

``sudo nano /etc/bind/named.conf.options``

تنظیمات:

```
options {
    directory "/var/cache/bind";

    // این سرور باید به تمام دنیا (اینترنت) پاسخ دهد
    allow-query { any; };

    // عدم اجازه حل دامنه‌های دیگر (جلوگیری از Open Resolver شدن)
    recursion no;

    dnssec-validation auto;
    listen-on port 53 { any; };
};
```

### 🔹 مرحله ۳: تعریف زون به عنوان Slave (``/etc/bind/named.conf.local``)

``sudo nano /etc/bind/named.conf.local``

تنظیمات:

```
zone "example.com" {
    type slave;
    file "/var/cache/bind/db.example.com";
    
    // آدرس سرور Stealth Master در شبکه داخلی
    masters { 10.10.10.10; };

    // هیچکس نباید بتواند زون را از این سرور دانلود کند
    allow-transfer { none; };
};
```

### 🔹 مرحله ۴: راه‌اندازی و دریافت زون در Slave

```
# بررسی صحت فایل
sudo named-checkconf

# ری‌استارت سرویس
sudo systemctl restart bind9

# بررسی لاگ‌ها جهت اطمینان از دریافت فایل زون از Master
sudo journalctl -u bind9 -f
```

اگر کانفیگ درست باشد، در لاگ‌ها عبارت زیر را می‌بینید:

``transfer of 'example.com/IN' from 10.10.10.10#53: Transfer completed``

### 5️⃣ تست سناریو، عیب‌یابی و اثبات مخفی بودن Master

برای اطمینان از اینکه معماری Stealth به درستی عمل می‌کند، تست‌های زیر را انجام دهید:

🧪 تست ۱: بررسی پاسخ‌دهی سرور عمومی DMZ

از یک سیستم بیرونی در اینترنت دستور زیر را بزنید:

``dig @203.0.113.50 [www.example.com](https://www.example.com)``

نتیجه انتظار می‌رود: سرور پاسخ ``203.0.113.100`` را به درستی برمی‌گرداند.

### 🧪 تست ۲: استعلام رکوردهای NS (اثبات مخفی بودن Master)

``dig @203.0.113.50 example.com NS``

خروجی:

```
;; ANSWER SECTION:
example.com.    86400   IN      NS      ns1.example.com.

;; ADDITIONAL SECTION:
ns1.example.com. 86400  IN      A       203.0.113.50
```

نتیجه: هیچ اثری از نام یا آدرس IP سرور اصلی (``10.10.10.10``) در پاسخ وجود ندارد! دنیا فقط سرور DMZ را می‌شناسد.

### 🧪 تست ۳: عدم امکان Zone Transfer از دید هکرها

اگر یک هکر سعی کند کل زون شما را از سرور DMZ دانلود کند:

``dig @203.0.113.50 example.com AXFR``

خروجی:

``; Transfer failed.`` (به دلیل وجود ``allow-transfer { none; }``)

### 🛡️ تنظیمات فایروال پیشنهاد شده (UFW / IPTables)

1. روی سرور Stealth Master (``10.10.10.10``):
* بستن تمام ورودی‌های اینترنت.
* فقط باز بودن پورت ``53 TCP/UDP`` از مبدا ``192.168.20.50`` (سرور DMZ).
2. روی سرور Public Slave (``192.168.20.50``):
* باز بودن پورت ``53 TCP/UDP`` برای همه (جهت پاسخ به اینترنت).
* اجازه برقراری ارتباط روی پورت ``53`` فقط به سمت ``10.10.10.10`` برای دریافت updates.

### 🎯 جمع‌بندی

با پیاده‌سازی معماری Stealth (DMZ) DNS Server، شما سرور اصلی مدیریت دامنه‌های خود را در امن‌ترین نقطه شبکه (LAN) مخفی کردید و فقط یک سرور پاسخ‌دهنده Read-Only را در DMZ در معرض اینترنت قرار دادید. این الگوی طراحی، استاندارد طلایی سازمان‌ها برای بالا بردن امنیت و پایداری سرویس DNS است.

