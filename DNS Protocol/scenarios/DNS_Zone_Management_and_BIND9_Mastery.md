# 📚 راهنمای جامع و روان: ساخت، نگهداری و تست DNS Zoneها در لینوکس (BIND9)

این داکیومنت یک راهنمای تخصصی و در عین حال روان برای درک ساختار فایل‌های پیکربندی BIND9 (`named.conf`)، نحوه نوشتن و مدیریت **Zone Fileها**، انواع **Record Typeها**، تکنیک‌های نگهداری و ابزارهای تست و عیب‌یابی در لینوکس است.

---

## 📋 فهرست مطالب
1. [مفاهیم پایه: DNS Zone چیست؟](#1-مفاهیم-پایه-dns-zone-چیست)
2. [بررسی پیکربندی اصلی BIND9 (`named.conf`)](#2-بررسی-پیکربندی-اصلی-bind9-namedconf)
3. [ساختار فایل زون (Zone File Structure)](#3-ساختار-فایل-زون-zone-file-structure)
4. [معرفی کامل انواع رکوردها (DNS Record Types) همراه با مثال](#4-معرفی-کامل-انواع-رکوردها-dns-record-types-همراه-با-مثال)
5. [پیاده‌سازی کامل یک Forward Zone و Reverse Zone](#5-پیاده‌سازی-کامل-یک-forward-zone-و-reverse-zone)
6. [اصول نگهداری و بروزرسانی زون‌ها (Zone Maintenance)](#6-اصول-نگهداری-و-بروزرسانی-زون‌ها-zone-maintenance)
7. [ابزارها و روش‌های تست و عیب‌یابی (Zone Verification)](#7-ابزارها-و-روش‌های-تست-و-عیب‌یابی-zone-verification)

---

## 1️⃣ مفاهیم پایه: DNS Zone چیست؟

برای درک ساده، فضای نام دامنه در اینترنت مانند یک درخت بزرگ است. **DNS Zone** قسمتی از این درخت است که مسئولیت و مدیریت آن به یک سرور خاص واگذار شده است.

* **Zone File (فایل زون):** یک فایل متنی ساده (Text File) روی سرور لینوکس است که شامل تمام اطلاعات، نام‌ها و آدرس‌های IP مرتبط با یک دامنه مشخص می‌باشد.
* **Forward Zone:** ترجمه **نام دامنه به آدرس IP** (مثلاً تبدیل `web.example.com` به `192.168.10.50`).
* **Reverse Zone:** ترجمه **آدرس IP به نام دامنه** (مثلاً تبدیل `192.168.10.50` به `web.example.com`).

---

## 2️⃣ بررسی پیکربندی اصلی BIND9 (`named.conf`)

در سرویس BIND9، پیکربندی اصلی معمولاً به چند فایل تقسیم می‌شود تا مدیریت آن آسان‌تر باشد.

### 🔹 ساختار فایل‌های کانفیگ در دبیان/اوبونتو:
* `/etc/bind/named.conf`: فایل اصلی که بقیه فایل‌ها را Include می‌کند.
* `/etc/bind/named.conf.options`: تنظیمات عمومی مانند پورت‌ها، دسترسی‌ها و Forwarderها.
* `/etc/bind/named.conf.local`: محلی که زون‌های اختصاصی خود را در آن تعریف می‌کنیم.

---

### 💡 مثال ۱: تنظیمات فایل `/etc/bind/named.conf.options`
این فایل رفتار کلی سرور DNS شما را مشخص می‌کند.

```acl
acl "trusted_network" {
    127.0.0.1;
    192.168.10.0/24;
};

options {
    directory "/var/cache/bind";

    // امنیت: فقط سیستم‌های شبکه داخلی حق پرس‌وجو دارند
    allow-query { trusted_network; };

    // فعال کردن Recursion برای شبکه داخلی
    recursion yes;

    // سرورهای بالادستی برای دامنه‌های خارجی
    forwarders {
        1.1.1.1;
        8.8.8.8;
    };

    dnssec-validation auto;
    listen-on port 53 { any; };
};
```

### 💡 مثال ۲: تعریف زون‌ها در ``/etc/bind/named.conf.local``

در این فایل، زون‌های خود را معرفی و آدرس فایل مربوط به آن‌ها را مشخص می‌کنیم:

```
// تعریف زون مستقیم (Forward Zone)
zone "company.local" {
    type master;
    file "/etc/bind/zones/db.company.local";
    allow-transfer { 192.168.10.11; }; // اجازه انتقال به سرور Slave
};

// تعریف زون معکوس (Reverse Zone)
zone "10.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168.10";
    allow-transfer { 192.168.10.11; };
};
```

### 3️⃣ ساختار فایل زون (Zone File Structure)

هر فایل زون از دو بخش کلی تشکیل شده است:

1. دستورالعمل‌ها (Directives): مانند ``$TTL`` و ``$ORIGIN``.
2. رکوردهای منابع (Resource Records - RRs): مانند SOA, NS, A, CNAME و غیره.

### 🔹 دایرکتیوهای اصلی:

* ا $TTL (Time To Live): زمان پیش‌فرض ذخیره‌سازی رکوردها در کش کلاینت‌ها (بر حسب ثانیه). مثلاً 86400 یعنی ۱ روز.
* ا ``$ORIGIN``: پسوند پیش‌فرض دامنه را مشخص می‌کند. اگر در انتهای یک نام علامت نقط (.) نگذارید، این پسوند به آن اضافه می‌شود.
* علامت ``@``: اشاره به نام خود زون (مثلاً company.local.) دارد.

### 4️⃣ معرفی کامل انواع رکوردها (DNS Record Types) همراه با مثال

در این بخش انواع رکوردهای استاندارد DNS را بررسی می‌کنیم:

1. 1. رکورد SOA (Start of Authority)
  
اولین و مهم‌ترین رکورد در هر فایل زون است که اطلاعات مدیریتی زون را در خود دارد.

```
@   IN  SOA  ns1.company.local. admin.company.company. (
             2026091501 ; Serial (تاریخ + شماره ویرایش)
                  604800 ; Refresh (زمان بررسی Slave)
                   86400 ; Retry (تلاش مجدد Slave در صورت خطا)
                 2419200 ; Expire (زمان منقضی شدن داده‌ها در Slave)
                   86400 ) ; Minimum TTL
```

نکته: در ``admin.company.local.`` کاراکتر اول ``@`` به دلیل قوانین DNS تبدیل به نقطه (``.``) شده است یعنی ایمیل مدیر ``admin@company.local`` است.

2. رکورد NS (Name Server)

سرورهای DNS معتبر برای این زون را معرفی می‌کند.

مثال:

```
@       IN      NS      ns1.company.local.
@       IN      NS      ns2.company.local.
```

3. رکورد A (IPv4 Address)

نام یک میزبان (Host) را به آدرس IPv4 وصل می‌کند.

مثال:

```
ns1     IN      A       192.168.10.10
web     IN      A       192.168.10.50
db      IN      A       192.168.10.60
```

4. رکورد AAAA (IPv6 Address)

نام یک میزبان را به آدرس IPv6 وصل می‌کند.

مثال:

``web     IN      AAAA    2001:db8:85a3::8a2e:370:7334``

5. رکورد CNAME (Canonical Name)

یک نام مستعار (Alias) برای یک رکورد دیگر ایجاد می‌کند.

مثال:

```
; اشاره ftp و www به رکورد اصلی web
www     IN      CNAME   web.company.local.
ftp     IN      CNAME   web.company.local.
```

6. رکورد PTR (Pointer Record)

در زون‌های معکوس استفاده می‌شود و آدرس IP را به نام دامنه تبدیل می‌کند.

مثال:

```
50      IN      PTR     web.company.local.
60      IN      PTR     db.company.local.
```

7. رکورد MX (Mail Exchanger)

سرورهای ایمیل دامنه را به همراه اولویت (Priority) مشخص می‌کند. عدد کمتر نشان‌دهنده اولویت بالاتر است.

مثال:

```
@       IN      MX  10  mail1.company.local.
@       IN      MX  20  mail2.company.local.
```

8. رکورد TXT (Text Record)

برای ذخیره متن‌های سفارشی استفاده می‌شود و نقش حیاتی در امنیت ایمیل (SPF, DKIM, DMARC) دارد.

مثال:

```
; تعریف رکورد SPF برای جلوگیری از اسپم شدن ایمیل‌ها
@       IN      TXT     "v=spf1 ip4:192.168.10.0/24 -all"
```

9. رکورد SRV (Service Record)

مکان یک سرویس خاص (پورت و پروتکل) در شبکه را مشخص می‌کند (پرکاربرد در Active Directory و VoIP).

فرمت: ``_service._proto.name. TTL Class SRV Priority Weight Port Target``

مثال:

``_sip._udp IN    SRV     10 60 5060 phone.company.local.``

### 5️⃣ پیاده‌سازی کامل یک Forward Zone و Reverse Zone

حالا تمام موارد بالا را در یک سناریوی عملی کنار هم می‌گذاریم:

### 📁 سناریو:

* Domain: ``company.local``
* Subnet: ``192.168.10.0/24``

### 🔹 فایل ۱: Forward Zone File (``/etc/bind/zones/db.company.local``)

```
$TTL    86400
$ORIGIN company.local.
@       IN      SOA     ns1.company.local. admin.company.local. (
                              2026091501 ; Serial
                                   604800 ; Refresh
                                    86400 ; Retry
                                  2419200 ; Expire
                                    86400 ) ; Minimum TTL

; --- Name Servers ---
@       IN      NS      ns1.company.local.
@       IN      NS      ns2.company.local.

; --- Mail Servers ---
@       IN      MX  10  mail.company.local.

; --- A Records ---
ns1     IN      A       192.168.10.10
ns2     IN      A       192.168.10.11
web     IN      A       192.168.10.50
mail    IN      A       192.168.10.60
app     IN      A       192.168.10.70

; --- CNAME Records ---
www     IN      CNAME   web
portal  IN      CNAME   app

; --- TXT Records ---
@       IN      TXT     "v=spf1 mx ~all"
```

### 🔹 فایل ۲: Reverse Zone File (``/etc/bind/zones/db.192.168.10``)

```
$TTL    86400
@       IN      SOA     ns1.company.local. admin.company.local. (
                              2026091501 ; Serial
                                   604800 ; Refresh
                                    86400 ; Retry
                                  2419200 ; Expire
                                    86400 ) ; Minimum TTL

; --- Name Servers ---
@       IN      NS      ns1.company.local.
@       IN      NS      ns2.company.local.

; --- PTR Records ---
10      IN      PTR     ns1.company.local.
11      IN      PTR     ns2.company.local.
50      IN      PTR     web.company.local.
60      IN      PTR     mail.company.local.
70      IN      PTR     app.company.local.
```

### 6️⃣ اصول نگهداری و بروزرسانی زون‌ها (Zone Maintenance)

برای اینکه زون‌ها همیشه سالم و بدون اختلال بمانند، رعایت نکات زیر الزامی است:
