# 🚀 Hybrid Scenario: Implementing LVM on top of Software RAID (LVM-on-RAID)

در پروژه‌ها و محیط‌های عملیاتی واقعی (Enterprise)، برای دستیابی هم‌زمان به **سخت‌افزاری پایدار و مقاوم در برابر خرابی (Fault Tolerance)** و **مدیریت پویای فضای ذخیره‌سازی (Dynamic Storage Management)**، معماری **LVM روی RAID** پیاده‌سازی می‌شود.

---

## 🏗️ معماری سناریو (Architecture Overview)

در این معماری، لایه‌ها به صورت زیر روی یکدیگر قرار می‌گیرند:

```text
  [ Physical Disk 1 ]     [ Physical Disk 2 ]
  (/dev/sdb - 20G)        (/dev/sdc - 20G)
          │                       │
          └───────────┬───────────┘
                      ▼
            [ Software RAID 1 ]          <-- Redundancy Layer (mdadm)
               (/dev/md0)
                      │
                      ▼
          [ Physical Volume (PV) ]       <-- LVM Layer Starts
               (/dev/md0)
                      │
                      ▼
           [ Volume Group (VG) ]
               (vg_hybrid)
                      │
          ┌───────────┴───────────┐
          ▼                       ▼
  [ Logical Volume 1 ]   [ Logical Volume 2 ]
    (lv_production)          (lv_logs)
```

### 📌 پیش‌نیازها (Prerequisites)

1. سیستم‌عامل لینوکس (Ubuntu / Debian / CentOS)
2. دو عدد دیسک سخت خام هم‌اندازه (در این سناریو ``/dev/sdb`` و ``/dev/sdc``)
3. ابزارهای ``mdadm`` و ``lvm2``

### 🛠️ مراحل پیاده‌سازی گام‌به‌گام (Step-by-Step Implementation)

### 1️⃣ گام اول: نصب ابزارهای مورد نیاز

```sudo apt update
sudo apt install mdadm lvm2 -y
```

### 2️⃣ گام دوم: آماده‌سازی دیسک‌ها و ساخت آرایه RAID 1

ابتدا دیسک‌ها را پاکسازی کرده و آرایه RAID 1 متناظر را روی آن‌ها ایجاد کنید:

```# پاکسازی امضاهای قبلی دیسک‌ها
sudo wipefs -a /dev/sdb /dev/sdc

# ساخت آرایه RAID 1 با نام /dev/md0
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
```

بررسی وضعیت آرایه:

``cat /proc/mdstat``

### 3️⃣ گام سوم: راه‌اندازی لایه LVM روی دستگاه RAID (/dev/md0)

حالا به جای استفاده مستقیم از دیسک‌های فیزیکی، کل دستگاه RAID (``/dev/md0``) را به عنوان Physical Volume (PV) تعریف می‌کنیم:

الف) ساخت Physical Volume روی RAID:

``sudo pvcreate /dev/md0``

ب) ساخت Volume Group روی PV جدید:

``sudo vgcreate vg_hybrid /dev/md0``

ج) ساخت Logical Volumeهای دلخواه از VG:

```# ساخت LV اول برای داده‌های اصلی به حجم ۱۰ گیگابایت
sudo lvcreate -L 10G -n lv_production vg_hybrid

# ساخت LV دوم با استفاده از کل فضای باقی‌مانده برای Logها
sudo lvcreate -l 100%FREE -n lv_logs vg_hybrid
```

### 4️⃣ گام چهارم: فرمت و متصل کردن (Mount)

پارتیشن‌های منطقی ساخته‌شده را فرمت کرده و به ساختار سیستم متصل کنید:

```
# ۱. فرمت سیستم‌فایل ext4
sudo mkfs.ext4 /dev/vg_hybrid/lv_production
sudo mkfs.ext4 /dev/vg_hybrid/lv_logs

# ۲. ساخت مسیرهای Mount
sudo mkdir -p /mnt/production
sudo mkdir -p /mnt/logs

# ۳. اتصال LVMها
sudo mount /dev/vg_hybrid/lv_production /mnt/production
sudo mount /dev/vg_hybrid/lv_logs /mnt/logs
```

### 5️⃣ گام پنجم: بررسی و اعتبارسنجی نهایی (Validation)

با اجرای دستور زیر، کل ساختار درختی LVM که روی آرایه RAID متکی است را مشاهده کنید:

``lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINTS``

خروجی مورد انتظار:

```
NAME                     SIZE FSTYPE            TYPE  MOUNTPOINTS
sdb                       20G linux_raid_member disk  
└─md0                     20G LVM2_member       raid1 
  ├─vg_hybrid-lv_production
  │                       10G ext4              lvm   /mnt/production
  └─vg_hybrid-lv_logs     10G ext4              lvm   /mnt/logs
sdc                       20G linux_raid_member disk  
└─md0                     20G LVM2_member       raid1 
  ├─vg_hybrid-lv_production
  │                       10G ext4              lvm   /mnt/production
  └─vg_hybrid-lv_logs     10G ext4              lvm   /mnt/logs
```

### ⚡ مزایای این معماری ترکیبی (Key Benefits)

1. اول Redundancy (امنیت داده): اگر یکی از دیسک‌های فیزیکی (``sdb`` یا ``sdc``) کاملاً بسوزد، RAID 1 مانع از قطع شدن سیستم و از دست رفتن داده‌ها می‌شود.
2. دوم Flexibility (انعطاف‌پذیری LVM): می‌توانید آنلاین حجم ``lv_production`` را افزایش دهید، اسنپ‌شات بگیرید یا LV جدید بسازید.
3. سوم Scalability (توسعه‌پذیری ساده): در آینده می‌توانید ۲ دیسک جدید را RAID کنید و با ``vgextend`` آن را به استخر ``vg_hybrid`` اضافه کنید تا فضای کل سیستم افزایش یابد.
