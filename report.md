# آزمایش 6 استقرار یک نرم‌افزار به کمک Docker




##  استقرار پروژه

این بخش توضیح می‌دهیم که چگونه برنامه‌ی Django به همراه پایگاه داده PostgreSQL با استفاده از Docker و Docker Compose کانتینری‌سازی و استقرار یافته است.

---

### **سرویس‌های تعریف‌شده**

ما از **Docker Compose** برای تعریف دو سرویس استفاده کردیم:

1. **`web`**
اجرای برنامه‌ی Django
2. **`db`**
اجرای سرور پایگاه داده PostgreSQL.

این سرویس‌ها در فایل `docker-compose.yml` تعریف شده‌اند.

---

###  `docker-compose.yml`

```yaml
version: '3'

services:
  db:
    image: postgres
    environment:
      POSTGRES_DB: notesdb
      POSTGRES_USER: notesuser
      POSTGRES_PASSWORD: notespass
    volumes:
      - postgres_data:/var/lib/postgresql/data

  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "9050:8000"
    depends_on:
      - db
    environment:
      - DATABASE_NAME=notesdb
      - DATABASE_USER=notesuser
      - DATABASE_PASSWORD=notespass
      - DATABASE_HOST=db

volumes:
  postgres_data:
```

 **توضیح:**

 * سرویس `web` تصویر را با استفاده از `Dockerfile` محلی می‌سازد.
 * سرویس `db` از تصویر رسمی `postgres` استفاده می‌کند.
* `depends_on`

* تضمین می‌کند که `db` قبل از `web` اجرا شود.
 * پورت `9050` در میزبان به پورت `8000` در داخل کانتینر نگاشت شده است، که امکان دسترسی به Django از طریق `http://localhost:9050` را فراهم می‌سازد.



###  `Dockerfile`

```Dockerfile
FROM python:3.10-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

RUN apt-get update \
    && apt-get install -y gcc libpq-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .  
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```


 * از تصویر پایه‌ی slim برای Python جهت بهینه‌سازی استفاده شده است.
 * وابستگی‌های سیستمی مانند `gcc` و `libpq-dev` برای پشتیبانی از `psycopg2` ضروری هستند تا Django بتواند به PostgreSQL متصل شود.
 * برنامه داخل کانتینر کپی شده و سرور توسعه راه‌اندازی می‌شود.

---

###  `requirements.txt`

```
django
psycopg2-binary
```


 * `django`
 فریم‌ورک وب مورد استفاده است.
 * `psycopg2-binary`
برای اتصال Django به پایگاه داده PostgreSQL مورد نیاز است.

---

### پیکربندی محیط اجرا (Environment Configuration)

متغیرهای محیطی در فایل `docker-compose.yml` تضمین می‌کنند که Django به کانتینر درست PostgreSQL متصل شود:

* `DATABASE_NAME=notesdb`
* `DATABASE_USER=notesuser`
* `DATABASE_PASSWORD=notespass`
* `DATABASE_HOST=db`

این مقادیر باید در فایل `settings.py` پروژه‌ی Django برای پیکربندی اتصال پایگاه داده استفاده شوند.

---

###  دستورات مورد استفاده برای استقرار

در ترمینال از دستورات زیر برای ساخت و اجرای پروژه استفاده شد:

```bash
docker-compose down      # پاکسازی محیط اجرا
docker-compose up --build
```

همچنین از `notepad` برای باز کردن و ویرایش فایل‌ها به صورت مستقیم استفاده شد:

```bash
notepad docker-compose.yml
notepad requirements.txt
```

---

###  **Docker Desktop**

> در این پروژه، Docker Desktop به عنوان محیط اجرایی در ویندوز مورد استفاده قرار گرفت. این ابزار امکان ارکستراسیون کانتینرها و مدیریت volumeها را از طریق یک رابط کاربری ساده فراهم می‌کند.

