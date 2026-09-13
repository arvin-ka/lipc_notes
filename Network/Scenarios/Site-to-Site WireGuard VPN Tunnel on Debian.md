# 🛡️ Step-by-Step Guide: Building a Site-to-Site WireGuard VPN Tunnel on Debian

این پروژه راهنمای پیاده‌سازی یک تونل ارتباطی امن و رمزنگاری‌شده **Site-to-Site VPN** با استفاده از پروتکل **WireGuard** در توزیع دبیان است. با این معماری، دو شبکه محلی (LAN) مجزا در دو نقطه جغرافیایی مختلف (مثلاً دفتر تهران و دفتر شیراز یا دو دیتاسنتر مختلف) به گونه‌ای به یکدیگر متصل می‌شوند که سیستم‌های هر دو شبکه بتوانند بدون نیاز به اینترنت عمومی و کاملاً امن با یکدیگر ارتباط برقرار کنند.

---

## 📌 مفاهیم و معماری سناریو (Architecture Overview)

در این سناریو، دو روتر/سرور دبیان نقش GateWay را بازی می‌کنند:
* **Site-A (Gateway 1):** شبکه محلی `192.168.10.0/24` | آی‌پی تونل: `10.0.0.1`
* **Site-B (Gateway 2):** شبکه محلی `192.168.20.0/24` | آی‌پی تونل: `10.0.0.2`

```text
  [ Local Network A ]                                                  [ Local Network B ]
  192.168.10.0/24                                                      192.168.20.0/24
         │                                                                    │
         ▼                                                                    ▼
┌──────────────────┐          ==============================          ┌──────────────────┐
│  Debian GW A     │          │    Encrypted WireGuard     │          │  Debian GW B     │
│  (Site-A)        │◀────────▶│       Tunnel (wg0)         │◀────────▶│  (Site-B)        │
│  IP: 10.0.0.1    │          │  10.0.0.1 ↔ 10.0.0.2       │          │  IP: 10.0.0.2    │
└──────────────────┘          ==============================          └──────────────────┘
   Public IP: A.A.A.A                                                    Public IP: B.B.B.B
```

### 📋 پیش‌نیازها (Prerequisites)

* دو سرور یا ماشین مجازی Debian (Debian 11 / 12) با دسترسی ``sudo``
* آدرس IP عمومی (Public IP) روی حداقل یکی از سرورها (یا پورت فوردواردینگ مناسب)
* پورت 51820 UDP روی دیوار آتش (Firewall) هر دو سمت باز باشد.

### 🚀 مراحل اجرا (Step-by-Step Implementation)

### 1️⃣ گام اول: نصب WireGuard و ابزارهای مورد نیاز (روی هر دو سرور)

روی هر دو سرور (Site-A و Site-B) دستورات زیر را اجرا کنید:

``sudo apt update && sudo apt install wireguard iptables -y``

فعال‌سازی قابلیت IP Forwarding در لینوکس تا سرورها بتوانند ترافیک شبکه‌های محلی را عبور دهند:

```
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### 2️⃣ گام دوم: تولید کلیدهای امنیتی (Public/Private Keys)

ا  WireGuard برای احراز هویت و رمزنگاری از کلیدهای عمومی و خصوصی asymmetric استفاده می‌کند.

روی سرور Site-A:

* روی سرور Site-A:

```
umask 077
wg genkey | tee privatekey-a | wg pubkey > publickey-a
```

* روی سرور Site-B:

```
umask 077
wg genkey | tee privatekey-b | wg pubkey > publickey-b
```

(مقادیر کلیدهای تولید شده را با دستور ``cat privatekey-a`` و ``cat publickey-a`` مشاهده و یادداشت کنید).

فایل تنظیمات اینترفیس ``wg0`` را در سرور Site-A ایجاد کنید:

``sudo nano /etc/wireguard/wg0.conf``

محتوای زیر را قرار دهید (کلیدها و IP عمومی Site-B را جایگزین کنید):

```
[Interface]
PrivateKey = <محتوای_privatekey-a>
Address = 10.0.0.1/30
ListenPort = 51820

