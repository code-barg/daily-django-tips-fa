

![enter image description here](https://github.com/code-barg/daily-django-tips/blob/data/img/01-model-inheritance.jpeg?raw=true)

<h1 align="center"> ارث بری مدل ها در جنگو - قسمت اول 💡</h1>

سلام 🖐️ حال و احوالتون خوش، قراره امروز بریم سراغ **ارث بری** در مدل های جنگو و یه سری از نکات رو ازش یاد بگیریم، خب از اینجا شروع میکنیم:

#### اصلا مدل چی هست❓

> A model is the single, definitive source of information about your data. It contains the essential fields and behaviors of the data you’re storing. Generally, each model maps to a single database table.

مدل دقیقا خود جنگو اینطور تعریفش میکنه : یک مدل منبع انفرادی و قطعی اطلاعات در مورد داده های شما است. مدل ها شامل فیلدها و رفتارهای ضروری داده هایی است که ذخیره می کنید. به طور کلی، هر مدل به یک جدول پایگاه داده  میباشد !

و حالا ORM جنگو یه لطفی در حق ما داشته و  به ما قابلیت های جالبی در ارث بری مدل ها داده که امروز میخواهیم باهم در موردش بدونیم.

> ORM stands for **Object Relational Mapping**. Django allows us to add, delete, modify and query objects, using an API called ORM. Django provides a default admin interface that is required to perform operations like create, read, update and delete on the model directly.

بله ، orm هم طبق توضیح مخفف **Object Relational Mapping** است. جنگو به ما این امکان را می دهد که با استفاده از یک API به نام ORM، اشیاء را اضافه، حذف، اصلاح و پرس و جو کنیم. جنگو یک رابط مدیریت پیش‌فرض ارائه می‌کند که برای انجام مستقیم عملیاتی مانند ایجاد، خواندن، به‌روزرسانی و حذف روی مدل لازم است.

##  انواع ارث بری در مدل ها

> وقتی صحبت از ارث بری در مدل های جنگو میشه در اصل یعنی قراره کلاسی از یک یا چند کلاس دیگر که همه از `django.db.models.Model` ارث برده اند، ارث بری کند ! 
> چه شیر تو شیری شد ! سخت ترین کار دنیا، توضیح مفهومی کد 

### ارث بری انتزاعی (abstract)

به مثال زیر توجه کنید :

```python
class TimeStampedModel(models.Model):
    created_at = models.DateTimeField(auto_add_now=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True


class Post(TimeStampedModel):
    title = models.CharField(max_length=128)
    ...


class Product(TimeStampedModel):
    name = models.CharField(max_length=128)
    ...
```

ما اگر n کلاس دیگر مثل Product هم داشته باشیم ، باز هم میتونیم براحتی از کلاس انتزاعی خودمون بنام TimeStampedModel استفاده کنیم

خب حالا این عملیات چه نتیجه ای داره ؟ قبل از هر چیز به نکته ای که داره در این کد دقت کنید : 
‍‍‍
```python
class ...:
    ...
	
    class Meta:
        abstract = True
...
```

برای ساخت Mixin مدل ها که در حقیقت بصورت انتزاعی هستند باید این قاعده را رعایت کنیم .

خب حالا تعریفش کنیم :‌ مدل های انتزاعی مدل هایی هستند که

 1.  اصلا موجودیتی در دیتابیس به تنهایی ندارند (جدول TimeStampedModel نداریم )
 2. زمانی مفید هستند که می‌خواهید برخی از اطلاعات رایج را در تعدادی مدل دیگر قرار دهید.
 3. وقتی به عنوان یک کلاس پایه برای مدل‌های دیگر استفاده می‌شود، فیلدهای آن به کلاس فرزند اضافه می‌شود.

 - - -
بریم برای یک مثال عملی و جالب :

```python
from django.db import models  
  
  
class TimeStampedModel(models.Model):  
	"""  
	TimeStampedModel  
	  
	An abstract base class model that provides self-managed "created" and  
	"updated" fields.  
	"""  
	created = models.DateTimeField(auto_now_add=True)  
	updated = models.DateTimeField(auto_now=True)  
	  
	class Meta:  
		abstract = True  
		get_latest_by = 'created' # default to get latest by created  
		ordering = ['-created'] # default to order by created descending  
  
  
class LogicalDeleteModel(models.Model):  
	"""  
	LogicalDeleteModel  
	  
	An abstract base class model that provides an "is_deleted" field.  
	"""  
	is_deleted = models.BooleanField(default=False)  
	  
	class Meta:  
		abstract = True  
  
  
class LogicalActiveModel(models.Model):  
	"""  
	LogicalActiveModel  
	  
	An abstract base class model that provides an "is_active" field.  
	"""  
	is_active = models.BooleanField(default=True)  
	  
	class Meta:  
		abstract = True  
  
  
class Product(LogicalActiveModel, LogicalDeleteModel, TimeStampedModel):  
	"""  
	Product model  
	  
	A product that can be purchased.  
	"""  
	name = models.CharField(max_length=128)  
	description = models.TextField()  
	price = models.DecimalField(max_digits=10, decimal_places=2)  
	quantity = models.IntegerField()  
	  
	def __str__(self):  
		return self.name
```

ما در این مثال ۴ مدل را پیاده سازی کرده ایم
هر سه مدل اول در حقیقت BaseModel یا مدل های انتزاعی هستند، و مدل های دیگر که اینجا ما فقط از مدل Product استفاده کردیم مدل هایی هستند که از قابلیت های مدل های انتزاعی بهره می ببرند.

به فایل `migrations` این مدل ها دقت کنید‌:
```python
from django.db import migrations, models  
  
class Migration(migrations.Migration):  
	initial = True  
	operations = [  
		migrations.CreateModel(  
			name='Product',  
			fields=[  
				('id', models.BigAutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),  
				('created', models.DateTimeField(auto_now_add=True)),  
				('updated', models.DateTimeField(auto_now=True)),  
				('is_deleted', models.BooleanField(default=False)),  
				('is_active', models.BooleanField(default=True)),  
				('name', models.CharField(max_length=128)),  
				('description', models.TextField()),  
				('price', models.DecimalField(decimal_places=2, max_digits=10)),  
				('quantity', models.IntegerField()),  
			],  
			options={  
				'abstract': False,  
			},  
		),  
	]

```
متوجه شدید ؟ هیچ یک از مدل های abstract ما اصلا موجودیتی در دیتابیس ما ندارد ! ولی تمام خصوصیات هر یک از آن مدل ها روی مدل Product وجود دارد و در حقیقت ما از قواعد کد نویسی تمیز به همراه دسترسی و نظم بالا بهره برده ایم.

نکته قابل توجه هم آنجاست که تمام متد (method) و پراپرتی (property) های کلاس های خود را دارند.

خسته نباشید 🤠

---

### پینوشت ها:
سطح این سند عمومی و آسان است ، پس در قسمت های بعدی سعی میکنیم ذره ذره سطح را ارتقا بدهیم.

از دو abstract base class های مهم اینجا برای مثال در حد نمایش رو نمایی شده ، LogicalActive و LogicalDelete دو مدل انتزاعی پر استفاده و کارآمد هستند که بزودی برای هر کدوم جلسه ی مفصلی داریم.
