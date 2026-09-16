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


### 3️⃣ ابزارهای بررسی و عیب‌یابی سرویس DNS سمت Client

برای تست درست کار کردن، اندازه‌گیری سرعت، یا رفع اشکال درخواست‌های DNS در سمت کلاینت لینوکس، ابزارهای قدرتمندی وجود دارند.

### 🛠️ ۱. ابزار dig`` (Domain Information Groper)``

محبوب‌ترین و دقیق‌ترین ابزار عیب‌یابی DNS که تمامی جزئیات پاسخ دریافت شده از سرور را نمایش می‌دهد.

#### مثال ۱: پرس‌وجوی ساده برای دریافت رکورد A

``dig web.lab.local``

#### مثال ۲: پرس‌وجو از یک سرور DNS خاص (عدم استفاده از DNS پیش‌فرض سیستم)

``dig @192.168.10.10 web.lab.local``

#### مثال ۳: دریافت نوع خاصی از رکورد (مثلاً MX یا TXT یا NS)

```
dig lab.local MX
dig lab.local TXT
```

#### مثال ۴: نگاشت معکوس (Reverse Lookup - دریافت نام از روی IP)

``dig @192.168.10.10 -x 192.168.10.50``

#### مثال ۵: دریافت پاسخ مختصر و کوتاه (Short Output)

``dig web.lab.local +short``

خروجی: ``192.168.10.50``

### 🛠️ ۲. ابزار nslookup

ابزاری ساده‌تر و کلاسیک برای بررسی سریع رکوردها.

#### مثال ۱: استفاده در حالت یک‌خطی (Interactive & Non-Interactive)

``nslookup web.lab.local 192.168.10.10``

#### مثال ۲: ورود به محیط تعاملی nslookup

```
nslookup
> server 192.168.10.10
> set type=MX
> lab.local
> exit
```

### 🛠️ ۳. ابزار host

ابزاری بسیار سریع، ساده و خوانا برای بررسی وضعیت نام‌ها و آدرس‌ها.

#### مثال ۱: تبدیل نام به آدرس IP

``host web.lab.local``

خروجی: ``web.lab.local has address 192.168.10.50``

#### مثال ۲: تبدیل IP به نام (Reverse Lookup)

``host 192.168.10.50``

#### مثال ۳: نمایش تمام رکوردهای یک دامنه

``host -a lab.local``

### 🛠️ ۴. ابزار resolvectl (مخصوص سیستم‌های مبتنی بر systemd-resolved)

برای مشاهده وضعیت لحظه‌ای و خالی کردن کش سمت کلاینت استفاده می‌شود.

مشاهده وضعیت DNS فعال روی هر کارت شبکه:

``resolvectl status``

تست تبدیل نام از طریق systemd-resolved:

``resolvectl query web.lab.local``

خالی کردن کش (Flush) سمت کلاینت:

``sudo resolvectl flush-caches``


### 4️⃣ معرفی و بررسی انواع DNS Serverها (همراه با سناریو و مثال)

سرورهای DNS بر اساس وظیفه‌ای که در چرخه ترجمه آدرس بر عهده دارند، به دسته‌های زیر تقسیم می‌شوند:

### 💡 ۱. Authoritative DNS Server (سرور مرجع / صاحب زون)
این سرور **پاسخ نهایی و قطعی** رکوردهای یک یا چند دامنه مشخص را در اختیار دارد. اطلاعات دامنه‌ها مستقیماً در فایل‌های زون این سرور ذخیره شده‌اند.

* **ویژگی‌ها:** 
  * مسئولیت مستقیم روی دامنه دارد (مثلاً `example.com`).
  * اگر رکوردی در آن نباشد، پاسخ `NXDOMAIN` (عدم وجود دامنه) می‌دهد و از سرور دیگری سوال نمی‌کند.
* **مثال سناریو:**
  وقتی شما قصد دارید سایت شرکت خود (`mycompany.com`) را روی اینترنت بالا بیاورید، رکوردهای A و MX شما روی یک Authoritative DNS Server قرار می‌گیرند تا تمام کاربران دنیا بتوانند آدرس سایت شما را پیدا کنند.

---

### 💡 ۲. Recursive DNS Server / Resolver (سرور حل‌کننده بازگشتی)
این سرور میانجی بین کلاینت (کاربر) و کل شبکه اینترنت/DNS جهان است. کلاینت فقط از این سرور سوال می‌پرسد و این سرور به نیابت از کلاینت، کل اینترنت را می‌گردد تا پاسخ را پیدا کند.

* **ویژگی‌ها:**
  * فرایند **Recursion** (پرس‌وجوی مرحله‌به‌مرحله از Root Serverها، TLDها و Authoritativeها) را انجام می‌دهد.
  * نتایج را در حافظه کوتاه مدت خود (Cache) نگه می‌دارد.
* **مثال سناریو:**
  در یک سازمان با ۵۰۰ کارمند، تمام کامپیوترها به یک Recursive DNS اشاره می‌کنند. وقتی اولین نفر سایت `google.com` را باز می‌کند، سرور آدرس را از اینترنت پیدا کرده و کش می‌کند. وقتی ۴۹۹ نفر بعدی همان سایت را باز کنند، سرور بلافاصله از کش پاسخ می‌دهد و ترافیک اینترنت مصرف نمی‌شود.