# Forwarding Rules for LAN Routing
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <محتوای_publickey-b>
Endpoint = <Public_IP_Site_B>:51820
AllowedIPs = 10.0.0.2/32, 192.168.20.0/24
PersistentKeepalive = 25
```

### 4️⃣ گام چهارم: پیکربندی سرور Site-B

فایل تنظیمات اینترفیس ``wg0`` را در سرور Site-B ایجاد کنید:

``sudo nano /etc/wireguard/wg0.conf``

محتوای زیر را قرار دهید:

```
[Interface]
PrivateKey = <محتوای_privatekey-b>
Address = 10.0.0.2/30
ListenPort = 51820

# Forwarding Rules for LAN Routing
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <محتوای_publickey-a>
Endpoint = <Public_IP_Site_A>:51820
AllowedIPs = 10.0.0.1/32, 192.168.10.0/24
PersistentKeepalive = 25
```

### 🔐 توضیح پارامترهای کلیدی:

* ا ``AllowedIPs``: مشخص می‌کند چه رنج‌های IP حق دارند از درون این تونل عبور کنند (شامل IP تونل طرف مقابل + رنج شبکه محلی LAN طرف مقابل).
* ا ``PersistentKeepalive = 25``: ارسال یک پکت کوچک در هر ۲۵ ثانیه برای زنده نگه داشتن ارتباط از پشت NAT و Firewall.

### 5️⃣ گام پنجم: راه‌اندازی و فعال‌سازی سرویس

 روی هر دو سرور سرویس WireGuard را استارت کرده و آن را طوری تنظیم کنید که با بوت شدن سیستم به‌صورت اتوماتیک بالا بیاید:

``sudo systemctl enable --now wg-quick@wg0``

 ### 🧪 اعتبارسنجی و تست صحت عملکرد (Validation & Testing)

1. بررسی وضعیت تونل WireGuard

برای مشاهده وضعیت اتصال، تبادل کلیدها و میزان ترافیک رد و بدل شده دستور زیر را وارد کنید:

``sudo wg show``

خروجی مورد انتظار:

```
interface: wg0
  public key: <publickey-a>
  listening port: 51820

peer: <publickey-b>
  endpoint: B.B.B.B:51820
  allowed ips: 10.0.0.2/32, 192.168.20.0/24
  latest handshake: 14 seconds ago
  transfer: 1.24 KiB received, 1.45 KiB sent
```

(بررسی کنید که ``latest handshake`` مقدار داشته باشد که نشان‌دهنده موفقیت‌آمیز بودن ارتباط است).

2. تست ارتباط بین دو شبکه (End-to-End Ping)

1. تست آی‌پی تونل: از سرور Site-A آدرس 10.0.0.2 را پینگ کنید:

``ping -c 4 10.0.0.2``

2. تست ارتباط بین دو شبکه محلی (LAN): از یک کلاینت درون شبکه ``192.168.10.0/24`` یک کلاینت یا سیستم در شبکه ``192.168.20.0/24`` را پینگ کنید.

3. ``ping 192.168.20.50``

4. 3. ۳. بنچمارک سرعت و سنجش پهنای باند (Performance Test)
  
   برای ارزیابی سرعت انتقال داده درون تونل رمزشده از ابزار ``iperf3`` استفاده کنید:

   * روی سرور Site-B (حالت Server):

```
sudo apt install iperf3 -y
iperf3 -s
```

* روی سرور Site-A (حالت Client):

```
sudo apt install iperf3 -y
iperf3 -c 10.0.0.2
```

### 🎯 مزایای این معماری

1. امنیت فوق‌العاده بالا: استفاده از الگوریتم‌های مدرن رمزنگاری مانند ChaCha20 و Poly1305.
2. کارایی و سرعت عالی: اجرای مستمر در لایه کرنل لینوکس (Kernel Space) با کمترین میزان مصرف CPU و تاخیر (Latency).
3. شفافیت برای کاربران: سیستم‌های موجود در شبکه تهران بدون نیاز به نصب هیچ نرم‌افزار VPN روی سیستم شخصی خود، به سرورهای شیراز دسترسی دارند.
4. 
