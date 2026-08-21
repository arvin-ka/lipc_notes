# 🗄️ Storage RAID Concepts & Architectures Guide

تکنولوژی **RAID** (مخفف Redundant Array of Independent Disks) روشی برای ترکیب چند دیسک سخت فیزیکی به یک واحد منطقی است. هدف از RAID افزایش **سرعت (Performance)**، **پشتیبانی از خطاپذیری (Redundancy/Fault Tolerance)** یا هر دو می‌باشد.

---

## 📌 مفاهیم کلیدی (Key Concepts)
* **Striping (تکه‌تکه‌سازی):** تقسیم داده‌ها به بخش‌های کوچک‌تر و نوشتن موازی آن‌ها روی چند دیسک (افزایش سرعت).
* **Mirroring (آینه‌سازی):** کپی دقیق داده‌ها روی یک دیسک دیگر (افزایش امنیت و Redundancy).
* **Parity (پاریتی):** محاسبه الگوریتمی داده‌ها برای بازسازی اطلاعات در صورت خراب شدن یکی از دیسک‌ها.

---

## ⚙️ انواع RAIDهای اصلی (Standard RAID Levels)

| نام RAID | حداقل دیسک | مکانیزم | نرخ بهره‌وری فضا | تحمل خرابی (Fault Tolerance) | مزایا و معایب |
| :--- | :---: | :--- | :---: | :---: | :--- |
| **RAID 0** | ۲ دیسک | Striping | ۱۰۰٪ | ❌ صفر دیسک | ⚡ سرعت عالی / ⚠️ بدون امنیت (خرابی ۱ دیسک = نابودی کل داده‌ها) |
| **RAID 1** | ۲ دیسک | Mirroring | ۵۰٪ | 🛡️ ۱ دیسک | 🛡️ امنیت بسیار بالا / ⚠️ هزینه بالا به ازای هر گیگابایت |
| **RAID 5** | ۳ دیسک | Striping + Distributed Parity | $N-1 / N$ | 🛡️ ۱ دیسک | ⚖️ تعادل عالی بین سرعت، امنیت و ظرفیت / ⚠️ سرعت نوشتن متوسط |
| **RAID 6** | ۴ دیسک | Striping + Dual Parity | $N-2 / N$ | 🛡️🛡️ ۲ دیسک | 🛡️ امنیت بالا در برابر خرابی هم‌زمان ۲ دیسک / ⚠️ افت سرعت در نوشتن |

---

## ⚙️ ساختار دیداری و انواع RAIDهای اصلی

### 1️⃣ RAID 0 (Striping)

داده‌ها به قطعات کوچک (Data Blocks) تقسیم شده و به طور هم‌زمان روی دیسک‌ها نوشته می‌شوند.

<p align="center"> <img width="600" height="400" alt="Gemini_Generated_Image_6dmqli6dmqli6dmq" src="https://github.com/user-attachments/assets/05459ab2-3f66-4881-b96e-7e6de8a91aa2" />

### 2️⃣ RAID 1 (Mirroring)

هر داده‌ای که روی دیسک اول نوشته شود، عیناً روی دیسک دوم کپی (Mirror) می‌شود.

<p align="center"> <img width="600" height="400" alt="Gemini_Generated_Image_yaxxlpyaxxlpyaxx" src="https://github.com/user-attachments/assets/83bfcfde-a510-4d88-861f-0542658fb132" />

### 3️⃣ RAID 5 (Striping with Distributed Parity)

داده‌ها و اطلاعات بازیابی (Parity) به‌صورت چرخشی بین تمام دیسک‌ها پخش می‌شوند.

<p align="center"> <img width="600" height="400" alt="Gemini_Generated_Image_ucoj1mucoj1mucoj" src="https://github.com/user-attachments/assets/062cde26-4769-42bb-b637-edc95835df60" />

### 4️⃣ RAID 6 (Striping with Dual Parity)

مشابه RAID 5 است اما از دو بلاک Parity مجزا (P و Q) استفاده می‌کند.

<p align="center"> <img width="1000" height="400" alt="Gemini_Generated_Image_3aaiz13aaiz13aai" src="https://github.com/user-attachments/assets/ab4b0a01-57c1-43e1-b9d5-21675479542d" />


### 🔀 RAIDهای ترکیبی (Nested RAID)

### 1️⃣ RAID 10 (RAID 1+0)
ترکیب Mirroring و Striping؛ دیسک‌ها ابتدا جفت‌جفت RAID 1 شده و سپس مجموعه آن‌ها RAID 0 می‌شود.

- **حداقل دیسک:** ۴ دیسک
- **ساختار:** ابتدا دیسک‌ها به صورت RAID 1 آینه‌سازی شده و سپس مجموعه آن‌ها RAID 0 می‌شوند.
- **مزایا:** سرعت بسیار بالا همراه با امنیت عالی؛ محبوب‌ترین گزینه برای **پایگاه‌های داده (Databases)** و سرورهای حساس.
- **ظرفیت:** ۵۰٪ مجموع دیسک‌ها.

<p align="center"> <img width="1000" height="400" alt="Gemini_Generated_Image_91oqlb91oqlb91oq" src="https://github.com/user-attachments/assets/94bf5b23-fcf0-454a-b5d9-78c69d72502f" />

### 2️⃣ RAID 50 (RAID 5+0)

- **حداقل دیسک:** ۶ دیسک
- **ساختار:** ترکیب دو یا چند آرایه RAID 5 به‌صورت Striping (RAID 0).
- **مزایا:** مناسب برای حجم داده‌های بسیاااار بالا با حفظ سرعت خواندن/نوشتن مناسب.

<p align="center"> <img width="1000" height="400" alt="Gemini_Generated_Image_x7jzflx7jzflx7jz" src="https://github.com/user-attachments/assets/e20c534e-4453-4714-aca9-2f3d6822e0c3" />

## 💻 انواع پیاده‌سازی RAID (Hardware vs Software)

### ۱. Hardware RAID (سخت‌افزاری)
توسط یک کارت پردازش مجزا (RAID Controller) روی سرور مدیریت می‌شود.
* **مزایا:** عدم استفاده از CPU و RAM اصلی سیستم، کارایی بسیار بالا.
* **معایب:** هزینه بالا و وابستگی به سخت‌افزار خاص.

### ۲. Software RAID (نرم‌افزاری)
توسط سیستم‌عامل (مثلاً ابزار `mdadm` در لینوکس) مدیریت می‌شود.
* **مزایا:** کاملاً رایگان، بدون نیاز به سخت‌افزار خاص، انعطاف‌پذیری بالا.
* **معایب:** درگیر کردن بخشی از پردازنده و رم سیستم.

---

## 🛠️ دستور کاربردی بررسی وضعیت RAID نرم‌افزاری در لینوکس

```bash
# مشاهده وضعیت آرایه‌های RAID نرم‌افزاری فعال
cat /proc/mdstat

# بررسی جزئیات یک آرایه خاص با ابزار mdadm
sudo mdadm --detail /dev/md0
