## مفاهیم اسکریپت نویسی به زبان Bash در لینوکس

### در این مستند جامع، مفاهیم کلیدی بش اسکریپتینگ شامل:

1. شیبنگ (shebang) و مجوز های دسترسی
2. تعریف متغیرها و اصول آن‌ها
3. شرط‌ها (if, elif, else) و عملگرهای مختلف
4. ورودی‌های خط فرمان و تعاملی (read)
5. حلقه‌ها (for, while) و پروژه‌های تعاملی
6. توابع، پاس دادن ورودی و مقادیر بازگشتی
7. محاسبات ریاضی و عملگرها (Arithmetic Operations)
   
### شیبنگ (shebang) چیست؟

در سطر اول هر فایل اسکریپت باید مشخص کنیم که سیستم‌عامل از کدام مترجم (Shell) برای اجرای کدهای این فایل استفاده کند. به این سطر Shebang گفته می‌شود که با کاراکترهای #! شروع می‌شود:

``#!/bin/bash``

``echo "Hello, Shell Programming!"``

#! ترکیب هشتگ (#) و بنگ (!) است که اصطلاحاً Shebang خوانده می‌شود.

و  /bin/bash مسیر اجرایی شل Bash در سیستم‌عامل است.

### مجوزهای دسترسی (Permissions)

اگر فایل اسکریپت را به صورت زیر اجرا کنید:

``./first.sh``

احتمالاً با خطای Permission denied مواجه می‌شوید. دلیل این امر نداشتن مجوز اجرای فایل (Execute) است.

بررسی و اعطای مجوز اجرا:

برای مشاهده مجوزهای فعلی:

``ls -l first.sh``

برای اعطای مجوز اجرا به کاربر جاری (User):

``sudo chmod u+x first.sh``

پس از اعطای مجوز، اسکریپت با دستور ./first.sh به راحتی اجرا خواهد شد.

### متغیرها (Variables)

متغیرها برای ذخیره‌سازی موقت داده‌ها استفاده می‌شوند.

قواعد تعریف متغیر در Bash:

1. نباید هیچ فاصله‌ای (Space) در دو طرف علامت مساوی = وجود داشته باشد.

2. طبق استاندارد نام‌گذاری در لینوکس، نام متغیرهای چندکلمه‌ای با آندرلاین (_) جدا می‌شوند (Snake Case).

3. برای دسترسی به مقدار متغیر، قبل از نام آن علامت $ قرار می‌گیرد.

```
 #!/bin/bash

# تعریف متغیرها (بدون فاصله در اطراف =)
first_name="your name"
file_name="test.txt"

# فراخوانی متغیر با علامت $
echo "First Name is: $first_name"

# استفاده از متغیر برای اجرای دستورات سیستم‌عامل
touch $file_name
```
### دستورات شرطی (Conditions & If/Else)

ساختار شرط‌ها در بش اسکریپت به صورت زیر است:

```
if [ condition ]; then
    # دستورات در صورت درست بودن شرط
elif [ another_condition ]; then
    # دستورات شرط دوم
else
    # دستورات در صورت برقرار نبودن هیچ‌کدام
fi
```

نکته مهم: حتماً بین چوب‌خط‌های کروشه [ ] و عبارت داخل آن فاصله (Space) رعایت شود.

#### عملگرهای بررسی فایل (File Test Operators)

برای بررسی وضعیت فایل‌ها و دایرکتوری‌ها استفاده می‌شوند:

| عملگر | کاربرد |
|:---|:---|
-d | بررسی وجود داشتن **دایرکتوری**
-f | بررسی وجود داشتن **فایل معمولی**
-e | بررسی وجود داشتن فایل یا دایرکتوری
-r | بررسی داشتن مجوز **خواندن** (Read)
-w | بررسی داشتن مجوز **نوشتن** (Write)
-x | بررسی داشتن مجوز **اجرا** (Execute)

مثال کاربردی:

```
#!/bin/bash

dir_name="arvin"

if [ -d $dir_name ]; then
    echo "Directory $dir_name exists."
else
    echo "Directory $dir_name does not exist. Creating now..."
    mkdir "$dir_name"
fi
```

#### عملگرهای مقایسه‌ای اعداد (Relational Operators)

