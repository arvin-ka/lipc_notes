# 🔗 Step-by-Step Guide: Configuring Network Bonding (Active-Backup Mode) on Debian

پروژه پیش‌رو نحوه ادغام و پیکربندی دو کارت شبکه فیزیکی مجزا را در سیستم‌عامل Debian با استفاده از ماژول **Bonding** و در مد **Active-Backup (Mode 1)** آموزش می‌دهد. این معماری پایدار، دسترسی‌پذیری بالا (High Availability) و مقاومت در برابر قطع شدن لینک فیزیکی (Failover) را برای سرورهای حساس تضمین می‌کند.

---

## 📌 مفاهیم و معماری سناریو (Architecture Overview)

در حالت **Active-Backup (Mode 1)**:
- فقط **یک کارت شبکه** (کارت شبکه اصلی / Primary) در هر لحظه فعال است و تمام ترافیک شبکه از آن عبور می‌کند.
- کارت شبکه دوم در حالت **آماده‌به‌کار (Standby)** قرار دارد.
- در صورتی که لینک کارت شبکه اصلی قطع شود یا بسوزد، کارت شبکه دوم **بدون قطعی ارتباط و بدون تغییر IP** بلافاصله وارد مدار می‌شود.

```text
               ┌────────────────────────┐
               │    Bonding Interface   │
               │   (bond0 / 192.168.1.50)
               └───────────┬────────────┘
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
    ┌─────────────────┐         ┌─────────────────┐
    │ eth0 (Primary)  │         │ eth1 (Standby)  │
    │     ACTIVE      │         │     BACKUP      │
    └────────┬────────┘         └────────┬────────┘
             │                           │
  [ Normal Operations ]         [ Cable Unplugged / Fail ]
  (Transmits Traffic)           (Takes Over Instantly)
```

### 📋 پیش‌نیازها (Prerequisites)


* سیستم‌عامل Debian (تست شده روی Debian 11 / 12)
* دسترسی به کاربر با مجوزهای ``sudo``
* دو کارت شبکه فیزیکی یا مجازی (در این سناریو ``eth0`` و ``eth1``)
* بسته ``ifenslave`` جهت مدیریت Bonding در دبیان

 ### 🚀 مراحل اجرای پروژه (Step-by-Step Implementation)

 ### 1️⃣ گام اول: نصب ابزارها و بارگذاری ماژول Bonding

 ابتدا پکیج ``ifenslave`` را نصب کنید تا ابزار اتصال کارت‌های شبکه به اینترفیس Bonding فراهم شود:

```
sudo apt update && sudo apt install ifenslave -y
```

بررسی و فعال‌سازی ماژول کرنی لینوکس برای Bonding:

``sudo modprobe bonding``

### 2️⃣ گام دوم: شناسایی کارت‌های شبکه موجود

آدرس و نام دقیق کارت‌های شبکه متصل به سیستم را بررسی کنید:

``ip link show``

(مطمئن شوید که نام دو اینترفیس مورد نظر را یادداشت کرده‌اید؛ مثلاً ``eth0`` و ``eth1`` یا ``enp0s3`` و ``enp0s8``).

### 3️⃣ گام سوم: پیکربندی فایل ``/etc/network/interfaces``

فایل اصلی تنظیمات شبکه دبیان را ویرایش کنید:

``sudo nano /etc/network/interfaces``

تنظیمات زیر را به انتهای فایل اضافه کرده یا با تنظیمات قبلی جایگزین کنید:

```
# loopback interface
auto lo
iface lo inet loopback

# Primary Network Interface (Slave 1)
allow-hotplug eth0
iface eth0 inet manual
    bond-master bond0

# Secondary Network Interface (Slave 2)
allow-hotplug eth1
iface eth1 inet manual
    bond-master bond0

# Bonding Interface (Master)
auto bond0
iface bond0 inet static
    address 192.168.1.50
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 1.1.1.1
    bond-slaves eth0 eth1
    bond-mode active-backup
    bond-miimon 100
    bond-primary eth0
```

### 🔐 توضیح متغیرهای کلیدی Bonding:

* ا ``bond-slaves eth0 eth1``: کارت‌های شبکه‌ای که زیرمجموعه این Bond می‌شوند.
* ا ``bond-mode active-backup``: تعیین مد کاری پروژه روی حالت Active-Backup (Mode 1).
* ا ``bond-miimon 100``: بررسی سلامت لینک فیزیکی کارت‌ها در هر 100 میلی‌ثانیه (MII Monitoring).
* ا ``bond-primary eth0``: تعیین کارت شبکه eth0 به عنوان اولویت اول و اصلی.

* ### 4️⃣ گام چهارم: راه‌اندازی و اعمال تغییرات

* سرویس شبکه دبیان را ریستارت کنید یا سیستم را یک‌بار Reboot نمایید تا تنظیمات اعمال شوند:

 ``sudo systemctl restart networking``

(یا خاموش و روشن کردن دستی اینترفیس‌ها):

```
sudo ifdown eth0 eth1 bond0
sudo ifup bond0
```

### 🧪 اعتبارسنجی و تست صحت عملکرد (Validation & Testing)

1. بررسی وضعیت زنده اینترفیس Bond
برای مشاهده گزارش کاملی از وضعیت اینترفیس bond0 و شناسایی کارت Active و Backup دستور زیر را بزنید:

``cat /proc/net/bonding/bond0``

خروجی مورد انتظار:

```
Ethernet Channel Bonding Driver: v6.x

Bonding Mode: fault-tolerance (active-backup)
Primary Slave: eth0 (currently active)
Currently Active Slave: eth0
MII Status: up
MII Polling Interval (ms): 100

Slave Interface: eth0
MII Status: up
Speed: 1000 Mbps

Slave Interface: eth1
MII Status: up
Speed: 1000 Mbps
```

2. شبیه‌سازی قطع ارتباط (Failover Test)
برای اطمینان از کارکرد صحیح مد Active-Backup، مراحل زیر را طی کنید:

1. دستور ping مداوم: روی یک سیستم دیگر، آی‌پی 192.168.1.50 را پینگ کنید:

``ping 192.168.1.50``

2. غیرفعال کردن کارت شبکه اصلی (eth0):

``sudo ip link set eth0 down``

3. مشاهده نتیجه:

* متوجه خواهید شد که پینگ سیستم **بدون قطع شدن** به کار خود ادامه می‌دهد.
* با زدن ``cat /proc/net/bonding/bond0`` مشاهده می‌کنید که ``Currently Active Slave`` بلافاصله به ``eth1`` تغییر یافته است.

4. بازگرداندن کارت اصلی به مدار:

``sudo ip link set eth0 up``

(به دلیل تنظیم ``bond-primary eth0``، ترافیک مجدداً و به‌صورت خودکار به ``eth0`` منتقل خواهد شد).

### 🎯 مزایای استفاده از Network Bonding در این پروژه

* ا  High Availability (پایداری بالا): جلوگیری از قطعی سرور در اثر قطعی کابل، خرابی سوئیچ یا سوختن کارت شبکه.
* شفافیت برای لایه‌های بالاتر (Transparency): تمام سرویس‌ها و برنامه‌ها فقط با یک IP ثابت (``bond0``) کار می‌کنند و درگیر تغییرات لایه فیزیکی نمی‌شوند.
