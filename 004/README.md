![enter image description here](https://github.com/code-barg/daily-django-tips/blob/data/img/04-dump-set-up-and-load-initial-data.png?raw=true)

<h1 align="center">استخراج، تنظیم و افزودن داده های ثابت - قسمت اول ⛽</h1>
جلسه قبل تر : [<u>**ارث بری مدل ها در جنگو - قسمت سوم**</u>](https://github.com/code-barg/daily-django-tips-fa/tree/main/003)

---

به نام خدا ، سلام دوستان 🖐️ اومدیم با قسمت بعدی از نکات و ترفند های جالب جنگو، امروز قراره داده های ثابت (Fixture data) رو بشناسیم و ببینیم جنگو چطور نیاز های مارو برای آماده سازی یا استخراج براورده میکنه 😇

#### منطور از <u>داده های ثابت</u> دقیقا چیه ❓

> ممکنه بعضی وقت ها، برای سرویس هایی که برنامه نویسی میکنیم، داده هایی را بصورت از پیش تعریف شده نیاز داشته باشیم که موجودیتی در پایگاه داده داشته باشند.
> 
> یا امکان دارد داده هایی را که در پایگاه داده موجود است، بدور از فرمت BackUp از پایگاه داده، استخراج کنیم! (مثلا بصورت JSON) 🤩
> 
> و حتی داده های استخراج شده را ویرایش ، افزایش یا کاهش دهیم و سپس به پایگاه داده وارد کنیم !

تمام این ها میتوانند نوعی از تعریف داده های ثابت باشند !

بریم سراغ مثال ، فایل `models.py` رو مثل همیشه تعریف میکنیم :

```python
class Author(models.Model):
    """
    Author model

    fields:
        name (CharField): name of the author
    """
    name = models.CharField(max_length=255)

    def __str__(self) -> str:
        return self.name


class Book(models.Model):
    """
    Book model

    fields:
        name (CharField): name of the book
        price (PositiveIntegerField): price of the book
        author (ForeignKey): author of the book
    """

    name = models.CharField(max_length=255)
    price = models.PositiveIntegerField()
    author = models.ForeignKey(Author, on_delete=models.CASCADE)

    class Meta:
        default_related_name = 'books'

    def __str__(self) -> str:
        return self.name
```

خیلی ساده دوتا مدل داریم بنام های نویسنده (Author) و کتاب (Book) ✨

خب باهم میریم چندتا کتاب و نویسنده بسازیم ، از `shell` کمک میگیریم :

```bash
 python manage.py shell -i ipython
```

از آپشن `-i` استفاده کردم برای انتخاب interface بهتر از رابط عمومی پایتون ، من اینجا از `ipython` استفاده میکنم ، اگر دوست داشتید که امتحانش کنید و لذت ببرید ، کافیه فقط در محیط فعالی که پایتون حضور داره (Environment) از دستور نصبش استفاده کنید :

> python -m pip install ipython

خب خب از مطلب اصلی دور نشیم ، بریم سراغ پوسته (Shell) :

```python
from app.models import Author, Book

ferdowsi = Author.objects.create(name="Abul-Qâsem Ferdowsi Tusi")
hafez = Author.objects.create(name="Khwāje Shams-od-Dīn Moḥammad Ḥāfeẓ-e Shīrāzī")
maulana = Author.objects.create(name="Jalāl al-Dīn Muḥammad Balkhī")

Book.objects.bulk_create([
    Book(name='Masnavi', author=maulana, price=100000),
    Book(name='Masnavi', author=maulana, price=100000),
    Book(name='Fih ma Fih', author=maulana, price=220000),
    Book(name='Fal-E Hafez', author=hafez, price=50000),
    Book(name='The Divan of Hafez', author=hafez, price=100000),
    Book(name='Hafiz', author=hafez, price=500000),
    Book(name='Shahnameh', author=ferdowsi, price=800000),
    Book(name='Rostam and Sohrab', author=ferdowsi, price=300000)
])
```

خب دیتا هامون رو داخل پایگاه داده وارد کردیم ، دلیل استفاده از ‍‍`bulk_create` هم بعدا در جلسه های پیش رو که مربوط میشه به Query Optimization بصورت کامل توضیح میدیم ولی فقط برای توضیح کوتاه باید بدونیم بهینه تر و سریع تره در موقعیت های جمعی ! 😉

خب حالا اگر قصد خروجی گرفتن داده هارو از پایگاه داده داشتیم، بصورتی که بعدا هم بتونیم دوباره واردش کنیم چکار کنیم ؟

دقت کنید ما فرضمون استفاده از SQL Backup نیست چون دو دلیل داریم :

-- اولی : داده ها قابل بررسی و مشاهده به فرمت های آشنا باشد (json - yml - ...)

-- دومی : دسترسی و ویرایش آسان

دقیقا همین نیاز ها بود که جنگو بطور پیشفرض برای ما قابلیت `dump` داده هارا پیاده کرده است:

```bash
python manage.py dumpdata
```

جالبه ؟! تمام داده هایی که در پایگاه داده ذخیره شده بود رو بصورتی شبیه به دیکشنری های پایتونی که همان JSON هستند در اینجا ، دریافت کرده ایم ، باهم ببینیم :

```json
[
	{
		"model": "auth.permission",
		"pk": 1,
		"fields": {
			"name": "Can add log entry",
			"content_type": 1,
			"codename": "add_logentry"
		}
	},
	...
	{
		"model": "contenttypes.contenttype",
		"pk": 1,
		"fields": {
			"app_label": "admin",
			"model": "logentry"
		}
	},
	...
	{
		"model": "app.author",
		"pk": 1,
		"fields": {
			"name": "Abul-Qâsem Ferdowsi Tusi"
		}
	},
	...
	{
		"model": "app.book",
		"pk": 1,
		"fields": {
			"name": "Masnavi",
			"price": 100000,
			"author": 3
		}
	},
	...
	{
		"model": "app.book",
		"pk": 8,
		"fields": {
			"name": "Rostam and Sohrab",
			"price": 300000,
			"author": 1
		}
	}
]
```

خب این خیلی خوبه ولی ما به جدول های خودمون فقط نیاز داریم و بهتره داخل فایلی ذخیره داشته باشیم پس :

```bash
python manage.py dumpdata --format json -o authors.json app.author
python manage.py dumpdata --format json -o books.json app.book
```

مگه میشه از جنگو لذت نبرد ؟ ❤️‍🔥

براحتی خروجی با فرمت json از داده هامون به تفکیک نام app و نام مدل (model) گرفت !

خب این فقط اولشه‌ ! ما میتونیم براحتی چنین فایل هایی رو خودمون بسازیم و بعدا به جنگو معرفی کنیم تا برامون اضافه کنه ، با ما باشید تا در ادامه ، افزودن و بارگزاری (load) داده هارو هم یاد بگیریم :

دستوری داریم که دقیقا برعکس dumpdata عمل میکند بنام loaddata :

```bash
  python manage.py loaddata <fixtures files>
```

براحتی میتوان فایل های استخراج (dump) شده رو با دستور loaddata به پایگاه داده افزود!

ما به این فایل ها که حاوی داده هایی با الگو آدرس مدل (model) و خصوصیاتشان هستند ، <mark>فایل های ثابت (fixture)</mark> میگیم !

### ذره بینِ جزئیات 🔎

به داده های یک فایل fixture نگاه کنید :

```json
[
  {
    "model": "app.author",
    "pk": 1,
    "fields": {
      "name": "Abul-Qâsem Ferdowsi Tusi"
    }
  },
  {
    "model": "app.author",
    "pk": 2,
    "fields": {
      "name": "Khwāje Shams-od-Dīn Moḥammad Ḥāfeẓ-e Shīrāzī"
    }
  },
  {
    "model": "app.author",
    "pk": 3,
    "fields": {
      "name": "Jalāl al-Dīn Muḥammad Balkhī"
    }
  }
]
```

اگر pk ها حذف بشه هیچ اتفاقی نمیوفته و عملیات load براحتی انجام میشه ! دلیل وجود داشتنشون فقط بخاطر رابطه هاست که به این صورت باهم ارتباط بگیرند.

---

فرض کنید مدلی بنام Contributor به عنوان مشارکت کننده ها و رابطه M2M با Book داشته باشیم، بصورت زیر :

```python
class Contributor(models.Model):
    """
    Contributor model
    
    fields:
        name (CharField): name of the contributor
    """
    name = models.CharField(max_length=255)
    
    def __str__(self):
        return self.name


class Book(models.Model):
    ...
    contributors = models.ManyToManyField(Contributor)
```

اگر به یکی از کتاب ها چند مشارکت کننده (Contributor) به کمک `shell` اضافه کنیم :

```python
import random
from app.models import Contributor, Book

contributors = Contributor.objects.bulk_create([
    Contributor(name='Ali'),
    Contributor(name='Fatemeh'),
    Contributor(name='Mohammad'),
    Contributor(name='Nima'),
    Contributor(name='Sina'),
    Contributor(name='Arezoo'),
    Contributor(name='Akbar'),
    Contributor(name='Susan'),
    Contributor(name='David'),
    Contributor(name='Maria'),
    Contributor(name='Yasmin'),
    Contributor(name='Sara'),
])

books = Book.objects.all()

for book in books:
    book.contributors.set(random.sample(contributors, random.randint(0, len(contributors))))
```

حالا خروجی dump زیباست ! :

```json
[
  {
    "model": "app.book",
    "pk": 1,
    "fields": {
      "name": "Masnavi",
      "price": 100000,
      "author": 3,
      "contributors": [1, 3, 4, 7]
    }
  },
  {
    "model": "app.book",
    "pk": 2,
    "fields": {
      "name": "Masnavi",
      "price": 100000,
      "author": 3,
      "contributors": [2, 10]
    }
  },
...
]
```

و دقیقا ممکنه مشارکت کننده ها مقداری برابر با [] داشته باشد .

---

برای بارگزاری فایل ها ، اگر از رابطه ها پیروی میکنید ، باید ترتیب را رعایت کنید ، فرض کنید داده ثابت کتاب ما اینگونه باشد :

```json
{
    "model": "app.book",
    "pk": 1,
    "fields": {
      "name": "Masnavi",
      "price": 100000,
      "author": 3,
      "contributors": [1, 7]
    }
  }
```

پس حتما باید قبل از بارگزاری این داده ی ثابت ، داده های 1 , 7 از مدل Contributor را بارگزاری (load) کرده باشیم .

---

در جلسه آینده نمونه بسیار جالب و کاربردی از فایل های داده ثابت (Fixture) ، همراه با نحوه ی درست معماری این فایل ها و کارایی نسبتا عالیشان منتشر میکنم .

موفق باشید . ❤️
