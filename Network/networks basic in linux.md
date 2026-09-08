# 🌐 LPIC-1 Study Guide: Fundamentals of Networking & TCP/IP (Objective 109.1)

این داکیومنت پوشش‌دهنده کامل **ماژول 109.1 از آزمون LPIC-1** است که مفاهیم پایه شبکه، مدل‌های ارجاعی OSI و TCP/IP، آدرس‌دهی IP (IPv4 & IPv6)، ماسک شبکه (Subnet Mask)، پورت‌ها و پروتکل‌های اصلی سیستم‌عامل لینوکس را بررسی می‌کند.

---

## 📌 1. مفاهیم پایه شبکه و مدل‌های شبکه (Network Models)

برای درک نحوه انتقال داده‌ها در شبکه، دو مدل مرجع استاندارد وجود دارد:

### 🔹 مدل OSI (هفت لایه‌ای)
کتاب رسمی LPIC-1 تاکید دارد که یک لینوکس‌کار باید لایه‌های مدل OSI را برای عیب‌یابی (Troubleshooting) بشناسد:

1. **Physical Layer (لایه فیزیکی):** انتقال بیت‌ها روی کابل، فیبر نوری یا امواج بی‌سیم.
2. **Data Link Layer (لایه پیوند داده):** آدرس‌دهی فیزیکی (MAC Address) و مدیریت فریم‌ها (Ethernet).
3. **Network Layer (لایه شبکه):** آدرس‌دهی منطقی (IP Address) و مسیریابی (Routing).
4. **Transport Layer (لایه انتقال):** مدیریت ارتباط انتهای به انتها (End-to-End) و کنترل جریان (TCP / UDP).
5. **Session Layer (لایه نشست):** مدیریت، برپایی و خاتمه اتصالات.
6. **Presentation Layer (لایه ارائه):** رمزنگاری، فشرده‌سازی و قالب‌بندی داده‌ها.
7. **Application Layer (لایه کاربرد):** پروتکل‌های مورد استفاده برنامه‌ها (HTTP, SSH, FTP).

### 🔹 مدل TCP/IP (چهار لایه‌ای)
مدل عملیاتی که لینوکس و اینترنت بر پایه آن کار می‌کنند:

| لایه TCP/IP | لایه‌های معادل در OSI | پروتکل‌های مرتبط |
| :--- | :--- | :--- |
| **Link / Network Access** | Physical + Data Link | Ethernet, Wi-Fi, MAC |
| **Internet** | Network | IPv4, IPv6, ICMP, ARP |
| **Transport** | Transport | TCP, UDP |
| **Application** | Session + Presentation + Application | HTTP, HTTPS, SSH, DNS, DHCP |

---

## 🔢 2. آدرس‌دهی IP و ساختار Subnetting

### 🔹 ساختار IPv4
یک آدرس IPv4 از **32 بیت** تشکیل شده که به ۴ بخش ۸ بیتی (Octet) تقسیم می‌شود:
`192.168.1.50`

هر آدرس IP به دو بخش اصلی تقسیم می‌شود:
1. **Network ID:** بخش معرف شبکه
2. **Host ID:** بخش معرف دستگاه خاص در آن شبکه

### 🔹 ماسک شبکه (Subnet Mask) و CIDR Notation
ماسک شبکه تعیین می‌کند کدام بخش از IP مربوط به شبکه و کدام بخش مربوط به Host است:
- **Netmask استاندارد کلاس C:** `255.255.255.0`
- **CIDR Notation:** نمایش تعداد بیت‌های شبکه با اسلش (مثلاً `/24`)

> **مثال LPIC-1:**  
> آدرس `192.168.1.10/24` به این معناست که ۲۴ بیت اول (`192.168.1`) مربوط به Network ID و ۸ بیت آخر (`10`) مربوط به Host ID است.

### 🔹 آدرس‌های خاص در IPv4 (Special & Reserved IPv4 Ranges)
طبق سرفصل کتاب LPIC-1، شناخت رنج‌های زیر الزامی است:
- **Loopback Address:** `127.0.0.1` (اشاره به خود ماشین / Localhost)
- **Private IP Ranges (آدرس‌های خصوصی RFC 1918):**
  - Class A: `10.0.0.0/8`
  - Class B: `172.16.0.0/12`
  - Class C: `192.168.0.0/16`
- **APIPA / Link-Local:** `169.254.0.0/16` (تخصیص خودکار در صورت عدم دریافت IP از DHCP)
- **Broadcast Address:** آخرین آدرس در یک Subnet که داده را به تمام دستگاه‌های شبکه می‌فرستد.

---

## 🌐 3. مقدمه‌ای بر IPv6