---

### 💡 ۳. Caching-Only DNS Server (سرور فقط کش‌کننده)
یک نوع خاص از Recursive DNS است که **هیچ زون اختصاصی یا رکوردی روی خود ندارد**. تنها هدف آن، ذخیره‌سازی پاسخ‌ها در حافظه کش برای افزایش سرعت شبکه داخلی است.

* **مثال سناریو:**
  راه‌اندازی یک سرور DNS کوچک روی روترهای خانگی یا شعب کوچک شرکت‌ها برای افزایش سرعت وب‌گردی کاربران.

---

### 💡 ۴. Forwarding DNS Server (سرور هدایت‌کننده)
این سرور درخواست‌های کلاینت‌ها را دریافت کرده و به‌جای اینکه خودش در اینترنت به دنبال پاسخ بگردد، تمام درخواست‌ها را به یک سرور بالادستی (Upstream DNS) مشخص هدایت (Forward) می‌کند.

* **مثال سناریو:**
  یک سرور در شبکه داخلی سازمان درخواست‌ها را دریافت کرده و به سرورهای عمومی مانند `8.8.8.8` یا `1.1.1.1` پاس می‌دهد.

---

### 💡 ۵. Master (Primary) & Slave (Secondary) DNS Servers
این دو نقش برای ایجاد پشتیبان (Redundancy) در سرورهای Authoritative استفاده می‌شوند:

* ا **Master Server:** فایل اصلی زون‌ها روی این سرور ویرایش و نگهداری می‌شود.
* ا **Slave Server:** یک کپی خواندنی (Read-Only) از Master می‌گیرد. اگر Master از کار بیفتد، Slave بدون قطعی پاسخ کاربران را می‌دهد.
* **مثال سناریو:**
  سرور `ns1.example.com` نقش Master و سرور `ns2.example.com` نقش Slave را دارد. هر تغییر در `ns1` خودکار به `ns2` منتقل می‌شود (Zone Transfer).

---

### 💡 ۶. Stealth / Hidden Master DNS Server
یک سرور Master است که آدرس IP آن از دید اینترنت و کلاینت‌ها **مخفی** است و در رکوردهای NS دامنه ثبت نمی‌شود. این سرور فقط اطلاعات را به سرورهای Slave داخل DMZ ارسال می‌کند.

* **مثال سناریو:**
  جهت جلوگیری از حملات DDoS و نفوذ هکرها به سرور اصلی مدیریت دامنه‌ها در سازمان‌های حساس.

---

## 3️⃣ بررسی و مقایسه نرم‌افزارهای مختلف DNS در لینوکس

در دنیای لینوکس نرم‌افزارهای مختلفی برای راه‌اندازی DNS وجود دارند که هرکدام برای هدف خاصی طراحی شده‌اند:

---

### 🛠️ ۱. BIND9 (Berkeley Internet Name Domain)
قدیمی‌ترین، استاندارد‌ترین و پرکاربردترین نرم‌افزار DNS در جهان.

* **کاربرد اصلی:** استفاده جامع به عنوان Authoritative، Recursive، Master/Slave و Stealth.
* **مزایا:**
  * پشتیبانی کامل از تمام استانداردهای RFC و قابلیت‌های جدید مثل DNSSEC و TSIG.
  * مستندات بسیار فراوان و جامع در سراسر اینترنت.
  * پایداری بسیار بالا در سناریوهای پیچیده و بزرگ.
* **معایب:**
  * پیچیدگی پیکربندی نسبت به ابزارهای جدیدتر.
  * مصرف منابع بیشتر نسبت به ابزارهای سبک.
* **مثال کاربرد:** سرویس‌دهنده‌های بزرگ اینترنتی، دیتاسنترها و شبکه‌های سازمانی بزرگ.

---

### 🛠️ ۲. Dnsmasq
یک نرم‌افزار بسیار سبک و کم‌حجم که همزمان نقش **DNS Forwarder/Cache** و **DHCP Server** را ایفا می‌کند.

* **کاربرد اصلی:** شبکه‌های کوچک، روترهای خانگی، سیستم‌عامل‌های شخصی و محیط‌های مجازی‌سازی (مثل KVM/Docker).
* **مزایا:**
  * کانفیگ بسیار ساده (همه چیز در یک فایل).
  * مصرف بسیار کم RAM و CPU.
  * یکپارچگی عالی با سرویس DHCP (سیستم‌ها به محض گرفتن IP، نامشان در DNS ثبت می‌شود).
* **معایب:**
  * عدم پشتیبانی مناسب از زون‌های بزرگ و قابلیت‌های پیشرفته مثل Zone Transfer و DNSSEC کامل.
* **مثال کاربرد:** سیستم‌عامل OpenWrt روی روترها، یا شبکه داخلی لپ‌تاپ‌های توسعه‌دهندگان.

---

