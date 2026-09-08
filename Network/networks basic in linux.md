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

