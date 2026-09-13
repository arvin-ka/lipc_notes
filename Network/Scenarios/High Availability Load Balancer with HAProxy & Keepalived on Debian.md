# 🔀 Step-by-Step Guide: High Availability Load Balancer with HAProxy & Keepalived on Debian

این پروژه راهنمای جامع راه‌اندازی یک زیرساخت تقسیم بار (Load Balancing) فوق‌العاده پایدار و بدون قطعی (**Zero-Downtime High Availability**) در لایه‌های ۴ و ۷ شبکه با استفاده از ابزارهای **HAProxy** و **Keepalived (پروتکل VRRP)** روی توزیع دبیان است.

در این سناریو، ترافیک ورودی کاربران بین چند وب‌سرور خلفی (Backend Web Servers) تقسیم می‌شود و در صورت از دست رفتن هر یک از لودبالانسرها یا وب‌سرورها، سیستم به‌صورت خودکار و بدون بروز قطعی (Failover) ترافیک را هدایت می‌کند.

---

## 📌 مفاهیم و معماری سناریو (Architecture Overview)

در این معماری:
1. **Virtual IP (VIP):** یک آدرس IP مجازی (`192.168.1.100`) تعریف می‌شود که همیشه روی لودبالانسر فعال (Master) قرار دارد.
2. **Keepalived:** سلامت لودبالانسرها را مانیتور می‌کند و با پروتکل VRRP در صورت بروز خطا روی سرور Master، آدرس VIP را بلافاصله به سرور Backup منتقل می‌کند.
3. **HAProxy:** ترافیک ورودی روی VIP را گرفته و بر اساس الگوریتم Round-Robin بین سرورهای وب (`Web1` و `Web2`) تقسیم می‌کند.

```text
                                  [ Users / Clients ]
                                           │
                                           ▼
                                 Virtual IP (VIP):
                                   192.168.1.100
                                           │
                    ┌──────────────────────┴──────────────────────┐
                    │ (VRRP Heartbeat / Health Check)             │
                    ▼                                             ▼
       ┌────────────────────────┐                    ┌────────────────────────┐
       │   Load Balancer 1      │                    │   Load Balancer 2      │
       │   (LB1 - MASTER)       │                    │   (LB2 - BACKUP)       │
       │   IP: 192.168.1.10     │                    │   IP: 192.168.1.11     │
       │   HAProxy + Keepalived │                    │   HAProxy + Keepalived │
       └───────────┬────────────┘                    └───────────┬────────────┘
                   │                                             │
                   └──────────────────────┬──────────────────────┘
                                          │ (Load Balanced Traffic)
                                          ▼
                         ┌─────────────────────────────────┐
                         │                                 │
                         ▼                                 ▼
             ┌───────────────────────┐         ┌───────────────────────┐
             │     Web Server 1      │         │     Web Server 2      │
             │ (Backend 1 - Nginx)   │         │ (Backend 2 - Nginx)   │
             │    IP: 192.168.1.20   │         │    IP: 192.168.1.21   │
             └───────────────────────┘         └───────────────────────┘
```

### 📋 جدول مشخصات ماشین‌ها (Environment IP Plan)

| نقش ماشین | نام میزبان (Hostname) | آدرس IP فیزیکی | توضیحات |
| :--- | :--- | :--- | :--- |
Virtual IP | ``vip.local`` | ``192.168.1.100`` | آدرس شناور بین LB1 و LB2 |
Load Balancer 1 | ``lb1`` | ``192.168.1.10`` | Keepalived MASTER / HAProxy |
Load Balancer 2 | ``lb2`` | ``192.168.1.11`` | Keepalived BACKUP / HAProxy |
Web Server 1 | ``web1`` | ``192.168.1.20`` | Backend Nginx / Apache |
Web Server 2 | ``web2`` | ``192.168.1.21`` | Backend Nginx / Apache |

### 📋 پیش‌نیازها (Prerequisites)

* ۴ ماشین مجازی یا سرور Debian (Debian 11 / 12)
* دسترسی کاربر با مجوزهای ``sudo`` روی تمامی سرورها
* برقراری ارتباط شبکه‌ای (Ping) بین تمام ماشین‌ها

### 🚀 مراحل اجرا (Step-by-Step Implementation)

### 1️⃣ گام اول: آماده‌سازی وب‌سرورهای Backend (web1 و web2)

روی هر دو سرور ``web1`` و ``web2`` ابزار Nginx را نصب کنید تا صفحات تست متفاوتی را سرو کنند:

``sudo apt update && sudo apt install nginx -y``

تنظیم صفحه اختصاصی روی ``web1``:

``echo "<h1>Welcome to Web Server 1 (192.168.1.20)</h1>" | sudo tee /var/www/html/index.html``

تنظیم صفحه اختصاصی روی ``web2``:

``echo "<h1>Welcome to Web Server 2 (192.168.1.21)</h1>" | sudo tee /var/www/html/index.html``

### 2️⃣ گام دوم: نصب HAProxy و Keepalived (روی ``lb1`` و ``lb2``)

روی هر دو سرور لودبالانسر (``lb1`` و ``lb2``) پکیج‌های زیر را نصب کنید:

``sudo apt update && sudo apt install haproxy keepalived -y``

برای اینکه HAProxy بتواند روی آدرس‌های IP غیرمحلی (مثل آدرس VIP پیش از فعال شدن) نیز Listen کند، پارامتر زیر را در تنظیمات کرنل فعال کنید:

