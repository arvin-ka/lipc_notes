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
4. **Subdomain / Host:** مانند `mail` یا `www` در `www.example.com`.

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

* ا nameserver: آدرس IP سرور DNS را مشخص می‌کند (تا ۳ سرور قابل تعریف است).
* ه search: پسوند دامنه پیش‌فرض را برای نام‌های کوتاه تنظیم می‌کند (مثلاً اگر ping server1 را بزنید، سیستم server1.mydomain.local را جستجو می‌کند).

### 🔹 روش دوم: سرویس systemd-resolved (توزیع‌های جدید دبیان/اوزونتو)

در سیستم‌های مدرن، فایل /etc/resolv.conf اغلب یک لینک نمادین (Symlink) به سرویس systemd-resolved است و نباید مستقیماً ویرایش شود.

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

### 🔹 روش سوم: ابزار nmcli (NetworkManager - توزیع‌های RHEL/CentOS/Rocky)

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

### 🔹 اولویت‌بندی تبدیل نام (/etc/nsswitch.conf)

ترتیب بررسی نام‌ها (مثلاً اول فایل /etc/hosts بررسی شود یا DNS Server) در فایل /etc/nsswitch.conf مشخص می‌شود:

``hosts: files dns``

در خط بالا، ابتدا سیستم فایل /etc/hosts محلی را می‌خواند و در صورت عدم پیدا شدن، به سراغ DNS Server می‌رود.

### 3️⃣ معرفی انواع DNS Serverها و نرم‌افزارهای مختلف

ا DNS Serverها بر اساس وظیفه‌ای که در شبکه بر عهده دارند به دسته‌های زیر تقسیم می‌شوند:

1. ا Recursive Resolver (پاسخگوی بازگشتی): درخواست کلاینت را می‌گیرد و تمام درخت DNS را طی می‌کند تا پاسخ را پیدا کند و به کلاینت تحویل دهد (مانند 8.8.8.8).
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

در این سناریو، یک شبکه سازمانی با دامنه داخلی lab.local را پیاده‌سازی می‌کنیم.

### 📐 مشخصات سناریو:

* Domain Name: lab.local
* Subnet: 192.168.10.0/24
* Master Node IP: 192.168.10.10 (Hostname: dns-master)
* Slave Node IP: 192.168.10.11 (Hostname: dns-slave)

* 