برای مقایسه مقادیر عددی در Bash از کلیدواژه‌های زیر استفاده می‌شود:

| عملگر | معادل ریاضی | مفهوم |
| :--- | :--- | :--- |
-eq | = | Equal (مساوی)
-ne | != | Not Equal (نامساوی)
-gt | > | Greater Than (بزرگتر)
-ge | >= | Greater Than or Equal (بزرگتر یا مساوی)
-lt | < | Less Than (کوچکتر)
-le | <= | Less Than or Equal (کوچکتر یا مساوی)

مثال:

```
#!/bin/bash

num=10

if [ $num -eq 10 ]; then
    echo "The number is equal to 10"
fi
```

#### عملگرهای مقایسه‌ای رشته‌ها (String Operators)

برای مقایسه متون و رشته‌ها:

1. = یا == : برابر بودن دو رشته
2. != : نابرابر بودن دو رشته
3. -z : بررسی خالی بودن رشته (طول رشته صفر باشد)
4. -n : بررسی غیرخالی بودن رشته

مثال:

```
#!/bin/bash

user_group="arvin"

if [ $user_group = "admin" ]; then
    echo "User group is admin"
else
    echo "User group is not admin"
fi
```
#### شرط‌های چندگانه (elif)

```
#!/bin/bash

user_group="devops"

if [ $user_group = "admin" ]; then
    echo "Group is Admin"
elif [ $user_group = "devops" ]; then
    echo "Group is DevOps"
else
    echo "Unknown Group"
fi
```
### دریافت ورودی از کاربر (Arguments & Read)

#### آرگومان‌های خط فرمان (Positional Parameters)

می‌توان هنگام اجرای اسکریپت، مقادیری را به عنوان ورودی (آرگومان) به آن پاس داد:
`` ./first.sh arvin admin ``

در داخل اسکریپت با متغیرهای موقعیتی به این پارامترها دسترسی داریم:
1. $1 : پارامتر اول (arvin)
2. $2 : پارامتر دوم (admin)
3. $3 تا $9 : پارامترهای بعدی

```
#!/bin/bash

user_name=$1
echo "Username: $user_name"

user_group=$2
echo "Group: $user_group"
if [ $user_group = "admin" ]; then
    echo "Group is Admin"
elif [ $user_group = "arvin" ]; then
    echo "Group is arvin"
else
    echo "user group something else"
fi

```

متغیرهای ویژه آرگومان‌ها

1. $# : تعداد کل آرگومان‌های ورودی پاس داده‌شده.
2. $* یا $@ : نمایش تمام آرگومان‌های ورودی به صورت یکجا.

```
#!/bin/bash

echo " $# is params num "
echo " $* is params "
user_name=$1
echo "Username: $user_name"

user_group=$2
echo "Group: $user_group"
if [ $user_group = "admin" ]; then
    echo "Group is Admin"
elif [ $user_group = "arvin" ]; then
    echo "Group is arvin"
else
    echo "user group something else"
fi
```

#### دریافت ورودی تعاملی با دستور read

گاهی می‌خواهیم حین اجرای اسکریپت، سوالی از کاربر پرسیده شود و ورودی دریافت گردد. سوئیچ -p امکان نمایش پیام تعاملی را فراهم می‌کند:

```
#!/bin/bash

echo "welcome to the bash programming"
read -p "please enter user name" user_name
echo "user name is : $user_name"
read -p "please enter user group : " user_group
echo "user group is : $user_group"

if [ $user_group = "admin" ]
then
       echo"user group is arvin"
else
       echo"user group something else"
fi
```

### حلقه‌ها (Loops)

#### حلقه for

برای تکرار یک مجموعه دستور روی یک لیست از مقادیر استفاده می‌شود.

#### ۱. پیمایش آرگومان‌ها یا لیست کلمات:

```
#!/bin/bash

# پیمایش روی کلمات مجزا
for name in arvin ka ; do
    echo "Hello $name"
done
```

#### ۲. پیمایش روی بازه‌ای از اعداد:

```
#!/bin/bash

# اجرای حلقه از ۱ تا ۱۰
for i in {1..10}; do
    echo "Number: $i"
done
```

#### خواندن خط به خط یک فایل متنی:

