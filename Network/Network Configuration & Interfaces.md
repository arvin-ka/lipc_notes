# 🌐 LPIC-1 Study Guide: Persistent Network Configuration & Interfaces (Objective 109.2)

این داکیومنت پوشش‌دهنده کامل **ماژول 109.2 از آزمون LPIC-1** است که روی نحوه پیکربندی دائمی کارت‌های شبکه (Persistent Configuration)، سرویس‌های مدیریت شبکه در توزیع‌های مختلف لینوکس (Debian/Ubuntu و RHEL/CentOS) و مدیریت نام‌گذاری اینترفیس‌ها تمرکز دارد.

---

## 📌 1. پیکربندی دائمی شبکه در توزیع‌های خانواده Debian / Ubuntu

دستوراتی مانند `ip addr add` پس از ریستارت سیستم پاک می‌شوند. برای حفظ تنظیمات شبکه پس از بوت، باید فایل‌های پیکربندی ویرایش شوند.

### 🔹 روش سنتی (فایل `/etc/network/interfaces`)
در توزیع‌های قدیمی‌تر دبیان و اوبونتو، تمام تنظیمات در این فایل قرار می‌گرفت:

```text
# تنظیم اینترفیس Loopback
auto lo
iface lo inet loopback

# پیکربندی کارت شبکه اصلی به صورت DHCP
auto eth0
iface eth0 inet dhcp

# پیکربندی کارت شبکه به صورت IP استاتیک (Static)
auto eth1
iface eth1 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 1.1.1.1
```

### دستورات مدیریت سرویس:

```
sudo systemctl restart networking
# یا دستورات سنتی:
sudo ifup eth0
sudo ifdown eth0
```

### 🔹 روش جدید اوبونتو (Netplan)

در نسخه 17.10 به بعد اوبونتو، ابزار Netplan جایگزین روش قبلی شد که از ساختار YAML در مسیر ``/etc/netplan/*.yaml`` استفاده می‌کند:

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```

### اعمال تنظیمات Netplan:

``sudo netplan apply``

### 📌 2. پیکربندی دائمی شبکه در توزیع‌های خانواده RHEL / CentOS / Fedora

در توزیع‌های مبتنی بر رد هت، پیکربندی هر اینترفیس در یک فایل جداگانه در مسیر ``/etc/sysconfig/network-scripts/`` ذخیره می‌شود.

### 🔹 فایل‌های ifcfg-<interface_name>

برای مثال برای کارت شبکه ``eth0`` فایل ``ifcfg-eth0`` ویرایش می‌شود:

### ۱. نمونه تنظیمات حالت DHCP:

```
DEVICE=eth0
BOOTPROTO=dhcp
ONBOOT=yes
```

### ۲. نمونه تنظیمات حالت Static:

```
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=1.1.1.1
```

**توضیح کلیدهای مهم:**

* ``ONBOOT=yes``: فعال‌سازی خودکار کارت شبکه هنگام بوت سیستم.
* ``BOOTPROTO``: تعیین حالت آدرس‌دهی (``dhcp`` یا ``none``/``static``).

### دستورات مدیریت سرویس در RHEL:

```
# در نسخه‌های قدیمی‌تر:
sudo systemctl restart network

# در نسخه‌های جدیدتر (NetworkManager):
sudo nmcli connection reload
sudo nmcli connection up eth0
```

### 🏷️ 3. نام‌گذاری کارت‌های شبکه (Consistent Network Device Naming)

در گذشته کارت‌های شبکه بر اساس ترتیب شناسایی توسط کرنی نام‌گذاری می‌شدند (``eth0``, ``eth1``, ``wlan0``). در سیستم‌های مدرن با ابزار udev و systemd، نام‌گذاری بر اساس موقعیت فیزیکی یا سخت‌افزاری انجام می‌شود تا نام کارت‌ها تغییر نکند.

### 🔹 ساختار پیشوندهای جدید:

* ``en``: کارت شبکه سیمی (Ethernet)
* ``wl``: کارت شبکه بی‌سیم (WLAN)
* ``ww``: کارت شبکه پهن‌باند سیار (WWAN)

### 🔹 پسوندهای موقعیت مکانی:

* ``o<index>``: کارت شبکه onboard روی مادربورد (مثال: ``eno1``)
* ``s<slot>``: اسلات PCI Express (مثال: ``ens3``)
* ``p<bus>s<slot>``: اسلات روی باس مشخص (مثال: ``enp2s0``)

### 📂 4. فایل‌های کلیدی سیستم در لایه شبکه

طبق سرفصل کتاب LPIC-1، علاوه بر فایل‌های اینترفیس، شناسايی فایل‌های زیر الزامی است:

1. ا ``/etc/hostname``: نگهداری نام سیستم (Host Name).
2.  ا ``/etc/hosts``: جدول نگاشت محلی آدرس IP به نام دامنه (پیش از مراجعه به DNS).
3.  ا ``/etc/resolv.conf``: مشخص‌کننده سرورهای DNS مورد استفاده سیستم (``nameserver 8.8.8.8``).
4.  ا ``/etc/nsswitch.conf``: فایل تعیین اولویت منابع سرویس‌های نام‌گذاری (``hosts: files dns``).