### 🛠️ ۳. Unbound
یک سرور **Recursive و Caching DNS** بسیار مدرن، سریع و ایمن.

* **کاربرد اصلی:** فقط به عنوان Resolver / Caching DNS Server.
* **مزایا:**
  * سرعت و عملکرد بی‌نظیر در کش کردن.
  * امنیت فوق‌العاده بالا و طراحی شده بر پایه اصول مدرن امنیت (پیش‌فرض با حمایت از DNSSEC).
  * کد ساده‌تر نسبت به BIND که احتمال وجود باگ‌های امنیتی را کاهش می‌دهد.
* **معایب:**
  * نمی‌توان از آن به عنوان سرور Authoritative برای میزبانی دامنه‌ها استفاده کرد.
* **مثال کاربرد:** استفاده به عنوان Resolver اصلی در پروژه‌هایی مثل Pi-hole یا DNS داخلی سازمان‌ها جهت افزایش سرعت اینترنت.

---

### 🛠️ ۴. PowerDNS (pdns)
یک DNS Server فوق‌العاده حرفه‌ای و مدرن که معماری آن بر پایه اتصال به دیتابیس‌ها (MySQL, PostgreSQL, LDAP) بنا شده است.

* **کاربرد اصلی:** Authoritative DNS Server برای حجم داده‌های بالا و پنل‌های وب.
* **مزایا:**
  * ذخیره‌سازی اطلاعات زون‌ها در دیتابیس به جای فایل متنی.
  * قابلیت اتصال به پنل‌های مدیریتی تحت وب (Web GUI).
  * سرعت بالا و امکان تغییر لحظه‌ای رکوردها بدون نیاز به Reload کردن سرویس.
* **معایب:**
  * نیاز به راه اندازی و مدیریت دیتابیس مجزا.
* **مثال کاربرد:** شرکت‌های ارائه دهنده خدمات هاستینگ و ثبت دامنه که هزاران دامنه را مدیریت می‌کنند.

---

### 🛠️ ۵. Knot DNS / NSD
سرورهای اختصاصی و بسیار سریع از نوع **Authoritative-only**.

* **مزایا:** کارایی عالی (High Performance)، پاسخ‌دهی به میلیون‌ها درخواست در ثانیه با کمترین تاخیر.
* **مثال کاربرد:** استفاده در سرورهای ریشه اینترنت (Root DNS Servers) و پسوندهای ملی (TLDها مثل `.ir`).

---

## 📊 جدول مقایسه سریع نرم‌افزارها

| نرم‌افزار | نوع سرور (Role) | میزان پیچیدگی | منبع ذخیره‌سازی داده | مناسب برای |
| :--- | :--- | :--- | :--- | :--- |
| **BIND9** | همه کاره (All-in-One) | متوسط تا بالا | فایل‌های متنی (Text Files) | پروژه های سازمانی، آموزشی و جامع |
| **Dnsmasq** | Cache / Forwarder + DHCP | بسیار ساده | فایل کانفیگ + `/etc/hosts` | روترها، شبکه‌های کوچک و خانگی |
| **Unbound** | Recursive / Cache | ساده تا متوسط | حافظه (RAM Cache) | سرور حل‌کننده امن و سریع |
| **PowerDNS** | Authoritative / Recursor | متوسط | دیتابیس (SQL / LDAP) | هاستینگ‌ها و سیستم‌های تحت وب |

---

## 4️⃣ معیارهای انتخاب بهترین نرم‌افزار برای پروژه سازمان

برای انتخاب نرم‌افزار مناسب در پروژه لینوکس خود، کافیست سوالات زیر را بپرسید:

1. **آیا می‌خواهید دامنه‌های اختصاصی سازمان را روی اینترنت یا شبکه داخلی میزبانی کنید؟**
   * 👈 **انتخاب:** **BIND9** یا **PowerDNS**
2. **آیا هدف فقط افزایش سرعت اینترنت کاربران با کش کردن و بستن تبلیغات است؟**
   * 👈 **انتخاب:** **Unbound**
3. **آیا یک شبکه کوچک یا تست روی لپ‌تاپ/روتر دارید و DHCP هم می‌خواهید؟**
   * 👈 **انتخاب:** **Dnsmasq**
4. **آیا به دنبال یادگیری استانداردترین و کامل‌ترین ابزار DNS در مدرک‌های بین‌المللی (مثل LPIC-2) هستید؟**
   * 👈 **انتخاب قطعی:** **BIND9**

---

## 🎯 جمع‌بندی
در این داکیومنت با انواع نقش‌های سرورهای DNS از جمله Authoritative، Recursive، Master/Slave و Stealth آشنا شدیم و نرم‌افزارهای مطرح دنیای لینوکس را مقایسه کردیم. برای اکثر پروژه‌های استاندارد و جامع لینوکسی، **BIND9** به دلیل پشتیبانی کامل از تمامی این نقش‌ها، بهترین گزینه برای یادگیری و پیاده‌سازی است.

### 5️⃣ پیاده‌سازی عملی Master Node و Slave Node با BIND9

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

### 6 تست، عیب‌یابی و بررسی صحت عملکرد

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
