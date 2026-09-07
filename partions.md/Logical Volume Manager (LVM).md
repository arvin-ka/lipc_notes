# 🗄️ Comprehensive Guide: Logical Volume Manager (LVM) in Linux

مدیریت فضای ذخیره‌سازی با **LVM (Logical Volume Manager)** به مدیران سیستم اجازه می‌دهد تا فضای دیسک سخت را به‌صورت پویا، انعطاف‌پذیر و بدون نیاز به خاموش کردن سیستم (Hot-Resizing) تغییر سایز دهند و پارتیشن‌ها را مدیریت کنند.

---

## 📌 مفاهیم و معماری LVM (LVM Architecture)

ساختار LVM از ۳ لایه اصلی تشکیل شده است:

```text
  [ Physical Disk 1 ]     [ Physical Disk 2 ]
          │                       │
          ▼                       ▼
  ┌───────────────┐       ┌───────────────┐
  │ Physical Vol  │       │ Physical Vol  │  (PV)
  │  (/dev/sdb)   │       │  (/dev/sdc)   │
  └───────┬───────┘       └───────┬───────┘
          └───────────┬───────────┘
                      ▼
          ┌───────────────────────┐
          │     Volume Group      │          (VG)
          │      (vg_data)        │
          └───────────┬───────────┘
          ┌───────────┴───────────┐
          ▼                       ▼
  ┌───────────────┐       ┌───────────────┐
  │ Logical Vol 1 │       │ Logical Vol 2 │  (LV)
  │  (lv_projects)│       │  (lv_backup)  │
  └───────────────┘       └───────────────┘
```

1. اول  Physical Volume (PV): دیسک‌های فیزیکی یا پارتیشن‌های خام (مثل ``/dev/sdb``).
2. دوم Volume Group (VG): استخر فضا (Storage Pool) که از ترکیب یک یا چند PV ساخته می‌شود.
3. سوم Logical Volume (LV): پارتیشن‌های منطقی مجزا که از دل VG بیرون کشیده شده و فرمت می‌شوند.

### 🚀 مراحل راه اندازی LVM (Step-by-Step Implementation)

### 📌 پیش‌نیازها:

* دو دیسک خام به عنوان نمونه (``/dev/sdb`` و /``dev/sdc``)
* دسترسی ``sudo``

* ### 1️⃣ گام اول: ساخت Physical Volume (PV)

* ابتدا دیسک‌های خام را برای استفاده در LVM آماده و نشانه‌گذاری کنید:

``sudo pvcreate /dev/sdb /dev/sdc``

بررسی وضعیت PV ها:

```sudo pvs
# یا با جزئیات بیشتر:
sudo pvdisplay
```

### 2️⃣ گام دوم: ساخت Volume Group (VG)

دیسک‌های آماده‌شده را ترکیب کرده و یک Volume Group با نام ``vg_data`` بسازید:

```
sudo vgcreate vg_data /dev/sdb /dev/sdc
```

بررسی وضعیت VG:

```sudo vgs
# یا با جزئیات بیشتر:
sudo vgdisplay
```

### 3️⃣ گام سوم: ساخت Logical Volume (LV)

حالا از فضای استخر ``vg_data`` پارتیشن‌های منطقی جداگانه ایجاد کنید:

```
# ساخت یک LV به حجم ۱۰ گیگابایت با نام lv_projects
sudo lvcreate -L 10G -n lv_projects vg_data

# ساخت یک LV دیگر از تمام فضای باقی‌مانده VG
sudo lvcreate -l 100%FREE -n lv_backup vg_data
```

بررسی وضعیت LV ها:

``sudo lvs``

### 4️⃣ گام چهارم: فرمت و متصل کردن (Mount)

پارتیشن‌های ساخته‌شده در مسیر ``/dev/vg_name/lv_name`` یا ``/dev/mapper/...`` در دسترس هستند:

```
# ۱. فرمت با سیستم‌فایل ext4
sudo mkfs.ext4 /dev/vg_data/lv_projects

# ۲. ساخت مسیر Mount
sudo mkdir -p /mnt/projects

# ۳. اتصال LVM به سیستم
sudo mount /dev/vg_data/lv_projects /mnt/projects

# ۴. بررسی فضای در دسترس
df -h /mnt/projects
```

### ⚡ عملیات پیشرفته: تغییر سایز پویا (Dynamic Resizing)

یکی از بزرگ‌ترین مزایای LVM امکان افزایش آنلاین حجم بدون قطع شدن سیستم است.

### ➕ افزایش حجم یک Logical Volume

برای اضافه کردن ۵ گیگابایت به ``lv_projects`` و اعمال آن روی سیستم‌فایل:

```
# افزایش حجم LV همراه با توسعه آنلاین سیستم‌فایل (پرچم -r)
sudo lvextend -r -L +5G /dev/vg_data/lv_projects
```

### ➕ گسترش Volume Group با اضافه کردن دیسک جدید

اگر فضای استخر ``vg_data`` تمام شد، یک دیسک جدید (``/dev/sdd``) اضافه کرده و VG را گسترش دهید:

```
# ۱. تبدیل دیسک جدید به PV
sudo pvcreate /dev/sdd

# ۲. اضافه کردن PV جدید به VG موجود
sudo vgextend vg_data /dev/sdd
```

### 📸 گرفتن Snapshot در LVM (Backup & Rollback)

با استفاده از قابلیت Snapshot می‌توانید در یک لحظه از وضعیت داده‌های LV کپی بگیرید (بسیار کاربردی قبل از آپدیت‌ها):

```
# ساخت یک Snapshot با حجم ۲ گیگابایت از lv_projects
sudo lvcreate -L 2G -s -n lv_projects_snap /dev/vg_data/lv_projects
```

در صورت نیاز به بازگردانی (Merge):

```sudo lvconvert --merge /dev/vg_data/lv_projects_snap```

### 🧹 پاکسازی و حذف کامل LVM (Cleanup)

جهت حذف LVM، مراحل را دقیقاً به ترتیب معکوس انجام دهید:

```
# ۱. قطع اتصال سیستم‌فایل
sudo umount /mnt/projects

# ۲. حذف Logical Volume
sudo lvremove /dev/vg_data/lv_projects

# ۳. حذف Volume Group
sudo vgremove vg_data

# ۴. حذف Physical Volume
sudo pvremove /dev/sdb /dev/sdc
```

