# 🌐 Comprehensive Guide (Part 2): Caching & Forwarding DNS Servers in Linux

این داکیومنت، بخشی از سلسله راهنماهای عملی DNS در لینوکس است. در این بخش به بررسی مفاهیم تخصصی **Caching DNS Server** و **Forwarding DNS Server**، نحوه کاهش ترافیک شبکه، افزایش سرعت پاسخ‌دهی (Latency Reduction) و کانفیگ عملی آن‌ها با دو ابزار محبوب **BIND9** و **Unbound** می‌پردازیم.

---

## 📋 فهرست مطالب
1. [مفاهیم پایه: Caching و Forwarding در DNS چیست؟](#1-مفاهیم-پایه-caching-و-forwarding-در-dns-چیست)
2. [سناریوی اول: راه اندازی Caching & Forwarding DNS Server با BIND9](#2-سناریوی-اول-راه-اندازی-caching--forwarding-dns-server-با-bind9)
3. [سناریوی دوم: راه اندازی Caching/Forwarding DNS Server سبک و امن با Unbound](#3-سناریوی-دوم-راه-اندازی-cachingforwarding-dns-server-سبک-و-امن-با-unbound)
4. [مفاهیم پیشرفته: مدیریت TTL، Flush کردن کش و Conditional Forwarding](#4-مفاهیم-پیشرفته-مدیریت-ttl-flush-کردن-کش-و-conditional-forwarding)
5. [تست، بنچمارک سرعت و عیب‌یابی (Verification & Performance Test)](#5-تست-بنچمارک-سرعت-و-عیب‌یابی)

---

## 1️⃣ مفاهیم پایه: Caching و Forwarding در DNS چیست؟

قبل از ورود به تنظیمات، درک تفاوت و نحوه تعامل این دو مکانیزم اهمیت زیادی دارد:

### ⚡ Caching DNS Server چیست؟
وقتی یک کلاینت، آدرس یک سایت (مثلاً `google.com`) را درخواست می‌کند، سرور DNS باید تمام درخت سلسله‌مراتبی (Root, TLD, Authoritative) را بپیماید تا پاسخ را پیدا کند. 
یک **Caching DNS Server**:
* پاسخ دریافت شده را بر اساس مقدار **TTL (Time to Live)** در حافظه RAM خود ذخیره (Cache) می‌کند.
* اگر کلاینت دیگری (یا همان کلاینت) مجدداً همان آدرس را درخواست کند، سرور بدون طی کردن مراحل طولانی در اینترنت، **بلافاصله و از حافظه کش خود** پاسخ را در چند میلی‌ثانیه برمی‌گرداند.
* **مزایا:** کاهش شدید ترافیک مصرفی اینترنت، افزایش فوق‌العاده سرعت وب‌گردی کاربران.

### 🔄 Forwarding DNS Server چیست؟
سرور DNS به‌جای اینکه خودش مستقیماً فرآیند **Iterative Resolution** (پرس‌وجو از Root Serverها) را انجام دهد، تمامی درخواست‌های دریافتی از کلاینت‌ها را به یک یا چند سرور DNS بالا‌دستی (Upstream DNS) مانند `1.1.1.1` یا `8.8.8.8` پاس (Forward) می‌دهد.

* **Forwarding خالص:** فقط درخواست‌ها را به سرور دیگر می‌فرستد.
* **Caching Forwarder (ترکیبی):** درخواست را به Upstream می‌فرستد، پاسخ را از آن می‌گیرد، نتیجه را کش می‌کند و به کلاینت تحویل می‌دهد. (این رایج‌ترین و پرکاربردترین حالت در شبکه‌های سازمانی است).

---

### 📊 مقایسه سناریوهای مختلف پاسخ‌دهی:

| خصوصیت | Recursive Resolver (مستقیم) | Forwarding DNS Server | Caching Forwarder |
| :--- | :--- | :--- | :--- |
| **طریقه یافتن IP** | پرسان پرسان از Root و TLDها | ارسال تمام درخواست‌ها به Upstream | ارسال به Upstream + ذخیره در کش |
| **وابستگی به Upstream** | ندارد (مستقیماً با اینترنت در ارتباط است) | کاملاً وابسته به Upstream DNS است | وابسته به Upstream در اولین درخواست |
| **سرعت پاسخ بعدی** | کندتر در بار اول | تابع سرعت Upstream DNS | **فوق‌العاده سریع (زیر ۱ میلی‌ثانیه)** |
| **مصرف پهنای باند** | بیشتر | متوسط | **کمترین میزان ممکن** |

---

## 2️⃣ سناریوی اول: راه اندازی Caching & Forwarding DNS Server با BIND9

در این سناریو، یک سرور BIND9 در شبکه داخلی به عنوان کش‌سرور و فورواردر قرار می‌گیرد.

### 📐 مشخصات سناریو:
* **Server IP:** `192.168.10.15`
* **Internal Subnet:** `192.168.10.0/24`
* **Upstream DNS Servers:** `1.1.1.1` (Cloudflare) و `8.8.8.8` (Google)

---

### گام ۱: نصب BIND9
```bash
sudo apt update
sudo apt install bind9 bind9-utils -y
```

### گام ۲: کانفیگ متمرکز در ``/etc/bind/named.conf.options``

فایل کانفیگ اصلی را ویرایش می‌کنیم:

``sudo nano /etc/bind/named.conf.options``

محتوای کامل و بهینه‌شده:

```
// ۱. تعریف لیست کلاینت‌های مجاز به استفاده از کش سرور (کنترل امنیت)
acl "internal_network" {
    127.0.0.1;
    192.168.10.0/24;
};

options {
    directory "/var/cache/bind";

    // ۲. محدود کردن دسترسی برای جلوگیری از حملات DNS Amplification
    allow-query { internal_network; };
    allow-recursion { internal_network; };

    // ۳. فعال‌سازی قابلیت Recursion برای کلاینت‌های داخلی
    recursion yes;

    // ۴. تعریف Upstream DNS Serverها برای Forwarding
    forwarders {
        1.1.1.1;
        8.8.8.8;
    };

    // تعیین نحوه رفتار Forwarding:
    // 'only': اگر Upstreamها پاسخ ندادند، خودش مستقیماً در اینترنت بگردد؟ خیر! فقط از Forwarder بپرسد.
    // 'first': اول از Forwarder می‌پرسد، اگر پاسخ ندادند خودش مستقیماً به Root Serverها وصل می‌شود.
    forward first;

    // ۵. تنظیمات مدیریت کش (Cache Memory Limits)
    max-cache-size 256M;        // حداکثر حجم رم تخصیص یافته به کش DNS
    max-cache-ttl 86400;        // حداکثر زمان نگهداری کش مثبت (۱ روز)
    max-ncache-ttl 300;         // حداکثر زمان نگهداری کش منفی/پاسخ‌های ناموفق (۵ دقیقه)

    // ۶. امنیت و استانداردهای پروتکل
    dnssec-validation auto;
    auth-nxdomain no;           // رعایت استاندارد RFC1035
    listen-on-v6 { any; };
};
```

### گام ۳: تست کانفیگ و ری‌استارت سرویس

```
# بررسی عدم وجود خطا در دستورات
sudo named-checkconf

# راه‌اندازی مجدد BIND9
sudo systemctl restart bind9
sudo systemctl enable bind9
```

### 3️⃣ سناریوی دوم: راه اندازی Caching/Forwarding DNS Server سبک و امن با Unbound

ا Unbound یک سرویس‌دهنده بسیار پرسرعت، سبک و امن است که اختصاصاً برای Recursive Caching Resolver ساخته شده و بسیار سبک‌تر از BIND9 عمل می‌کند.

### گام ۱: نصب Unbound

```
sudo apt update
sudo apt install unbound unbound-host -y
```

### گام ۲: ایجاد فایل کانفیگ سفارشی ``/etc/unbound/unbound.conf.d/caching-forwarder.conf``

``sudo nano /etc/unbound/unbound.conf.d/caching-forwarder.conf``

محتوای فایل کانفیگ:

```
server:
    verbosity: 1
    interface: 0.0.0.0             # پاسخ‌دهی روی تمام کارت‌های شبکه
    port: 53
    do-ip4: yes
    do-ip6: no                     # اگر در شبکه IPv6 ندارید آن را دیستبل کنید
    do-udp: yes
    do-tcp: yes

    # کنترل دسترسی (Access Control)
    access-control: 127.0.0.0/8 allow
    access-control: 192.168.10.0/24 allow

    # بهینه‌سازی حافظه و عملکرد کش (Performance Tuning)
    num-threads: 2                 # تعداد کورهای CPU اختصاص داده شده
    msg-cache-slabs: 2
    rrset-cache-slabs: 2
    infra-cache-slabs: 2
    key-cache-slabs: 2

    # میزان رم تخصیص داده شده به کش رکوردها
    rrset-cache-size: 100m
    msg-cache-size: 50m

    # تنظیمات TTL کش
    cache-min-ttl: 60              # حداقل زمان کش شدن (ثانیه)
    cache-max-ttl: 86400           # حداکثر زمان کش شدن (ثانیه)

    # مخفی کردن اطلاعات سرور برای امنیت بیشتر
    hide-identity: yes
    hide-version: yes

# تنظیم بخش Forwarding به Upstream DNS
forward-zone:
    name: "."                      # فوروارد کردن تمام درخواست‌های جهان
    forward-addr: 1.1.1.1@53       # Cloudflare
    forward-addr: 9.9.9.9@53       # Quad9
```

### گام ۳: اعمال تنظیمات و راه‌اندازی Unbound

```
# بررسی سلامت فایل کانفیگ
sudo unbound-checkconf

# فعال‌سازی و ری‌استارت سرویس
sudo systemctl restart unbound
sudo systemctl enable unbound
```

### 4️⃣ مفاهیم پیشرفته: مدیریت TTL، Flush کردن کش و Conditional Forwarding

در محیط‌های واقعی و سازمان‌ها، گاهی نیاز به رفتارهای پیشرفته‌تری در DNS Server داریم:

### ا 🔀 ۱. Conditional Forwarding (فوروارد مشروط) چیست؟

فرض کنید سازمان شما علاوه بر درخواست‌های اینترنتی، یک دامین داخلی دیگر دارد (مثلا ``corp.local`` که روی یک سرور دیگر با آدرس ``172.16.0.100`` قرار دارد).

شما می‌خواهید فقط درخواست‌های مربوط به ``corp.local`` به سرور داخلی ارسال شوند و بقیه درخواست‌های اینترنت به ``1.1.1.1`` فوروارد گردند.

### 🔹 پیاده‌سازی Conditional Forwarding در BIND9 (``/etc/bind/named.conf.local``):

```
zone "corp.local" {
    type forward;
    forward only;
    forwarders {
        172.16.0.100;
        172.16.0.101;
    };
};
```

### 🔹 پیاده‌سازی Conditional Forwarding در Unbound:

```
forward-zone:
    name: "corp.local"
    forward-addr: 172.16.0.100
```

### 🧹 ۲. پاک کردن کش (Flush Cache)

اگر آدرس IP یک سایت تغییر کند ولی کش سرور شما همچنان IP قدیمی را برگرداند، باید کش سرور را به‌صورت دستی خالی (Flush) کنید:

در BIND9 (با ابزار ``rndc``):

```
# پاک کردن کل حافظه کش BIND9
sudo rndc flush

# پاک کردن کش فقط برای یک دامنه خاص
sudo rndc flushtree example.com
```

در Unbound (با ابزار ``unbound-control``):

```
# پاک کردن کامل کش
sudo unbound-control flush_zone .

# پاک کردن کش یک دامنه خاص
sudo unbound-control flush_zone example.com
```

### 5️⃣ تست، بنچمارک سرعت و عیب‌یابی (Verification & Performance Test)

برای اینکه اثبات کنیم سرور Caching ما به درستی کار می‌کند و اختلاف سرعت را اندازه‌گیری کنیم، از ابزار ``dig`` استفاده می‌کنیم.

### 🧪 تست اختلاف سرعت (Query Time Test)

دستور زیر را بار اول اجرا می‌کنیم (سرور هنوز این دامنه را کش نکرده است):

``dig @192.168.10.15 wikipedia.org | grep "Query time"``

پاسخ بار اول (نمونه):

``;; Query time: 85 msec``

توضیح: سرور مجبور شد درخواست را به Upstream (``1.1.1.1``) بفرستد و ۸۵ میلی‌ثانیه زمان برد.

بلافاصله همان دستور را بار **دوم** اجرا می‌کنیم:

``dig @192.168.10.15 wikipedia.org | grep "Query time"``

پاسخ بار دوم (نمونه):

``;; Query time: 0 msec``

توضیح: پاسخ در ``0 msec`` (زیر یک میلی‌ثانیه) داده شد! این نشان می‌دهد که پاسخ مستقیماً از رم (RAM Cache) سرور محلی خوانده شده و Caching DNS Server به درستی کار می‌کند.

### 🛠️ عیب‌یابی مشکلات رایج (Troubleshooting)

| مشکل / خطا | علت احتمالی | راه حل پیشنهادی |
| :--- | :--- | :--- |
| ``connection timed out`` | مسدود بودن پورت 53 توسط فایروال | باز کردن پورت با ``sudo ufw allow 53/udp`` یا ``firewalld`` |
| ``REFUSED`` در پاسخ dig | محدودیت acl در ``allow-query`` یا ``allow-recursion`` | اضافه کردن IP کلاینت به ACL موجود در کانفیگ سرور |
| ** عدم بروزرسانی IP سایت‌ها** | بالا بودن مقدار TTL در کش سرور | خالی کردن کش با دستور ``rndc flush`` یا کاهش ``max-cache-ttl`` | 
| عدم پاسخ‌دهی سرویس | اشتباه در ساختار فایل‌های کانفیگ | بررسی با ``named-checkconf`` یا ``unbound-checkconf`` |

### 🎯 جمع‌بندی

در این داکیومنت، با مکانیزم‌های Caching و Forwarding به عنوان دو روش اصلی جهت بهبود سرعت و مدیریت ترافیک شبکه آشنا شدید. همچنین پیاده‌سازی این دو قابلیت را در هر دو سرویس‌دهنده BIND9 و Unbound همراه با تکنیک‌های پیشرفته‌ای نظیر Conditional Forwarding و مدیریت کش به‌صورت کامل و عملی بررسی کردیم.