با توجه به اتمام آدرس‌های IPv4، استاندارد **IPv6** با طول **128 بیت** معرفی شد:
- نمایش به‌صورت ۱۶ تایی (Hexadecimal) و جداشده با کلون (`:`).
- **مثال:** `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- **خلاصه‌سازی IPv6:** حذف صفر‌های متوالی پیشین و جایگزینی صفر‌های پشت‌سر‌هم با `::` (فقط یک‌بار در هر آدرس مجاز است).
  - شکل خلاصه‌شده آدرس بالا: `2001:db8:85a3::8a2e:370:7334`
- **Loopback در IPv6:** `::1`

---

## 🚚 4. لایه انتقال: مقایسه TCP و UDP

در لایه Transport دو پروتکل اصلی وجود دارد که در لینوکس کاربرد فراوان دارند:

| ویژگی | TCP (Transmission Control Protocol) | UDP (User Datagram Protocol) |
| :--- | :--- | :--- |
| **نوع ارتباط** | Connection-Oriented (نیازمند برپایی اتصال) | Connectionless (بدون تایید اتصال) |
| **قابلیت پایداری** | تضمین تحویل داده (Reliable) | عدم تضمین تحویل (Unreliable) |
| **مکانیزم handshake** | ۳ مرحله‌ای (3-Way Handshake: SYN, SYN-ACK, ACK) | ندارد |
| **سرعت** | کندتر به دلیل کنترل خطا | بسیار سریع و خلوت |
| **کاربردها** | Web (HTTP/S), SSH, FTP, Email | Streaming, DNS queries, VoIP, Gaming |

---

## 🔌 5. پورت‌های معروف (Well-Known Ports)

در لینوکس پورت‌ها عدد ۲ بایتی (از ۰ تا ۶۵۵۳۵) هستند که سرویس‌ها روی آن‌ها گوش به زنگ (Listen) می‌باشند. موارد زیر از پورت‌های مهم آزمون LPIC-1 هستند:

* **Port 21:** FTP (File Transfer Protocol)
* **Port 22:** SSH (Secure Shell) / SFTP
* **Port 23:** Telnet (Unencrypted text)
* **Port 25:** SMTP (Simple Mail Transfer Protocol)
* **Port 53:** DNS (Domain Name System) - هم TCP و هم UDP
* **Port 67/68:** DHCP (Dynamic Host Configuration Protocol)
* **Port 80:** HTTP (Hypertext Transfer Protocol)
* **Port 123:** NTP (Network Time Protocol)
* **Port 143:** IMAP
* **Port 443:** HTTPS (HTTP Secure)

---

## 📄 6. فایل‌های پیکربندی مرتبط در لینوکس

در سیستم‌عامل لینوکس فایل‌های متنی پایه‌ای برای نگهداری این اطلاعات وجود دارد:

### ۱. فایل نگاشت پورت‌ها و سرویس‌ها:
```bash
cat /etc/services
```

۲. فایل نگاشت نام سیستم‌ها به IP (Local Name Resolution):

``cat /etc/hosts``

نمونه محتوا:

```
127.0.0.1   localhost
127.0.1.1   arvin-VMware
192.168.1.5 my-custom-server.local
```

# 🌐 LPIC-1 Study Guide: Linux Network Configuration & Troubleshooting (Part 2)

این داکیومنت پوشش‌دهنده کامل بخش دوم از مباحث شبکه **LPIC-1 (Topic 109)** است که روی ابزارها، دستورات کاربردی عیب‌یابی شبکه، مدیریت اینترفیس‌ها و فایل‌های پیکربندی اصلی در سیستم‌عامل لینوکس تمرکز دارد.

---

## 🛠️ 1. بررسی و مدیریت کارت‌های شبکه (Network Interfaces)

در لینوکس برای مشاهده و مدیریت اینترفیس‌های شبکه از ابزارهای سنتی (Legacy) و مدرن (`iproute2`) استفاده می‌شود.

### 🔹 دستورات کلاسیک در برابر مدرن

| عملکرد | دستور سنتی (Legacy) | دستور استاندارد و جدید (`iproute2`) |
| :--- | :--- | :--- |
| **نمایش وضعیت اینترفیس‌ها** | `ifconfig` | `ip addr show` / `ip a` |
| **فعال کردن کارت شبکه** | `ifconfig eth0 up` | `ip link set eth0 up` |
| **غیرفعال کردن کارت شبکه** | `ifconfig eth0 down` | `ip link set eth0 down` |
| **تخصیص موقت IP** | `ifconfig eth0 192.168.1.10/24` | `ip addr add 192.168.1.10/24 dev eth0` |
| **حذف IP از کارت شبکه** | - | `ip addr del 192.168.1.10/24 dev eth0` |

> **نکته LPIC-1:** ابزارهای سنتی مانند `ifconfig` و `route` منسوخ (Deprecated) شده‌اند، اما همچنان در آزمون LPIC-1 مورد سؤال قرار می‌گیرند. ابزار اصلی مدرن `ip` است.

---

## 🗺️ 2. مدیریت جدول مسیریابی (Routing Table) & Gateway

برای اتصال سیستم به شبکه‌های دیگر یا اینترنت، تعریف **Default Gateway** الزامی است.

### 🔹 مشاهده جدول مسیریابی
- **روش سنتی:**
  ```bash
  route -n
  # یا
  netstat -rn
