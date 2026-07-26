**پردازش و آنالیز فایل های متنی به وسیله AWK در لینوکس**
---
یک زبان برنامه نویسی و ابزار قدرتمند در خط فرمان برای پردازش متون و گزارش گیری و دستکاری داده های ستونی (مانند فایل های csv یا خروجی دستورات) است  
سینتکس این ابزار از سه بخش Begin Block - Body Block - End Block تشکل شده است که نوشتن Begin و End Block اختیاری اما Body Block اجباری است  
  Body به ازای هر خط ورودی از فایل یک بار بر روی آن اجرا میشود  
Begin و End فقط یکبار در ابتدا انتهای فایل اجرا میشوند  
|دستور | توضیح |
| :--- | :--- |
`apt install gawk` | برای نصب کردن ابزار
`yum install gawk` | برای نصب کردن ابزار
`awk 'Begin{printf "List if Students\n"} {print 0$} End{printf "End of the file\n"}' test.txt` | سینتکس کلی این ابزار  

**در AWK تعدادی Built-in Variables (متغیرهای داخلی) وجود دارند که بسیار پرکاربرد هستند. مهم‌ترین آن‌ها عبارت‌اند از:**

|دستور | توضیح |
| :--- | :--- |
FS | Field Separator؛ جداکننده‌ی فیلدهای ورودی. مقدار پیش‌فرض فاصله و تب است.
OFS | Output Field Separator؛ جداکننده‌ی فیلدها هنگام چاپ با print. پیش‌فرض یک فاصله است.
RS | Record Separator؛ جداکننده‌ی رکوردهای ورودی. پیش‌فرض \n (هر خط یک رکورد). 
ORS | Output Record Separator؛ جداکننده‌ی رکوردهای خروجی. پیش‌فرض \n.
NF | Number of Fields؛ تعداد فیلدهای رکورد جاری.
NR | Number of Records؛ شماره‌ی رکورد جاری از ابتدای همه‌ی فایل‌ها.
FNR | شماره‌ی رکورد جاری در فایل فعلی (برای هر فایل از ۱ شروع می‌شود).
$0 | کل رکورد (کل خط فعلی).
$1, $2, ... | فیلدهای اول، دوم و ...
FILENAME | نام فایل ورودی فعلی.
ARGC | تعداد آرگومان‌های خط فرمان.
ARGV | آرایه‌ی آرگومان‌های خط فرمان.
ENVIRON | آرایه‌ای از متغیرهای محیطی (Environment Variables).
RSTART | محل شروع آخرین تطابق تابع match().
RLENGTH | طول آخرین تطابق match().
SUBSEP | جداکننده‌ی اندیس‌ها در آرایه‌های چندبعدی (پیش‌فرض "\034").  

**عملگر های ریاضی و Regular Expressions در AWK**
|عملگر | توضیح | مثال
| :--- | :--- | :--- |
| + | جمع | a + b
| - | تفریق | a - b
| * | ضرب | a * b 
| / | تقسیم | a / b
| % | باقیمانده تقسیم | a % b
| ^ یا ** | توان (در GNU awk هر دو پشتیبانی می‌شوند) | a ^ b
|++ | افزایش یک واحد | i++, ++i
|-- | کاهش یک واحد | i--, --i
|= | انتساب | x = 10
| +=, -=, *=, /=, %=, ^= | انتساب همراه با عملیات | sum += $1 


سینتکس های نمونه


``awk 'BEGIN { a = 50; b = 20; print "(a + b) = ", (a + b) }'``


``awk 'BEGIN { a = 50; b = 20; print "(a - b) = ", (a - b) }'``


``awk 'BEGIN { a = 50; b = 20; print "(a * b) = ", (a * b) }'``


``awk 'BEGIN { a = 50; b = 20; print "(a / b) = ", (a / b) }'``


``awk 'BEGIN { a = 50; b = 20; print "(a % b) = ", (a % b) }'``

نکته: اولویت عملگرها مشابه زبان C است و برای کنترل ترتیب اجرا می‌توان از پرانتز () استفاده کرد.

## Pre/Post Increment & Decrement

``awk 'BEGIN { a = 10; b = ++a; printf "a = %d, b = %d\n", a, b }'``

``awk 'BEGIN { a = 10; b = --a; printf "a = %d, b = %d\n", a, b }'``

``awk 'BEGIN { a = 10; b = a++; printf "a = %d, b = %d\n", a, b }'``

``awk 'BEGIN { a = 10; b = a--; printf "a = %d, b = %d\n", a, b }'``

## Logical AND (&&), Logical OR (||):

``awk 'BEGIN { num = 5; if (num >= 0 && num <= 7) printf "%d is in octal format\n", num }'``

``awk 'BEGIN { ch = "\n"; if (ch == " " || ch == "\t" || ch == "\n") print "Current character is whitespace." }'``

## AWK-Regular Expressions

1. One Occurrence (.)

``echo -e "cat\nbat\nfun\nfin\nfan" | awk '/f.n/'``

2. Zero or One Occurrence (?)

``echo -e "Colour\nColor" | awk '/Colou?r/'``

3. Zero or More Occurrence (*)

``echo -e "ca\ncat\ncatt" | awk '/cat*/'``

4. One or More Occurrence (+)

``echo -e "111\n22\n123\n234\n456\n222" | awk '/2+/'``

5. Exclusive Set ([^ ])

## استفاده از ساختاری های کنترلی if-else و تعریف شرط در ابزار AWK 

Syntax:

if (condition) {


    action-1
    action-1
    .
    .
    action-n

    
}

Example:

``awk 'BEGIN {num = 10; if (num % 2 == 0) printf "%d is even number.\n", num }'``
