# 🌐 Comprehensive Guide: DNS Protocol, Client Configuration, and BIND9 Master/Slave Setup in Linux

این داکیومنت یک راهنمای عملی و مفصل برای درک عمیق **پروتکل DNS**، پیکربندی سمت کلاینت در سیستم‌عامل لینوکس، آشنایی با نرم‌افزارهای مختلف سرویس‌دهنده DNS و در نهایت **پیاده‌سازی سناریوی عملی Master/Slave با BIND9** است.

---

## 📋 فهرست مطالب
1. [معرفی پروتکل DNS و مفاهیم پایه در لینوکس](#1-معرفی-پروتکل-dns-و-مفاهیم-پایه-در-لینوکس)
2. [پیکربندی سرویس DNS Client در لینوکس](#2-پیکربندی-سرویس-dns-client-در-لینوکس)
3. [معرفی انواع DNS Serverها و نرم‌افزارهای مختلف](#3-معرفی-انواع-dns-serverها-و-نرم‌افزارهای-مختلف)
4. [پیاده‌سازی عملی Master Node و Slave Node با BIND9](#4-پیاده‌سازی-عملی-master-node-و-slave-node-با-bind9)
5. [تست، عیب‌یابی و بررسی صحت عملکرد (Verification & Troubleshooting)](#5-تست-عیب‌یابی-و-بررسی-صحت-عملکرد)

---

## 1️⃣ معرفی پروتکل DNS و مفاهیم پایه در لینوکس

پروتکل **DNS (Domain Name System)** وظیفه نگاشت (Mapping) نام‌های دامنه قابل فهم برای انسان (مانند `example.com`) به آدرس‌های IP قابل فهم برای ماشین (مانند `192.168.10.50`) را بر عهده دارد.

### 🔹 خصوصیات کلیدی پروتکل:
* **پورت کاری:** پورت `53` در دو لایه **UDP** (برای پرس‌وجوهای استاندارد) و **TCP** (برای انتقال زون‌ها/Zone Transfer و پاسخ‌های بزرگتر از ۵۱۲ بایت).
* **معماری:** درخت سلسله‌مراتبی (Hierarchical Tree Structure).

### 🌳 ساختار سلسله‌مراتبی DNS:
1. **Root Zone (`.`):** در بالاترین سطح قرار دارد و توسط ۱۳ گروه سرور منطقی در جهان مدیریت می‌شود.
2. **TLD (Top-Level Domain):** دامنه‌های سطح بالا مانند `.com`, `.org`, `.ir`.
3. **SLD (Second-Level Domain):** نام اختصاصی سازمان یا مجموعه مانند `example` در `example.com`.
4. **Subdomain / Host:** مانند `mail` یا `www` در `www.example.com` (یک زیر شاخه از دامین اصلیمون یعنی همون example.com).

<p align="center"> <img width="750" height="400" alt="Gemini_Generated_Image_vxl2llvxl2llvxl2" src="https://github.com/user-attachments/assets/c139c082-d224-48d8-bcff-2112d3be3517" />


---

### 📑 رکورد‌های مهم DNS (Resource Records):

| نوع رکورد | نام کامل | کاربرد و توضیحات |
| :--- | :--- | :--- |
| **A** | IPv4 Address | نگاشت نام دامنه به آدرس IPv4 |
| **AAAA** | IPv6 Address | نگاشت نام دامنه به آدرس IPv6 |
| **CNAME** | Canonical Name | ایجاد نام مستعار (Alias) برای یک نام دیگر |
| **PTR** | Pointer Record | نگاشت آدرس IP به نام دامنه (Reverse DNS Lookup) |
| **MX** | Mail Exchanger | معرفی سرورهای دریافت ایمیل مربوط به دامنه |
| **NS** | Name Server | معرفی سرورهای DNS معتبر (Authoritative) برای یک Zone |
| **SOA** | Start of Authority | حاوی اطلاعات مدیریتی Zone (شامل Serial Number، زمان‌های Refresh/Retry و غیره) |
| **TXT** | Text Record | ذخیره متن‌های سفارشی (پرکاربرد در SPF, DKIM و احراز هویت دامنه‌ها) |

---

## 2️⃣ پیکربندی سرویس DNS Client در لینوکس

هر سیستم‌عامل لینوکسی برای تبدیل نام به IP (Name Resolution) از تنظیمات کلاینت استفاده می‌کند. در توزیع‌های مختلف، روش‌های متفاوتی برای این کار وجود دارد.

---

### 🔹 روش اول: فایل کلاسیک `/etc/resolv.conf`
این فایل قدیمی‌ترین و مستقیم‌ترین روش تعریف DNS Server برای کلاینت است.

**نمونه محتوای فایل `/etc/resolv.conf`:**
```text
nameserver 8.8.8.8
nameserver 1.1.1.1
search mydomain.local
```

* ا ``nameserver``: آدرس IP سرور DNS را مشخص می‌کند (تا ۳ سرور قابل تعریف است).
* ا ``search``: پسوند دامنه پیش‌فرض را برای نام‌های کوتاه تنظیم می‌کند (مثلاً اگر ``ping server1`` را بزنید، سیستم ``server1.mydomain.local`` را جستجو می‌کند).

### 🔹 روش دوم: سرویس ``systemd-resolved`` (توزیع‌های جدید دبیان/اوزونتو)

در سیستم‌های مدرن، فایل ``/etc/resolv.conf`` اغلب یک لینک نمادین (Symlink) به سرویس ``systemd-resolved`` است و نباید مستقیماً ویرایش شود.

تنظیم از طریق فایل اصلی:

``sudo nano /etc/systemd/resolved.conf``

محتوای پیشنهادی:

```
[Resolve]
DNS=192.168.10.10 8.8.8.8
FallbackDNS=1.1.1.1
Domains=mydomain.local
```

اعمال تغییرات:

``sudo systemctl restart systemd-resolved``

### 🔹 روش سوم: ابزار ``nmcli`` (NetworkManager - توزیع‌های RHEL/CentOS/Rocky)

اگر سیستم از NetworkManager استفاده می‌کند، بهترین روش اعمال تنظیمات از طریق CLI آن است:

```
# مشاهده کارت‌های شبکه
nmcli connection show

# ست کردن DNS روی کارت شبکه eth0
sudo nmcli connection modify eth0 ipv4.dns "192.168.10.10 8.8.8.8"
sudo nmcli connection modify eth0 ipv4.ignore-auto-dns yes

# اعمال مجدد تنظیمات کارت شبکه
sudo nmcli connection up eth0
```

### 🔹 اولویت‌بندی تبدیل نام (``/etc/nsswitch.conf``)

ترتیب بررسی نام‌ها (مثلاً اول فایل ``/etc/hosts`` بررسی شود یا DNS Server) در فایل ``/etc/nsswitch.conf`` مشخص می‌شود:

``hosts: files dns``

در خط بالا، ابتدا سیستم فایل ``/etc/hosts`` محلی را می‌خواند و در صورت عدم پیدا شدن، به سراغ DNS Server می‌رود.

### 3️⃣ معرفی انواع DNS Serverها و نرم‌افزارهای مختلف

ا DNS Serverها بر اساس وظیفه‌ای که در شبکه بر عهده دارند به دسته‌های زیر تقسیم می‌شوند:

1. ا Recursive Resolver (پاسخگوی بازگشتی): درخواست کلاینت را می‌گیرد و تمام درخت DNS را طی می‌کند تا پاسخ را پیدا کند و به کلاینت تحویل دهد (مانند ``8.8.8.8``).
2. ا Authoritative Server (سرور مرجع): پاسخ‌های قطعی و رسمی برای یک Zone خاص را در اختیار دارد.
3. ا Master (Primary) Node: سرور اصلی که فایل‌های اصلی Zone روی آن قرار دارد و ویرایش داده‌ها فقط روی آن انجام می‌شود.
4. ا Slave (Secondary) Node: کپی همگام‌سازی‌شده از Master دریافت می‌کند و برای بالا بردن پایداری (Redundancy) و توزیع بار (Load Balancing) استفاده می‌شود.
5. ا Forwarding DNS: درخواست‌ها را پردازش نمی‌کند، بلکه فقط به سرورهای دیگر پاس می‌دهد و نتیجه را Cache می‌کند.

### 🛠️ محبوب‌ترین نرم‌افزارهای DNS در لینوکس:

* BIND9 (Berkeley Internet Name Domain):

ویژگی‌ها: قدیمی‌ترین، استاندارترین و پرکاربردترین نرم‌افزار DNS سرور. پشتیبانی کامل از تمام استانداردهای IETF و امکانات Master/Slave و Dynamic DNS.

کاربرد: شبکه سازمان‌ها، ISPها و دیتا‌سنترها.

* Unbound:

ویژگی‌ها: بسیار سبک، امن و سریع. صرفاً به عنوان یک Recursive Caching Resolver طراحی شده است.

* Dnsmasq:

ویژگی‌ها: ترکیب DNS Resolver و DHCP Server با حجم بسیار کم و کانفیگ بسیار ساده.

کاربرد: شبکه‌های کوچک خانگی، مودم‌ها، سیستم‌های IoT و محیط‌های تست.

* PowerDNS:

ویژگی‌ها: قابلیت ذخیره‌سازی رکوردها در دیتابیس‌های SQL (MySQL, PostgreSQL) و دارای Web UIهای قدرتمند.

### 4️⃣ پیاده‌سازی عملی Master Node و Slave Node با BIND9

در این سناریو، یک شبکه سازمانی با دامنه داخلی``lab.local`` را پیاده‌سازی می‌کنیم.

### 📐 مشخصات سناریو:

* Domain Name: ``lab.local``
* Subnet: ``192.168.10.0/24``
* Master Node IP: ``192.168.10.10`` (Hostname: ``dns-master``)
* Slave Node IP: ``192.168.10.11`` (Hostname: ``dns-slave``)

### ا 🅰️ بخش اول: کانفیگ سرور Master (``192.168.10.10``)

### گام ۱: نصب BIND9

```
sudo apt update
sudo apt install bind9 bind9-utils bind9-doc -y
```

### گام ۲: تنظیم فایل کانفیگ اصلی (``/etc/bind/named.conf.options``)

این فایل تنظیمات عمومی سرور شامل Forwarderها و دسترسی‌ها را کنترل می‌کند.

``sudo nano /etc/bind/named.conf.options``

محتوای فایل:

```
// تعریف لیست دسترسی شبکه‌های مجاز
acl "allowed_clients" {
    127.0.0.1;
    192.168.10.0/24;
};

options {
    directory "/var/cache/bind";

    // امنیت: فقط شبکه‌های تعریف شده حق پرس‌وجو دارند
    allow-query { allowed_clients; };

    // فعال‌سازی قابلیت Recursion برای کاربران شبکه داخلی
    recursion yes;

    // ارسال درخواست‌های خارج از زون به این سرورها
    forwarders {
        8.8.8.8;
        1.1.1.1;
    };

    dnssec-validation auto;
    listen-on-v6 { any; };
};
```

### گام ۳: تعریف Zoneها در Master (``/etc/bind/named.conf.local``)

در این فایل زون مستقیم (Forward Zone) و زون معکوس (Reverse Zone) را معرفی می‌کنیم.

``sudo nano /etc/bind/named.conf.local``

محتوای فایل:

```
// Forward Zone
zone "lab.local" {
    type master;
    file "/etc/bind/zones/db.lab.local";
    allow-transfer { 192.168.10.11; }; // مجوز انتقال زون فقط به Slave Node
    also-notify { 192.168.10.11; };    // اطلاع‌رسانی به Slave هنگام تغییرات
};

// Reverse Zone
zone "10.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.192.168.10";
    allow-transfer { 192.168.10.11; };
    also-notify { 192.168.10.11; };
};
```

### گام ۴: ایجاد دایرکتوری و ساخت فایل زون مستقیم (``db.lab.local``)

```
sudo mkdir -p /etc/bind/zones
sudo nano /etc/bind/zones/db.lab.local
```

محتوای فایل:

```
$TTL    86400
@       IN      SOA     ns1.lab.local. admin.lab.local. (
                              2026091401 ; Serial (Format: YYYYMMDDnn)
                                   604800 ; Refresh (1 week)
                                    86400 ; Retry (1 day)
                                  2419200 ; Expire (4 weeks)
                                    86400 ) ; Minimum TTL
;
; Name Servers (NS Records)
@       IN      NS      ns1.lab.local.
@       IN      NS      ns2.lab.local.

; A Records for Name Servers
ns1     IN      A       192.168.10.10
ns2     IN      A       192.168.10.11

; Hosts A Records
@       IN      A       192.168.10.10
web     IN      A       192.168.10.50
mail    IN      A       192.168.10.60

; CNAME Record
www     IN      CNAME   web.lab.local.
```

### گام ۵: ساخت فایل زون معکوس (``db.192.168.10``)

``sudo nano /etc/bind/zones/db.192.168.10``

محتوای فایل:

```
$TTL    86400
@       IN      SOA     ns1.lab.local. admin.lab.local. (
                              2026091401 ; Serial
                                   604800 ; Refresh
                                    86400 ; Retry
                                  2419200 ; Expire
                                    86400 ) ; Minimum TTL
;
; Name Servers
@       IN      NS      ns1.lab.local.
@       IN      NS      ns2.lab.local.

; PTR Records (Last octet of IP address)
10      IN      PTR     ns1.lab.local.
11      IN      PTR     ns2.lab.local.
50      IN      PTR     web.lab.local.
60      IN      PTR     mail.lab.local.
```

### گام ۶: تست سلامت کانفیگ‌ها و راه‌اندازی سرویس در Master

```
# بررسی سینتکس فایل‌های کانفیگ عمومی
sudo named-checkconf

# بررسی صحت ساختار فایل‌های زون
sudo named-checkzone lab.local /etc/bind/zones/db.lab.local
sudo named-checkzone 10.168.192.in-addr.arpa /etc/bind/zones/db.192.168.10

# ری‌استارت و فعال‌سازی سرویس
sudo systemctl restart bind9
sudo systemctl enable bind9
```

### ا 🅱️ بخش دوم: کانفیگ سرور Slave (``192.168.10.11``)

در سرور Slave، نیازی به ساخت فایل‌های زون به‌صورت دستی نیست؛ این فایل‌ها به‌صورت خودکار از سرور Master دریافت (Zone Transfer) و ذخیره می‌شوند.

### گام ۱: نصب BIND9 در سرور Slave

```
sudo apt update
sudo apt install bind9 bind9-utils -y
```

### گام ۲: تنظیم فایل کانفیگ عمومی (``/etc/bind/named.conf.options``)

``sudo nano /etc/bind/named.conf.options``

محتوای فایل:

```
acl "allowed_clients" {
    127.0.0.1;
    192.168.10.0/24;
};

options {
    directory "/var/cache/bind";
    allow-query { allowed_clients; };
    recursion yes;
    forwarders {
        8.8.8.8;
    };
    dnssec-validation auto;
};
```

### گام ۳: تعریف زون‌ها به عنوان Slave (``/etc/bind/named.conf.local``)

``sudo nano /etc/bind/named.conf.local``

محتوای فایل:

```
// Forward Zone
zone "lab.local" {
    type slave;
    file "/var/cache/bind/db.lab.local"; // مسیر ذخیره‌سازی فایل همگام‌شده
    masters { 192.168.10.10; };          // آدرس سرور Master
};

// Reverse Zone
zone "10.168.192.in-addr.arpa" {
    type slave;
    file "/var/cache/bind/db.192.168.10";
    masters { 192.168.10.10; };
};
```

### گام ۴: راه‌اندازی و بررسی دریافت زون در Slave

```
# بررسی کانفیگ
sudo named-checkconf

# ری‌استارت سرویس
sudo systemctl restart bind9
sudo systemctl enable bind9

# بررسی لاگ‌ها برای اطمینان از انتقال موفق زون (Zone Transfer)
sudo journalctl -u bind9 -f
```

در لاگ‌ها جملاتی شبیه به ``transfer of 'lab.local/IN' from 192.168.10.10#53: Transfer completed`` مشاهده خواهید کرد.

### 5️⃣ تست، عیب‌یابی و بررسی صحت عملکرد

پس از راه‌اندازی کامل Master و Slave، باید عملکرد سرورها را با ابزارهای تست بررسی کنیم.

### 🔹 ۱. ابزار dig`` (Domain Information Groper)``

الف) تست نگاشت مستقیم (A Record):

``dig @192.168.10.10 web.lab.local``

خروجی نمونه:

```
;; ANSWER SECTION:
web.lab.local.      86400   IN      A       192.168.10.50
```

ب) تست نگاشت معکوس (PTR Record):

``dig @192.168.10.10 -x 192.168.10.50``

خروجی نمونه:

```
;; ANSWER SECTION:
50.10.168.192.in-addr.arpa. 86400 IN PTR     web.lab.local.
```

ج) تست درخواست از سرور Slave:

``dig @192.168.10.11 www.lab.local``

### 🔹 ۲. ابزار ``nslookup``

``nslookup mail.lab.local 192.168.10.10``

### 🔹 ۳. بررسی فرایند Zone Transfer به صورت دستی

برای تست اینکه آیا زون می‌تواند از Master به Slave منتقل شود:

``dig @192.168.10.10 lab.local AXFR``

نکته امنیتی: این دستور فقط باید از سمت IP سرور Slave پاسخ داده شود و برای سایر کلاینت‌ها باید پیغام ``Transfer failed`` برگرداند.

### ⚠️ نکات کلیدی در بروزرسانی رکوردها:

هرگاه رکوردی را در سرور Master تغییر می‌دهید، حتماً باید مقدار ``Serial`` را در فایل زون افزایش دهید (مثلاً از ``2026091401``به ``2026091402``). سپس سرویس را reload کنید:

``sudo systemctl reload bind9``

سرور Slave با مقایسه عدد Serial متوجه تغییرات شده و زون جدید را دریافت می‌کند.

### 🎯 جمع‌بندی

در این داکیومنت، مفاهیم اولیه و ساختاری پروتکل DNS، نحوه تنظیم کلاینت در توزیع‌های مختلف لینوکس، مقایسه ابزارهای محبوب مانند BIND9 و Unbound و در نهایت پیاده‌سازی کامل سناریوی Master/Slave BIND9 همراه با زون‌های مستقیم و معکوس آموزش داده شد.