فرضا فایلی به نام files.txt داریم، با دستور cat و ترکیب آن با for می‌توان خطوط آن را پیمایش کرد:

```
#!/bin/bash

for f in $(cat files.txt); do
    echo "Processing file: $f"
done
```

### حلقه while

تا زمانی که یک شرط درست باشد، این حلقه ادامه می‌یابد.

#### حلقه شمارنده با شرط عددی:

```
#!/bin/bash

i=1

while [ $i -lt 10 ]; do
    echo "Count: $i"
    # افزایش مقدار متغیر آی
    ((i++))
    # یا ((i+=2)) یا ((i=i*2))
done
```

#### پروژه کاربردی: ساخت تعاملی کاربران سیستم

اسکریپت زیر در یک حلقه بی‌پایان (while true) نام کاربری را دریافت کرده و کاربر سیستم ساخت می‌شود. در صورتی که کاربر حرف q را وارد کند، اجرای حلقه متوقف (break) خواهد شد:

```
#!/bin/bash

while true; do
    read -p "Enter username to create (or 'q' to quit): " username
    
    # شرط خروج از حلقه
    if [ "$username" = "q" ]; then
        echo "Exiting user creation tool."
        break
    fi
    
    # ساخت کاربر در لینوکس
    sudo adduser "$username"
    echo "User $username successfully created!"
done
```

### توابع (Functions)

برای جلوگیری از تکرار کد، تمیزتر شدن برنامه و ارتقای مدیریت اسکریپت از توابع استفاده می‌شود.

#### تعریف و فراخوانی تابع

```
#!/bin/bash

# تعریف تابع
say_hello() {
    echo "Hello from arvin Style Coding!"
}

# فراخوانی تابع (Call)
say_hello
```
#### ارسال پارامتر به تابع

پارامترهای ورودی تابع دقیقا مثل آرگومان‌های اسکریپت با $1 و $2 داخل تابع دریافت می‌شوند:

```
#!/bin/bash

# تعریف تابع با دو ورودی
greet_user() {
    echo "Hello $1 from $2!"
}

# فراخوانی با مقادیر مختلف
greet_user "Arvin" "Arvin Style Coding"
greet_user "Hamed" "DevOps Course"
```

مثال کاربردی: تابع بررسی وجود فایل

```
#!/bin/bash

check_file_exists() {
    if [ -f "$1" ]; then
        echo "File '$1' exists."
    else
        echo "File '$1' does NOT exist."
    fi
}

check_file_exists "files.txt"
check_file_exists "non_existing_file.txt"
```

#### بازگرداندن مقادیر با return و $?

در بش اسکریپت دستور return یک کد وضعیت خروجی (Exit Code بین 0 تا 255) برمی‌گرداند. مقدار برگردانده شده توسط آخرین دستور/تابع اجرا شده، در متغیر ویژه $? ذخیره می‌شود.

```
#!/bin/bash

sum() {
    local total=$(($1 + $2))
    return $total
}

# فراخوانی تابع
sum 10 5

# دریافت خروجی تابع با $?
result=$?

echo "Sum result is: $result"
```

### محاسبات ریاضی و عملگرها (Arithmetic Operations)

در بش اسکریپت به‌صورت پیش‌فرض متغیرها به عنوان **رشته (String)** در نظر گرفته می‌شوند. برای انجام عملیات ریاضی (جمع، تفریق، ضرب، تقسیم و...) باید از روش‌ها و سینتکس‌های مخصوص محاسبات عددی استفاده کنیم.

#### روش‌های انجام محاسبات ریاضی

#### ۱. استفاده از `(( ))` (روش مدرن و پیشنهادی)

بهترین، سریع‌ترین و استانداردترین روش برای انجام محاسبات عددی در Bash استفاده از دو پرانتز `(( ))` است. در این روش نیازی به گذاشتن علامت `$` پیش از نام متغیرها در داخل پرانتز نیست.

```
bash
#!/bin/bash

a=10
b=3

# انجام عملیات و ذخیره در متغیر جدید
sum=$((a + b))
sub=$((a - b))
mul=$((a * b))
div=$((a / b))    # تقسیم صحیح (بدون اعشار)
mod=$((a % b))    # باقی‌مانده تقسیم

echo "Sum: $sum"        # 13
echo "Sub: $sub"        # 7
echo "Mul: $mul"        # 30
echo "Div: $div"        # 3 (نتیجه عدد صحیح است)
echo "Mod: $mod"        # 1
```