```
```
**(سویچ ``-n`` باعث می‌شود آدرس‌های IP به‌جای تلاش برای تبدیل به نام، به صورت عددی سریع‌تر نمایش داده شوند).**

### روش جدید:

```
ip route show
# یا به اختصار:
ip r
```

### 🔹 افزودن و حذف Default Gateway

```
# اضافه کردن مسیر پیش‌فرض (Gateway)
sudo ip route add default via 192.168.1.1

# حذف مسیر پیش‌فرض
sudo ip route del default via 192.168.1.1
```

### 🔍 3. ابزارهای عیب‌یابی و تست ارتباطات (Troubleshooting Tools)

کتاب رسمی LPIC-1 ابزارهای زیر را برای تست و عیب‌یابی لایه‌های مختلف شبکه معرفی می‌کند:

### ا 1️⃣ ping (بررسی لایه شبکه / ICMP)

ارسال بسته ICMP Echo Request برای بررسی زنده بودن و پاسخ‌دهی یک میزبان (Host):

```
# ارسال ۴ بسته و توقف
ping -c 4 8.8.8.8
```

### ا 2️⃣ traceroute / tracepath (ردیابی مسیر بسته‌ها)

برای مشاهده تمام روترها و گام‌هایی (Hops) که بسته طی می‌کند تا به مقصد برسد:

```
traceroute google.com
# یا بدون نیاز به دسترسی root:
tracepath google.com
```

### ا 3️⃣ netstat / ss (بررسی سوکت‌ها و پورت‌های باز)

برای مشاهده پورت‌هایی که سیستم روی آن‌ها به درخواست‌ها گوش می‌دهد (Listen) یا اتصالات فعال:

### دستور مدرن و سریع ss:

``ss -tuln``

* ``-t``: اتصالات TCP
* ``-u``: اتصالات UDP
* ``-l``: فقط پورت‌های گوش‌به‌زنگ (Listening)
* ``-n``: نمایش عددی پورت‌ها و IPها
* ``-p``: نمایش نام و PID فرآیند (Process) مربوطه (نیازمند ``sudo``)

### 📂 4. فایل‌های پیکربندی مهم شبکه در لینوکس

طبق سرفصل کتاب LPIC-1، تغییرات اعمال‌شده با دستوراتی مثل ``ip`` یا ``ifconfig`` موقتی هستند (با ریستارت پاک می‌شوند). برای دائمی‌کردن پیکربندی‌ها از فایل‌های زیر استفاده می‌شود:

### 1️⃣ تنظیم نام سیستم: /etc/hostname

این فایل شامل نام سیستم (Hostname) است.

```
# مشاهده نام جاری
cat /etc/hostname

# تغییر نام سیستم به صورت زنده:
sudo hostnamectl set-hostname my-linux-node
```

### 2️⃣ سرویس نام‌گذاری محلی: /etc/hosts

برای نگاشت دستی IP به Name پیش از مراجعه به DNS:

```
127.0.0.1   localhost
192.168.1.50   server1.lab.local server1
```

### 3️⃣ تنظیمات سرورهای DNS: /etc/resolv.conf

آدرس DNS Serverهایی که سیستم برای حل نام دامنه‌ها به IP از آن‌ها استفاده می‌کند در این فایل قرار دارد:

```
nameserver 8.8.8.8
nameserver 1.1.1.1
```

### 4️⃣ اولویت‌بندی حل نام: /etc/nsswitch.conf

تعیین می‌کند لینوکس ابتدا برای تبدیل نام به IP به کجا مراجعه کند (مثلاً ابتدا فایل ``hosts`` یا ابتدا ``DNS``):

``hosts:          files dns``

**(معنا: اول فایل ``/etc/hosts`` را بررسی کن، اگر پیدا نشد به ``DNS`` رجوع کن).**

### ⚡ 5. مدیریت شبکه با NetworkManager (nmcli / nmtui)

در توزیع‌های مدرن، NetworkManager مدیریت شبکه را برعهده دارد.

#### رابط متنی گرافیکی (TUI):

``nmtui``

#### ابزار خط فرمانی (nmcli):

```
# مشاهده وضعیت اتصالات
nmcli connection show

# فعال/غیرفعال کردن یک اتصال
nmcli connection up eth0
nmcli connection down eth0
```
