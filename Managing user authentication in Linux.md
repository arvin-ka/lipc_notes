## مدیریت احراز هویت کابران در لینوکس
### سیستم احراز هویت PAM در لینوکس 

سیستم **PAM (Pluggable Authentication Modules)** یک لایه میانجی بین برنامه‌ها (`sudo`, `sshd`, `su`) و روش‌های احراز هویت است تا بدون دستکاری سورس‌کد برنامه‌ها، قوانین امنیتی و احراز هویت تغییر یابند.

هر سطر از فایل‌های داخل `/etc/pam.d/` دارای ساختار زیر است:

type    control    module-path    arguments

### انواع ماژول‌ها (Types)

1. auth: بررسی صحت رمز عبور یا هویت کاربر.
2. account: بررسی مجوز ورود (مجاز بودن زمان ورود، منقضی نشدن حساب).
3. password: مدیریت و تغییر رمز عبور.
4. session: اقدامات قبل/بعد لاگین (ایجاد دایرکتوری، لاگ‌گیری).

### کنترل‌کننده‌های اصلی (Control Flags)

1. required: در صورت شکست، احراز هویت رد می‌شود اما ادامه خطوط اجرا می‌شوند.
2. requisite: در صورت شکست، روند فوراً متوقف و رد می‌شود.
3. sufficient: در صورت موفقیت، روند فوراً تایید می‌شود.

### سناریوهای مهم و پرکاربرد با مثال

#### سناریو ۱: جلوگیری از حملات Brute Force

قفل کردن حساب کاربری پس از ۳ بار پسورد اشتباه به مدت ۱۰ دقیقه:

```
# در فایل /etc/pam.d/sshd یا system-auth
auth        required      pam_faillock.so preauth silent deny=3 unlock_time=600
auth        sufficient    pam_unix.so try_first_pass
auth        requisite     pam_faillock.so authfail deny=3 unlock_time=600
```

#### دستورات مدیریت قفل:

```
faillock --user username          # مشاهده وضعیت[cite: 2]
faillock --user username --reset  # باز کردن قفل[cite: 2]
```
### سناریو ۲: اجبار به پسورد قوی

حداقل ۱۲ کاراکتر شامل عدد، حرف بزرگ و نماد:

```
# در فایل /etc/pam.d/common-password
password    requisite     pam_pwquality.so retry=3 minlen=12 dcredit=-1 ucredit=-1 ocredit=-1[cite: 2]
```

 نکات حیاتی هنگام تغییر فایل‌های PAM

1. همیشه یک Session فعال (ترمینال با دسترسی root/sudo) باز نگه دارید تا در صورت اشتباه قفل نشوید.
2. همیشه بک‌آپ بگیرید: ``sudo cp /etc/pam.d/sshd /etc/pam.d/sshd.bak``
3. بررسی لاگ‌ها جهت عیب‌یابی: ``tail -f /var/log/auth.log (یا /var/log/secure)``