### ۲. عملگرهای افزایش و کاهش (Increment / Decrement)

در داخل (( )) می‌توانید مانند زبان‌های C یا Java از عملگرهای کوتاه شده استفاده کنید:

```
#!/bin/bash

count=5

((count++))     # افزودن ۱ واحد
((count += 5))   # افزودن ۵ واحد
((count--))     # کاهش ۱ واحد

echo "Count: $count"
```

### ۳. استفاده از دستور expr (روش کلاسیک/قدیمی)

این روش در اسکریپت‌های قدیمی‌تر دیده می‌شود. نکته مهم در expr این است که باید بین اعداد و عملگرها حتماً فاصله (Space) باشد و برای عملگر ضرب * باید از بک‌اسلش \* برای Escaping استفاده شود.

```
#!/bin/bash

a=10
b=5

# استفاده از Backticks یا $()
result=$(expr $a +$b)
mul_result=$(expr $a \*$b)

echo "Result: $result"
```

### جدول عملگرهای ریاضی در Bash

عملگر | نام عملیات | مثال در Bash | توضیحات |
|:---|:---|:---|:---|
| + | جمع | $((a + b)) | مجموع دو عدد |
| - | تفریق | $((a - b)) | تفریق دو عدد |
| * | ضرب | $((a * b)) | حاصل‌ضرب دو عدد |
| / | تقسیم | $((a / b)) | تقسیم صحیح (بخش اعشاری حذف می‌شود) |
| % | باقی‌مانده (Modulus) | $((a % b)) | باقی‌مانده تقسیم دو عدد |
| ** | توان (Exponentiation) | $((a ** b)) | عدد a به توان b

### محاسبات با اعداد اعشاری (Floating-Point) با bc

دستورات داخلی Bash فقط از اعداد صحیح (Integer) پشتیبانی می‌کنند. اگر نیاز به محاسبات اعشاری یا توابع ریاضی پیشرفته (مثل جذر، سینوس و...) داشته باشید، باید از ابزار bc (Basic Calculator) استفاده کنید.

#### ۱. تقسیم اعشاری با تعیین دقت (scale):

پارامتر scale تعداد ارقام بعد از اعشار را مشخص می‌کند.

```
#!/bin/bash

num1=10
num2=3

# محاسبه تقسیم با ۳ رقم اعشار
result=$(echo "scale=3; $num1 / $num2" | bc)

echo "Float Result: $result"   # خروجی: 3.333
```

#### ۲. محاسبات پیچیده‌تر و ریاضیات پیشرفته:

```
#!/bin/bash

# محاسبه جذر (Square Root)
sqrt_res=$(echo "scale=2; sqrt(16)" | bc -l)

# محاسبه توان اعشاری یا توابع مثلثاتی
power_res=$(echo "scale=4; 2.5 ^ 3" | bc)

echo "Sqrt: $sqrt_res"
echo "Power: $power_res"
```
### پروژه کاربردی: ماشین حساب تعاملی (Interactive Calculator)

اسکریپت زیر نمونه کاملی از ترکیب شرط‌ها، ورودی کاربر و محاسبات ریاضی است:

```
#!/bin/bash

echo "=== Bash Simple Calculator ==="
read -p "Enter First Number: " num1
read -p "Enter Operator (+, -, *, /): " op
read -p "Enter Second Number: " num2

case $op in
    +)
        res=$((num1 + num2))
        ;;
    -)
        res=$((num1 - num2))
        ;;
    \*)
        res=$((num1 * num2))
        ;;
    /)
        if [ $num2 -eq 0 ]; then
            echo "Error: Division by zero!"
            exit 1
        fi
        # استفاده از bc برای داشتن نتیجه اعشاری دسیپلی
        res=$(echo "scale=2; $num1 / $num2" | bc)
        ;;
    *)
        echo "Invalid Operator!"
        exit 1
        ;;
esac

echo "-----------------------------"
echo "Result: $res"
```
