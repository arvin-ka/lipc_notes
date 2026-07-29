## مفاهیم اسکریپت نویسی به زبان Bash در لینوکس

### مفاهیم اولیه Shell Scripting در لینوکس 

مفهوم Shebang : در اسکریپت‌نویسی لینوکس و شل (از جمله Bash)، شِبانگ (Shebang) همان خط اول اسکریپت است که با کاراکترهای #! شروع می‌شود.
​وقتی یک فایل متنی قابل اجرا (Executable) را در ترمینال اجرا می‌کنیم، سیستم‌عامل از روی این خط می‌فهمد که کدام مفسر (Interpreter) باید این فایل و کدهای داخلش را بخواند و اجرا کند.

Shebang بالافاصله در اولین خط و اولین کاراکتر فایل قرار میگیرد:

``#!/bin/bash``

(#) کارکتر هش (pound/hash)

(!) کارکتر بنگ (exclamation/bang)

ترکیب ایندو میشود Shebang

### نحوه استفاده از نتیجه دستورات در Script

برای استفاده از نتیجه یک دستور در دستور دیگر و یا به عنوان متغیر .. روش زیر بکار می رود.

دستور موردنظر.. داخل یک (COMMAND)$ و یا COMMAND در بین `` (علامت Backtick) باید بکار روند

با استفاده از **دستور Tee** می توان یک stout نمایش داد و **هم در فایل** redirect کرد

[user@host ~]$ MYLIST=$(ls -l)
[user@host ~]$ echo $MYLIST
total 4 -rw-rw-r-- 1 user1 user1 58 12:24 4 FEB script1
[user@host ~]$ date | tee output.txt
Tue Mar 21 03:53:20 PM UTC 2021
[user@host ~]$ cat output.txt
Tue Mar 21 03:53:20 PM UTC 2021

Bash = Bourne again shell 

پسوند فایل های شل sh است




