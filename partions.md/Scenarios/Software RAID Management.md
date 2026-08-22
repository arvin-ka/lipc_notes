# 🛠️ Step-by-Step Guide: Configuring Software RAID in Linux using `mdadm`

این داکیومنت شامل سناریوی عملی برای ایجاد، قالب‌بندی (Formatting)، اتصال (Mounting) و مدیریت یک آرایه **Software RAID 1 (Mirroring)** روی لینوکس با استفاده از ابزار `mdadm` است.

---

## 📌 پیش‌نیازها (Prerequisites)
- یک سیستم‌عامل لینوکس (Ubuntu / Debian / RHEL / CentOS)
- دسترسی به کاربر با مجوزهای `sudo`
- حداقل **دو دیسک سخت خام مجزا** (در این سناریو `/dev/sdb` و `/dev/sdc`)

---

## 🚀 مراحل اجرا (Step-by-Step Implementation)

### 1️⃣ نصب ابزار `mdadm`
ابتدا ابزار مدیریت RAID نرم‌افزاری را روی سیستم نصب کنید:

- **در Ubuntu / Debian:**
  ```bash
  sudo apt update && sudo apt install mdadm -y

### در RHEL / CentOS / AlmaLinux:

``sudo dnf install mdadm -y``

### 2️⃣ بررسی و آماده‌سازی دیسک‌ها

لیست دیسک‌های متصل به سیستم را بررسی کنید تا از نام دقیق دیسک‌های خام مطمئن شوید:

``lsblk``

مطمئن شوید هیچ پارتیشن یا سیستم‌فایلی روی /dev/sdb و /dev/sdc وجود ندارد. (در صورت نیاز می‌توانید signature قبلی دیسک‌ها را پاک کنید):

```
sudo wipefs -a /dev/sdb
sudo wipefs -a /dev/sdc
```

### 3️⃣ ساخت آرایه RAID 1

با استفاده از دستور mdadm یک آرایه جدید با نام /dev/md0 ایجاد کنید:

```
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
```

توضیح پارامترها:

* --create /dev/md0: نام دستگاه منطقی جدید RAID.
* --level=1: تعیین سطح RAID (برای RAID 0 عدد 0 و برای RAID 5 عدد 5 قرار دهید).
* --raid-devices=2: تعداد دیسک‌های فیزیکی شرکت‌کننده در آرایه.
* /dev/sdb /dev/sdc: مسیر دیسک‌های خام انتخاب‌شده.

### 4️⃣ بررسی وضعیت همگام‌سازی (Sync Status)

پس از ایجاد، سیستم شروع به همگام‌سازی (Syncing) دیسک‌ها می‌کند. برای مشاهده روند همگام‌سازی و وضعیت آرایه:

``cat /proc/mdstat``

یا با جزئیات بیشتر:

``sudo mdadm --detail /dev/md0``

### 5️⃣ ساخت سیستم‌فایل و Mount کردن RAID

پس از آماده‌سازی آرایه /dev/md0، آن را مانند یک دیسک معمولی فرمت و متصل کنید:

```
# ایجاد سیستم‌فایل ext4
sudo mkfs.ext4 /dev/md0

# ساخت نقطه اتصال (Mount Point)
sudo mkdir -p /mnt/raid1_storage

# متصل کردن آرایه
sudo mount /dev/md0 /mnt/raid1_storage

# بررسی فضای ذخیره‌سازی
df -h /mnt/raid1_storage
```

### 6️⃣ دائمی‌کردن پیکربندی RAID (Permanent Configuration)

برای اینکه آرایه RAID و اتصال آن پس از ریستارت سیستم حفظ شوند، دو اقدام زیر الزامی است:

الف) ذخیره پیکربندی mdadm.conf

``sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf``

در سیستم‌های مبتنی بر سیستم‌عامل‌های Debian/Ubuntu، باید فایل initramfs را بروزرسانی کنید:

``sudo update-initramfs -u``

ب) افزودن به فایل /etc/fstab

ابتدا UUID آرایه ساخته‌شده را پیدا کنید:

``sudo blkid /dev/md0``

سپس فایل /etc/fstab را ویرایش کرده و خط زیر را به انتهای آن اضافه کنید:

``UUID=YOUR_MD0_UUID  /mnt/raid1_storage  ext4  defaults  0  2``

### تست شبیه‌سازی خرابی دیسک (Simulating Disk Failure)

یکی از قابلیت‌های کلیدی RAID 1 ادامه کار سیستم در صورت خرابی یک دیسک است.

### ۱. علامت‌گذاری یک دیسک به عنوان خراب (Fail)

``sudo mdadm --manage /dev/md0 --fail /dev/sdc``

### ۲. بررسی وضعیت آرایه (در حالت Degraded)

``sudo mdadm --detail /dev/md0``

(مشاهده خواهید کرد که وضعیت آرایه degraded شده اما اطلاعات روی /mnt/raid1_storage همچنان در دسترس هستند).

### ۳. جدا کردن دیسک معیوب

``sudo mdadm --manage /dev/md0 --remove /dev/sdc``

### ۴. اضافه کردن دیسک جدید (جایگزین)

``sudo mdadm --manage /dev/md0 --add /dev/sdc``

با اجرای این دستور، بازسازی خودکار (Rebuilding) اطلاعات روی دیسک جدید آغاز می‌شود. روند را می‌توانید با cat /proc/mdstat دنبال کنید.

### 🧹 پاکسازی و حذف کامل RAID (Cleanup & Reset)

اگر می‌خواهید آرایه RAID را کاملاً پاک کنید:

```
# ۱. قطع اتصال دایرکتوری
sudo umount /mnt/raid1_storage

# ۲. متوقف کردن آرایه RAID
sudo mdadm --stop /dev/md0

# ۳. حذف سوپرپلاک دیسک‌ها
sudo mdadm --zero-superblock /dev/sdb
sudo mdadm --zero-superblock /dev/sdc

# ۴. پاک کردن تنظیمات از fstab و mdadm.conf
```