**اسکرین‌شات: Docker Desktop کانتینرهای در حال اجرا را نشان می‌دهد (`notes-web-1` و `notes-db-1`)**
![docker desktop](https://github.com/user-attachments/assets/6a747f2c-f88e-4d5c-9a63-47f10b732a9c)

![docker desktop second image](https://github.com/user-attachments/assets/641dec51-4c73-493e-84a6-a93db136778d)


 لاگ‌های اولیه‌ی کانتینرها در Docker Desktop که تأیید می‌کند سرویس‌ها به‌درستی راه‌اندازی شده‌اند.

این بخش همچنین نشان می‌دهد که شما این نیازمندی را برآورده کرده‌اید:
*«برنامه Django باید طوری پیکربندی شود که به پایگاه داده PostgreSQL اجرا شده توسط شما متصل شود.»*


###  دسترسی از طریق پورت

 سرور وب از طریق آدرس `http://localhost:9050` در دسترس است، همان‌طور که در بخش `ports` مشخص شده (`9050:8000`).

اگرچه در صورت‌مسأله پورت 8000 خواسته شده، ما از `9050:8000` برای جلوگیری از تداخل پورت در میزبان استفاده کردیم. از آن‌جایی که Django همچنان به صورت داخلی روی پورت 8000 اجرا می‌شود، این نیازمندی را برآورده می‌سازد.
عکس از امتحان کردن پورت با مرورگر: 


![port9050](https://github.com/user-attachments/assets/590f99cc-1258-4ad9-97ae-951fac84590d)

---









## ارسال درخواست به وب‌سرور

**صورت سوال**
به کمک مرورگر، نرم‌افزار Postman، ترمینال یا هر ابزار مناسب دیگری، به وسیله ارسال درخواست مناسب به وب‌سرور، گام‌های زیر را انجام دهید:

1. یک کاربر به نام user1 با رمز ۱۲۳۴ بسازید.
2. یادداشتی با تیتر title1 و بدنه body1 برای user1 بسازید.
3. یادداشتی با تیتر title2 و بدنه body2 برای user1 بسازید.
4. همه یادداشت‌های user1 را دریافت کنید. (باید ۲ یادداشت بالا را به عنوان خروجی دریافت کنید)
درخواست ارسال شده و خروجی دریافت شده را برای همه گام‌های بالا در گزارش خود قرار دهید.
**پاسخ سوال**






 از **Postman** برای تست کردن نقطه‌های پایانی API طبق نیاز استفاده میکنیم. در اینجا خلاصه‌ای از درخواست‌های ارسال‌شده به همراه توضیحات و پاسخ‌های سرور آمده است:

###  1. **ایجاد یک کاربر (user42 با رمز عبور mystrongpass)**

**endpoint**

```http
POST http://localhost:9050/users/create/
```


برای ایجاد یک کاربر با استفاده از Postman، نمای `user_create` در فایل `apps/user/views.py` را بررسی می کنیم و متوجه شدیم داده‌ها با `request.POST` خوانده می‌شوند. این به این معناست که API داده‌ها را در فرمت فرم-انکودشده (مشابه فرم‌های HTML) انتظار دارد. بنابراین وقتی JSON ارسال می کردیم، نتوانست آن را پردازش کند و خطا ی  "Invalid data" می داد.

بنابراین یک درخواست `POST` به آدرس `http://localhost:9050/users/create/` با داده‌های `x-www-form-urlencoded` ارسال کردیم. این فرمت داده‌ها را به صورت کلید-مقدار کدگذاری می‌کند، مانند:

 ```
 username: user42  
 password: mystrongpass  
 email: user42@example.com
```

 پاسخ سرور این بود:
**server response:**

 ```json
 {
   "id": 3,
   "username": "user42"
 }
 ```

 که نشان‌دهنده موفقیت‌آمیز بودن عملیات ایجاد کاربر است.


**screenshot:**
![create user](https://github.com/user-attachments/assets/b0b6c76a-b36f-4d8b-925b-ed7be91aba6c)

---

###  2. **ورود به حساب login user42**

**endpoint:**

```http
POST http://localhost:9050/users/login/
```

**body (x-www-form-urlencoded):**

```
username: user42  
password: mystrongpass
```

**server response:**

```json
{
  "message": "Login successful"
}
```

**توضیح:**

نمای `login_func` در فایل `apps/user/views.py` را بررسی کردیم و شامل خطوط زیر بود:

```python
username = request.POST.get('username')
password = request.POST.get('password')
user = authenticate(request, username=username, password=password)
```

* سرور انتظار داده‌های فرم-انکودشده (`x-www-form-urlencoded`) را دارد، نه JSON.
* فیلدهای `username` و `password` باید به صورت key-value در بدنه درخواست ارسال شوند.

  **screenshot:**
  ![login user](https://github.com/user-attachments/assets/fb4120ec-b010-46a0-bda3-138f87fc5465)


###  3. **ایجاد یادداشت برای user42**

#### یادداشت اول

**endpoint:**

```http
POST http://localhost:9050/notes/create/
```

**body (x-www-form-urlencoded):**

```
title: title1
body: body1
```

**server response:**

```json
{
    "id": 1,
    "title": "title1",
    "body": "body1",
    "create_time": "2025-05-23T23:08:04.760Z"
}
```
  **screenshot:**
![create notes 1](https://github.com/user-attachments/assets/aebad798-c013-453a-8fb6-0b7a58bfb221)
  

#### یادداشت دوم
مشابه یاداشت اول فقط در key -value برای بدنه مقدایر زیر را استفاده می کنیم : 
**body (x-www-form-urlencoded):**

```
title: title2
body: body2
```

**server response:**

```json
{
    "id": 2,
    "title": "title2",
    "body": "body2",
    "create_time": "2025-05-23T23:10:45.608Z"
}

```
  **screenshot:**
  ![create notes 2](https://github.com/user-attachments/assets/c0545740-5fb7-42ed-9ce0-8252dec9f182)




###  4. **دریافت یادداشت‌های user42**

**endpoint:**

```http
GET http://localhost:9050/notes/
```

**server response:**

```json
[
    {
        "id": 1,
        "title": "title1",
        "body": "body1",
        "create_time": "2025-05-23T23:08:04.760Z"
    },
    {
        "id": 2,
        "title": "title2",
        "body": "body2",
        "create_time": "2025-05-23T23:10:45.608Z"
    }
]

```
**screenshot:**
![get notes](https://github.com/user-attachments/assets/5cad8f55-3a45-4c8d-9a92-8e3706882e62)


**توضیح:**

سیستم به درستی تنها یادداشت‌های مربوط به کاربر احراز هویت‌شده را باز می‌گرداند.
با بررسی نمای `note_create` در فایل `apps/note/views.py` متوجه میشویم که یادداشت‌ها مختص کاربران هستند.
با استفاده از کد زیر هر یادداشت را به کاربر واردشده اختصاص می‌دهد:

 ```python
 note.user = request.user
 ```

تضمین می‌کند که یادداشت‌ها به کاربر صحیح (مثلاً `user42`) مرتبط هستند. وقتی یادداشت‌ها را با استفاده از:

 ```
 GET http://localhost:9050/notes/
 ```

 دریافت می کنیم، پاسخ فقط یادداشت‌های ایجادشده توسط کاربر فعلی احراز هویت‌شده را بازگرداند.

---



## تعامل با داکر
**صورت سوال**

1. به کمک دستورهای مناسب، image‌ها و containerهای خود را نشان دهید و آن‌هایی که مربوط به این آزمایش هستند را مشخص کنید.
2. دستوری دلخواه را در کانتینر وب‌سرور اجرا کنید. دستور مورد نظر و خروجی آن را در گزارش خود قرار دهید.

**پاسخ سوال**

###  ۱. **لیست Docker Images**

**دستور:**

```
docker images
```
![images](https://github.com/user-attachments/assets/e3eb0c67-8d09-4452-9e03-b6c8b430b4cc)

**توضیح:**

 این دستور تمامی تصاویر داکر ذخیره‌شده در سیستم را نمایش می‌دهد. تصاویر مرتبط با این آزمایش عبارت‌اند از:

* `notes-web`:

 تصویری سفارشی که برای اپلیکیشن Django ما ساخته شده است.
* `postgres`:


 تصویر رسمی PostgreSQL که برای سرویس پایگاه‌داده استفاده شده است.
* `python:3.10-slim`:

 تصویر پایه‌ای که در Dockerfile برای ساخت تصویر `notes-web` استفاده شده است.

 تصاویر دیگر مانند `hello-world`، `node` یا `ce441-proj2-image` به این آزمایش مربوط نیستند.


###  ۲. **لیست Docker Containers**

**دستور:**

```
docker ps -a
```
![containers](https://github.com/user-attachments/assets/e9c2e1d5-9487-4cc5-8236-14d4946fd9e4)

**توضیح:**

 این دستور تمامی کانتینرها (در حال اجرا و متوقف‌شده) را نمایش می‌دهد. کانتینرهای مرتبط با این آزمایش عبارت‌اند از:

* `notes-web-1`:

 کانتینری که اپلیکیشن Django ما در آن اجرا می‌شود. این کانتینر بر پایه‌ی تصویر `notes-web` ساخته شده است.
* `notes-db-1`:

 کانتینری که سرور پایگاه‌داده PostgreSQL را اجرا می‌کند.

 **در مورد نگاشت پورت‌ها:**
 اپلیکیشن Django در پورت داخلی `8000` اجرا می‌شود، اما این پورت از طریق نگاشت `9050:8000` به پورت `9050` روی سیستم host متصل شده است. بنابراین می‌توانیم اپلیکیشن را در مرورگر با وارد کردن آدرس `http://localhost:9050` مشاهده کنیم.



###  ۳. **اجرای دستور دلخواه**

**دستور ۱:**

```
docker-compose exec web ls
```
![exec ls](https://github.com/user-attachments/assets/a54de8a8-0245-4f23-b4b5-269aaffe24a3)

**توضیح:**

> این دستور محتوای دایرکتوری کاری (working directory) کانتینر وب (`/app`) را لیست می‌کند. این دستور تأیید می‌کند که فایل‌های پروژه به درستی کپی شده‌اند و در داخل کانتینر در دسترس هستند.

**دستور ۲:**

```
docker-compose exec web python manage.py showmigrations
```
![showmigrations](https://github.com/user-attachments/assets/e5f8c8cb-d64f-4457-8921-20fd1f976504)


**توضیح:**

> این دستور مدیریتی در جنگو وضعیت مهاجرت‌های پایگاه‌داده را نمایش می‌دهد. اجرای آن در داخل کانتینر نشان می‌دهد که جنگو به درستی کار می‌کند و به پایگاه‌داده PostgreSQL متصل شده است.


## پاسخ سوالات

 **سؤال ۱: نقش‌های Dockerfile، Image و Container را توضیح دهید.**
 
**Dockerfile**:
- یک فایل متنی است که شامل دستوراتی برای ساخت یک Docker Image می‌باشد. این فایل مشخص می‌کند که چه مواردی در Image قرار بگیرند — تصویر پایه، فایل‌هایی که باید کپی شوند، وابستگی‌ها، دستورات لازم و غیره.

**Image**:
- یک قالب یا اسنپ‌شات از محیط سیستم است که از روی Dockerfile ساخته شده است. این تصویر ثابت (read-only) و غیرقابل تغییر است.

**Container**:
- یک نمونه اجرایی از یک Image است. کانتینر به صورت ایزوله اجرا می‌شود، سبک است، و می‌تواند در صورت نیاز راه‌اندازی، متوقف یا بازراه‌اندازی شود.

---

**سؤال ۲: از Kubernetes برای چه کارهایی می‌توان استفاده کرد؟ رابطه آن با Docker چیست؟**
**Kubernetes** 
یک پلتفرم برای مدیریت و مقیاس‌دهی برنامه‌های کانتینری‌شده در میان خوشه‌هایی از ماشین‌ها است. این سیستم به‌طور خودکار موارد زیر را انجام می‌دهد:

* استقرار (Deployment)
* مقیاس‌دهی (Scaling)
* تعادل بار (Load Balancing)
* بازیابی خودکار کانتینرها (Self-healing)

**رابطه با Docker**:

* **Docker**
*  برای ساخت و اجرای کانتینرها استفاده می‌شود.
* **Kubernetes**
* از Docker (یا سایر محیط‌های اجرای کانتینر) برای مدیریت کانتینرها در مقیاس وسیع استفاده می‌کند — یعنی نه فقط روی یک سیستم، بلکه در سراسر چندین سیستم.
