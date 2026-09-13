# ⚡ Step-by-Step Guide: Network Bonding with Load Balancing on Debian

این داکیومنت شامل راهنمای جامع پیکربندی و راه‌اندازی **Network Bonding با قابلیت تقسیم بار (Load Balancing)** روی توزیع دبیان است. در این سناریو، با ادغام دو کارت شبکه فیزیکی، علاوه بر ایجاد **پایداری در برابر خرابی (Fault Tolerance)**، **پهنای باند کل شبکه نیز افزایش می‌یابد**.

---

## 📌 بررسی مدهای تقسیم بار (Load Balancing Modes)

در ابزار Bonding لینوکس، دو مد اصلی برای تقسیم بار وجود دارد:

1. **Mode 0 - Round-Robin (`balance-rr`):**
   - بسته‌های شبکه به ترتیب و به‌صورت نوبتی از کارت اول (`eth0`) و سپس کارت دوم (`eth1`) ارسال می‌شوند.
   - **مزیت:** افزایش پهنای باند و تقسیم بار متوازن در ارسال و دریافت.
   - **نکته مهم:** این مد به **پیکربندی ویژه روی سوئیچ شبکه (EtherChannel / Link Aggregation)** نیاز دارد.

2. **Mode 6 - Adaptive Load Balancing (`balance-alb`):**
   - تقسیم بار هوشمند در لایه دریافت و ارسال بدون نیاز به سوئیچ‌های گران‌قیمت مدیریتی.
   - **مزیت:** **بدون نیاز به هیچ‌گونه تنظیمات روی سوئیچ شبکه** کار می‌کند.
   - **انتخاب این پروژه:** در این راهنما از **Mode 6** استفاده می‌کنیم چون نیازی به سخت‌افزار خاصی ندارد و تماماً در لایه نرم‌افزاری سیستم‌عامل مدیریت می‌شود.

---

## 🏗️ معماری سناریو (Architecture Overview)

```text
               ┌────────────────────────┐
               │    Bonding Interface   │
               │   (bond0 / 192.168.1.50)
               └───────────┬────────────┘
                           │
             ┌─────────────┴─────────────┐
             │   Adaptive Load Balancing │
             ▼                           ▼
    ┌─────────────────┐         ┌─────────────────┐
    │ eth0 (Slave 1)  │         │ eth1 (Slave 2)  │
    │  Traffic Share  │         │  Traffic Share  │
    └────────┬────────┘         └────────┬────────┘
             │                           │
  [ 50% Outgoing/Incoming ]    [ 50% Outgoing/Incoming ]
             └─────────────┬─────────────┘
                           ▼
                 [ Network / Switch ]
```

### 📋 پیش‌نیازها (Prerequisites)

* سیستم‌عامل Debian (تست شده روی نسخه 11 / 12)
* دسترسی به کاربر با مجوزهای ``sudo``
* دو کارت شبکه فیزیکی یا مجازی (در این سناریو ``eth0`` و ``eth1``)
* بسته ``ifenslave`` جهت اتصال کارت‌های شبکه

### 🚀 مراحل اجرا (Step-by-Step Implementation)

### 1️⃣ گام اول: نصب ابزارها و بارگذاری ماژول Bonding

ابتدا پکیج پیش‌نیاز را نصب و ماژول کرنی را فعال کنید:

```
sudo apt update && sudo apt install ifenslave -y
sudo modprobe bonding
```

### 2️⃣ گام دوم: پیکربندی فایل ``/etc/network/interfaces``

فایل اصلی تنظیمات شبکه را ویرایش کنید:

``sudo nano /etc/network/interfaces``

تنظیمات زیر را به فایل اضافه کنید:

```
# Loopback Interface
auto lo
iface lo inet loopback

# Primary Interface (Slave 1)
allow-hotplug eth0
iface eth0 inet manual
    bond-master bond0

# Secondary Interface (Slave 2)
allow-hotplug eth1
iface eth1 inet manual
    bond-master bond0

# Bonding Interface (Master - Load Balancing)
auto bond0
iface bond0 inet static
    address 192.168.1.50
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 1.1.1.1
    bond-slaves eth0 eth1
    bond-mode balance-alb
    bond-miimon 100
```

### 🔐 توضیح پارامترها:

* ا ``bond-mode balance-alb``: تنظیم پروژه روی Mode 6 (Adaptive Load Balancing) جهت توزیع هوشمند ترافیک ورودی و خروجی.
* ا ``bond-slaves eth0 eth1``: کارت‌های شبکه‌ای که پهنای باند آن‌ها با هم ترکیب می‌شود.
* ا ``bond-miimon 100``: چک کردن وضعیت سلامت لینک‌ها در هر ۱۰۰ میلی‌ثانیه.
### 3️⃣ گام سوم: اعمال تنظیمات و راه‌اندازی

سرویس شبکه را ریستارت کنید:

``sudo systemctl restart networking``

(یا با ریستارت دستی اینترفیس‌ها):

```
sudo ifdown eth0 eth1 bond0
sudo ifup bond0
```

### 🧪 اعتبارسنجی و تست صحت عملکرد (Validation)

1. ۱. بررسی وضعیت فعال بودن Load Balancing
گزارش کرنل لینوکس از اینترفیس ``bond0`` را مشاهده کنید:

``cat /proc/net/bonding/bond0``

**خروجی مورد انتظار:**

```
Ethernet Channel Bonding Driver: v6.x

Bonding Mode: adaptive load balancing (balance-alb)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: eth0
MII Status: up
Speed: 1000 Mbps
Link Failure Count: 0

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
Link Failure Count: 0
```

2. تست تقسیم بار واقعی (Load Distribution Test)

برای تست اینکه آیا پهنای باند و ترافیک واقعاً بین هر دو کارت شبکه تقسیم می‌شود:

1. نصب ابزار مانیتورینگ ترافیک ``iptraf-ng`` یا ``iftop``:

2. ``sudo apt install iftop -y``

2. مشاهده ترافیک روی هر کارت شبکه به‌صورت مجزا در دو ترمینال مختلف:
  
 ترمینال ۱: ``sudo iftop -i eth0``
 
 ترمینال ۲: ``sudo iftop -i eth1``
  
3. تولید ترافیک سنگین (دانلود همزمان چند فایل یا تست با ``iperf3``):

مشاهده خواهید کرد که ترافیک خروجی و ورودی به‌صورت هم‌زمان بین``eth0`` و ``eth1`` تقسیم می‌شود و بار شبکه فقط روی یک کارت نمی‌افتد.

### 📊 مقایسه Active-Backup با Load Balancing

| ویژگی | Active-Backup (Mode 1) | Load Balancing (Mode 6)
| :--- | :--- | :--- |
هدف اصلی | صرفاً پایداری و قطع نشدن (Failover) | پایداری + افزایش پهنای باند کل |
استفاده از کارت دوم | کارت دوم تا زمان خرابی کارت اول بیکار است | هر دو کارت هم‌زمان فعال و در حال انتقال داده هستند |
| نیاز به تنظیمات سوئیچ | ندارد | ندارد (در Mode 6) |
پهنای باند کل | برابر با سرعت ۱ کارت شبکه | مجموع سرعت هر دو کارت شبکه |