```
echo "net.ipv4.ip_nonlocal_bind=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 3️⃣ گام سوم: پیکربندی HAProxy (روی ``lb1`` و ``lb2``)

فایل کانفیگ HAProxy را در هر دو سرور لودبالانسر ویرایش کنید:

``sudo nano /etc/haproxy/haproxy.cfg``

محتوای زیر را به انتهای فایل اضافه کنید:

```
# Frontend Configuration (Layer 7 HTTP)
frontend http_front
    bind 192.168.1.100:80
    mode http
    default_backend http_back

# Backend Configuration
backend http_back
    mode http
    balance roundrobin
    option httpchk GET /
    server web1 192.168.1.20:80 check
    server web2 192.168.1.21:80 check

# HAProxy Stats Dashboard Configuration
listen stats
    bind 192.168.1.100:8404
    mode http
    stats enable
    stats uri /
    stats refresh 10s
    stats admin if LOCALHOST
```

سرویس HAProxy را ریستارت کنید:

``sudo systemctl restart haproxy``

### 4️⃣ گام چهارم: پیکربندی Keepalived (پروتکل VRRP)

🔹 پیکربندی روی سرور lb1 (Master):

فایل کانفیگ را ایجاد کنید:

``sudo nano /etc/keepalived/keepalived.conf``

تنظیمات زیر را قرار دهید:

```
vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 2
    weight 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 101
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass Secr3tPass!
    }

    virtual_ipaddress {
        192.168.1.100/24
    }

    track_script {
        check_haproxy
    }
}
```

🔹 پیکربندی روی سرور lb2`` (Backup)``:

فایل کانفیگ را ایجاد کنید:

``sudo nano /etc/keepalived/keepalived.conf``

تنظیمات زیر را قرار دهید:

```
vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 2
    weight 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass Secr3tPass!
    }

    virtual_ipaddress {
        192.168.1.100/24
    }

    track_script {
        check_haproxy
    }
}
```

### 🔐 توضیح متغیرهای Keepalived:

* ا ``state``: تعیین نقش سرور (``MASTER`` یا ``BACKUP``).
* ا ``priority``: اولویت سرور (سروری که priority بالاتری دارد VIP را در دست می‌گیرد).
* ا ``virtual_router_id``: شناسه گروه VRRP (باید روی هر دو سرور یکسان باشد).
* ا ``check_haproxy``: اسکریپت تست سلامت که در صورت کرش کردن HAProxy، اولویت سرور را کاهش می‌دهد تا Failover رخ دهد.

### 5️⃣ گام پنجم: فعال‌سازی سرویس Keepalived

روی هر دو سرور ``lb1`` و ``lb2`` سرویس را فعال و استارت کنید:

``sudo systemctl enable --now keepalived``

### 🧪 اعتبارسنجی و تست صحت عملکرد (Validation & Testing)

1.  بررسی اختصاص آدرس VIP

 روی سرور ``lb1`` دستور زیر را اجرا کنید تا مطمئن شوید IP مجازی ``192.168.1.100`` روی اینترفیس شبکه متصل شده است:

``ip addr show eth0``

 2. تست تقسیم بار (Load Balancing Test)
  
   از یک سیستم کلاینت، آدرس Virtual IP را به صورت مداوم فراخوانی کنید:

   ``while true; do curl [http://192.168.1.100](http://192.168.1.100); sleep 1; done``

   خروجی مورد انتظار:

   پاسخ‌ها به صورت تناوبی (Round-Robin) بین ``Web Server 1`` و ``Web Server 2`` جابه‌جا می‌شوند:

```
<h1>Welcome to Web Server 1 (192.168.1.20)</h1>
<h1>Welcome to Web Server 2 (192.168.1.21)</h1>
<h1>Welcome to Web Server 1 (192.168.1.20)</h1>
...
```

3.  تست پایداری و انتقال لحظه‌ای (Failover Test)

شبیه‌سازی قطعی لودبالانسر اصلی (``lb1``):

سرویس Keepalived یا خود ماشین ``lb1`` را خاموش کنید:

``sudo systemctl stop keepalived``

بررسی جابه‌جایی VIP:

در کمتر از ۱ ثانیه، آدرس ``192.168.1.100``به سرور ``lb2`` منتقل می‌شود.

بررسی ادامه سرویس‌دهی:

دستور ``curl http://192.168.1.100`` روی کلاینت بدون کوچک‌ترین قطعی به کار خود ادامه می‌دهد.

4. مشاهده داشبورد مانیتورینگ HAProxy

 برای مشاهده وضعیت زنده سرورها و میزان ترافیک، مرورگر خود را باز کرده و به آدرس زیر بروید:

``http://192.168.1.100:8404``

 ### 🎯 مزایای این معماری در محیط‌های صنعتی (Production)

 1.  ا ``High Availability`` کامل: حذف Single Point of Failure (SPOF) در لایه لودبالانسر و وب‌سرورها.
 2.  پایش مداوم سلامت (Health Checking): خارج کردن اتوماتیک وب‌سرورهای آسیب‌دیده از چرخه لودبالانس.
 3.  ا Zero-Downtime Maintenance: امکان آپدیت و سرویس‌دهی سرورها بدون نیاز به خاموش کردن کل سامانه.
