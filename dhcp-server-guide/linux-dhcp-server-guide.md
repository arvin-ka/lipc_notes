# 💻 راهنمای جامع و کاربردی: نصب، راه‌اندازی و پیکربندی DHCP Server و DHCP Relay در لینوکس

این داکیومنت به بررسی مفاهیم پروتکل **DHCP**، فرآیند تخصیص آدرس IP، نحوه نصب و پیکربندی سرویس‌دهنده DHCP روی توزیع‌های **Ubuntu/Debian** و **CentOS/RHEL**، و همچنین راه‌اندازی لینوکس به‌عنوان **DHCP Relay Agent** می‌پردازد.

---

## 📋 فهرست مطالب

1. [مفهوم DHCP Server و نحوه عملکرد آن (فرآیند DORA)](#1-مفهوم-dhcp-server-و-نحوه-عملکرد-آن-فرآیند-dora)
2. [نصب و راه‌اندازی DHCP Server روی Ubuntu / Debian](#2-نصب-و-راهاندازی-dhcp-server-روی-ubuntu--debian)
3. [نصب و راه‌اندازی DHCP Server روی CentOS / RHEL](#3-نصب-و-راهاندازی-dhcp-server-روی-centos--rhel)
4. [پیکربندی سرور لینوکس به‌عنوان DHCP Relay Agent](#4-پیکربندی-سرور-لینوکس-بهعنوان-dhcp-relay-agent)
5. [تست، عیب‌یابی و بررسی لاگ‌های DHCP](#5-تست-عیبیابی-و-بررسی-لاگ‌های-dhcp)

---

## 1️⃣ مفهوم DHCP Server و نحوه عملکرد آن (فرآیند DORA)

### 💡 DHCP چیست؟
سرویس **DHCP** (مخفف **Dynamic Host Configuration Protocol**) یک پروتکل شبکه‌ای روی پورت‌های **UDP 67** (برای سرور) و **UDP 68** (برای کلاینت) است. وظیفه اصلی آن تخصیص خودکار آدرس IP، Subnet Mask، Default Gateway، آدرس‌های DNS Server و سایر پارامترهای شبکه به دستگاه‌های موجود در شبکه است.

---

### 🔄 نحوه عملکرد: مکانیزم DORA

هنگامی که یک کلاینت به شبکه متصل می‌شود و تنظیمات شبکه آن روی حالت **Automatic / DHCP** قرار دارد، یک فرآیند ۴ مرحله‌ای به نام **DORA** طی می‌شود:

```text
[ Client ]                                     [ DHCP Server ]
    |                                                |
    | ------------ 1. DISCOVER (Broadcast) --------->|
    |                                                |
    |<------------ 2. OFFER (Unicast/Broadcast) -----|
    |                                                |
    | ------------ 3. REQUEST (Broadcast) ---------->|
    |                                                |
    |<------------ 4. ACKNOWLEDGEMENT (Unicast) -----|
    |                                                |
```

1. سناسایی (Discover)

* کلاینت به صورت همه‌پخشی (Broadcast با IP مبدأ ``0.0.0.0`` و IP مقصد ``255.255.255.255``) یک بسته درخواست می‌فرستد تا سرورهای DHCP موجود در شبکه را پیدا کند.

2. پیشنهاد (Offer)

* سرور DHCP بسته Discover را دریافت کرده و یک آدرس IP آزاد از رنج تعریف‌شده (Lease) به همراه مشخصاتی مانند Subnet Mask و Gateway به کلاینت پیشنهاد می‌دهد.

3. درخواست (Request)

* کلاینت پیشنهاد دریافت شده را می‌پذیرد و به صورت همه‌پخشی (Broadcast) اعلام می‌کند که این IP پیشنهادی را از این سرور مشخص قبول کرده است (تا سایر سرورهای احتمالی نیز مطلع شوند).

4. تایید نهایی (ACK / Acknowledgment)

* سرور DHCP تأییدیه نهایی را فرستاده، آدرس IP را به نام کلاینت ثبت کرده و مدت زمان اجاره (Leasing Time) را مشخص می‌سازد.

### 2️⃣ نصب و راه‌اندازی DHCP Server روی Ubuntu / Debian

در توزیع‌های مبتنی بر دبیان و اوبونتو، از سرویس دهنده ``isc-dhcp-server`` استفاده می‌شود.

گام اول: تعیین کارت شبکه و تنظیم IP ایستا (Static IP)

سرور DHCP خودش حتماً باید دارای یک آدرس IP ایستا باشد.

### ۱. مشاهده کارت‌های شبکه موجود:

``ip link show``

فرض می‌کنیم نام کارت شبکه ما ``eth0`` یا ``ens33`` است.

### ۲. تنظیم IP ایستا با Netplan (در اوبونتو ۱۸.۰۴ به بعد):

فایل تنظیمات Netplan را ویرایش کنید:

``sudo nano /etc/netplan/01-netcfg.yaml``

تنظیمات زیر را اعمال کنید:

```
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      addresses:
        - 192.168.10.1/24
      routes:
        - to: default
          via: 192.168.10.254
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

اعمال تغییرات:

``sudo netplan apply``

### گام دوم: نصب پکیج ``isc-dhcp-server``

```
sudo apt update
sudo apt install isc-dhcp-server -y
```

### گام سوم: مشخص کردن اینترفیس Listening

فایل زیر را باز کرده و مشخص کنید سرویس روی کدام کارت شبکه گوش دهد:

``sudo nano /etc/default/isc-dhcp-server``

متغیر ``INTERFACESv4`` را به‌صورت زیر مقداردهی کنید:

``INTERFACESv4="ens33"``

### ### گام چهارم: پیکربندی فایل اصلی ``DHCP (dhcpd.conf)``

از فایل اصلی بکاپ بگیرید و آن را ویرایش کنید:

```
sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bak
sudo nano /etc/dhcp/dhcpd.conf
```

محتویات نمونه زیر را اضافه یا جایگزین کنید:

```
# تنظیمات کلی (Global Settings)
option domain-name "lab.local";
option domain-name-servers 8.8.8.8, 1.1.1.1;

default-lease-time 600;     # ۱۰ دقیقه
max-lease-time 7200;        # ۲ ساعت

# مشخص کردن سرور اصلی شبکه (Authoritative)
authoritative;

# تعریف Subnet و رنج آدرس‌دهی (Pool)
subnet 192.168.10.0 netmask 255.255.255.0 {
  range 192.168.10.50 192.168.10.150;        # رنج IP اختصاصی به کلاینت‌ها
  option routers 192.168.10.1;               # آدرس Default Gateway
  option broadcast-address 192.168.10.255;    # آدرس Broadcast
}

# رزرو کردن IP ثابت بر اساس MAC Address (Static Reservation)
host Printer-HP {
  hardware ethernet 00:11:22:33:44:55;
  fixed-address 192.168.10.20;
}
```

### گام پنجم: راه‌اندازی و فعال‌سازی سرویس

```
# تست فایل کانفیگ از نظر خطای Syntax
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf

# استارت و فعال‌سازی سرویس هنگام بوت
sudo systemctl start isc-dhcp-server
sudo systemctl enable isc-dhcp-server

# بررسی وضعیت سرویس
sudo systemctl status isc-dhcp-server
```

### ### 3️⃣ نصب و راه‌اندازی DHCP Server روی CentOS / RHEL

در توزیع‌های CentOS/RHEL 7/8/9 پکیج ``dhcp-server`` (یا ``dhcp``) مورد استفاده قرار می‌گیرد.

### گام اول: تنظیم IP ایستا روی CentOS

با ابزار ``nmcli`` روی کارت شبکه (مثلاً ``eth0`` یا ``ens33``) آدرس ایستا تنظیم کنید:

```
nmcli con mod ens33 ipv4.addresses 172.16.0.1/24
nmcli con mod ens33 ipv4.gateway 172.16.0.254
nmcli con mod ens33 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con mod ens33 ipv4.method manual
nmcli con up ens33
```

### گام دوم: نصب سرویس DHCP

```
# در CentOS 8/9 یا RHEL
sudo dnf install dhcp-server -y

# در CentOS 7
sudo yum install dhcp -y
```

### گام سوم: پیکربندی فایل ``dhcpd.conf``

در سیستم‌های مبتنی بر RedHat، فایل تنظیمات در مسیر ``/etc/dhcp/dhcpd.conf`` قرار دارد:

``sudo nano /etc/dhcp/dhcpd.conf``

نمونه کانفیگ کاربردی:

```
option domain-name "company.lan";
option domain-name-servers 172.16.0.1, 8.8.8.8;

default-lease-time 86400;   # ۲۴ ساعت
max-lease-time 172800;      # ۴۸ ساعت

authoritative;

# تعریف شبکه 172.16.0.0/24
subnet 172.16.0.0 netmask 255.255.255.0 {
    range 172.16.0.100 172.16.0.200;
    option option-121 172.16.0.1;
    option routers 172.16.0.1;
    option broadcast-address 172.16.0.255;
}

# رزرو IP برای سرور فایل (File Server)
host FileServer {
    hardware ethernet 52:54:00:12:34:56;
    fixed-address 172.16.0.10;
}
```

### گام چهارم: تنظیمات فایروال (Firewalld)

در CentOS، فایروال به‌صورت پیش‌فرض پورت‌های DHCP را مسدود می‌کند. باید آن را باز کنید:

```
sudo firewall-cmd --add-service=dhcp --permanent
sudo firewall-cmd --reload
```

گام پنجم: استارت سرویس

```
sudo systemctl start dhcpd
sudo systemctl enable dhcpd
sudo systemctl status dhcpd
```

### 4️⃣ پیکربندی سرور لینوکس به‌عنوان DHCP Relay Agent

### ا 💡 DHCP Relay Agent چیست؟

پیام‌های DHCP Discover به‌صورت Broadcast ارسال می‌شوند و روترها پیام‌های Broadcast را از خود عبور نمی‌دهند. اگر DHCP Server در یک VLAN/Subnet دیگر باشد، کلاینت‌های موجود در VLAN‌های متفاوت نمی‌توانند درخواست‌های خود را به سرور برسانند.

وظیفه DHCP Relay Agent (یا ابزار ``dhcrelay``) دریافت بسته‌های همه‌پخشی (Broadcast) کلاینت‌ها، تبدیل آنها به Unicast و ارسال مستقیم آنها به آدرس DHCP Server اصلی است.

### 🌐 سناریو شبکه:

```
[ Subnet A: 192.168.10.0/24 ]          [ Subnet B: 10.0.0.0/24 ]
[ Clients ] ---> (Broadcast) ---> [ DHCP Relay Agent ] ---> (Unicast) ---> [ DHCP Server ]
                                 (ens33: 192.168.10.254)                    (10.0.0.100)
                                 (ens34: 10.0.0.254)
```

### گام اول: فعال کردن IP Forwarding در لینوکس Relay

روی سروری که قرار است نقش Relay داشته باشد، باید قابلیت مسیردهی (IP Forwarding) را فعال کنید:

```
# فعال‌سازی موقت
sudo sysctl -w net.ipv4.ip_forward=1

# فعال‌سازی دائمی
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### گام دوم: نصب ابزار ``isc-dhcp-relay``

روی Ubuntu/Debian:

```
sudo apt update
sudo apt install isc-dhcp-relay -y
```

روی CentOS/RHEL:

``sudo dnf install dhcp-relay -y``

### گام سوم: کانفیگ DHCP Relay

در هنگام نصب روی اوبونتو، از شما آدرس DHCP Server و اینترفیس‌ها پرسیده می‌شود. برای ویرایش یا کانفیگ دستی:

فایل تنظیمات را باز کنید:

``sudo nano /etc/default/isc-dhcp-relay``

پارامترها را تنظیم کنید:

```
# آدرس IP سرور اصلی DHCP
SERVERS="10.0.0.100"

# اینترفیس‌های ورودی (از سمت کلاینت‌ها) و خروجی (به سمت سرور)
INTERFACES="ens33 ens34"

# آپشن‌های اضافی (اختیاری)
OPTIONS=""
```

### گام چهارم: استارت سرویس Relay

```
sudo systemctl start isc-dhcp-relay
sudo systemctl enable isc-dhcp-relay
sudo systemctl status isc-dhcp-relay
```

نکته مهم: روی DHCP Server اصلی، باید علاوه بر Subnet محلی خود، بلاک subnet مربوط به شبکه کلاینت‌ها (``192.168.10.0/24``) را نیز تعریف کنید تا سرور بداند چه رنجی به درخواست‌های دریافتی از این Relay تخصیص دهد:

```
# تعریف Subnet کلاینت‌های دور (از طریق Relay) روی سرور اصلی
subnet 192.168.10.0 netmask 255.255.255.0 {
  range 192.168.10.100 192.168.10.200;
  option routers 192.168.10.254;
}
```

### 5️⃣ تست، عیب‌یابی و بررسی لاگ‌های DHCP

### ۱. مشاهده لیست IPهای اختصاص داده شده (Leases)

برای مشاهده تمامی آدرس‌های IP که سرور به کلاینت‌ها اجاره داده است:

در Ubuntu / Debian:

``cat /var/lib/dhcp/dhcpd.leases``

در CentOS / RHEL:

``cat /var/lib/dhcpd/dhcpd.leases``

### ۲. پایش زنده لاگ‌های DHCP

هنگام اتصال کلاینت‌ها برای رفع مشکل و دیدن فرآیند DORA، لاگ‌های سیستم را به‌صورت زنده چک کنید:

```
# در سیستم‌های بر پایه systemd
sudo journalctl -u isc-dhcp-server -f -n 50

# یا مشاهده لاگ‌های کلی syslog / messages
sudo tail -f /var/log/syslog | grep dhcpd    # Ubuntu
sudo tail -f /var/log/messages | grep dhcpd  # CentOS
```

### ۳. شنود بسته‌های DHCP با ``tcpdump``

برای اطمینان از دریافت و ارسال بسته‌های UDP روی پورت ۶۷ و ۶۸:

``sudo tcpdump -i ens33 -n port 67 or port 68``


