![enter image description here](https://github.com/code-barg/daily-django-tips/blob/data/img/01-model-inheritance.jpeg?raw=true)

<h1 align="center"> ارث بری مدل ها در جنگو - قسمت سوم 💡</h1>

جلسه قبل تر : [<u>**ارث بری مدل ها در جنگو - قسمت دوم**</u>](https://github.com/code-barg/daily-django-tips-fa/tree/main/002) 

---

به نام خدا ، سلام دوستان 🖐️ اومدیم با قسمت سوم ارث بری در مدل های فریمورک محبوب جنگو ، قبل تر از <u>[مدل های انتزاعی (abstract)](https://github.com/code-barg/daily-django-tips-fa/tree/main/001)</u> و <u>[مدل پراکسی (proxy)‌](https://github.com/code-barg/daily-django-tips-fa/tree/main/002)</u> و نحوه ارث بری گفتیم حالا بریم سراغ نوع سوم ارث بری ها که به ***multi-table*** معروفه بپردازیم 🧐

#### مانند قبل ، بریم تعریف این نوع از مدل رو از خود جنگو ببینیم ❓

> If you’re subclassing an existing model (perhaps something from another application entirely) and want each model to have its own database table, multi-table-inheritance is the way to go.
> 
>  is when each model in the hierarchy is a model all by itself. Each model corresponds to its own database table and can be queried and created individually. The inheritance relationship introduces links between the child model and each of its parents (via an automatically-created ***OneToOneField***).

دیدی چیشد ؟! میگه هر موقع میبینی یه سری فیلد های یک مدل رو دقیقا در یک مدل دیگه نیاز داری بیا از من استفاده کن ، اولین تفاوتش با مدل های پایه (abstract) خب اینه که والد هر کلاس فرزند هم خودش یک مدل خواهد بود ، یعنی جدول جداگانه داره و در حقیقت برامون یک رابطه OneToOne پیاده میشه 😇

به مثال زیر توجه کنید :

```python
class Image(models.Model):
    src = models.ImageField(upload_to='images/')
    alt = models.CharField(max_length=255)


class BannerImage(models.Model):
    image = models.OneToOneField(Image, on_delete=models.CASCADE)
    available = models.BooleanField(default=True)
    priority = models.IntegerField(default=0)
    link = models.CharField(max_length=255, blank=True, null=True)
    expired = models.DateTimeField(blank=True, null=True)
```

ما اینجا بدون ارث بری و مثل قدیم اومدیم دوتا مدل ساختیم که مدل اول Image و مدل دوم BannerImage نامیده میشه ✨
حالا به رابطه OneToOne ای که BannerImage رو به Image ارتباط میده دقت کنید:

```python
img = Image.objects.create(src="sample.jpg", alt="sample")
banner = BannerImage.objects.create(image=img)
```

خب مثل همیشه ، کنجکاوی اصل مهمیه پس :

```python
print(*banner._meta.get_fields(), sep="\n")
```

```python
app.BannerImage.id
app.BannerImage.image
app.BannerImage.available
app.BannerImage.priority
app.BannerImage.link
app.BannerImage.expired
```



چیز عجیبی وجود نداره و دقیقا همان طوری هست که پیادش کرده بودیم 🙋🏻‍♂️

دلیلی که خواستم فیلد های مدلی که خودمون نوشتیم رو دوباره ببینیم، از این به بعد آشکار میشه! پیش به سوی اصل ماجرا 🚀

```python
class Image(models.Model):
    src = models.ImageField(upload_to='images/')
    alt = models.CharField(max_length=255)


class BannerImage(Image):
    available = models.BooleanField(default=True)
    priority = models.IntegerField(default=0)
    link = models.CharField(max_length=255, blank=True, null=True)
    expired = models.DateTimeField(blank=True, null=True)
```

این که همون مدل های قبلیه ما هستند ، ولی فقط مدل BannerImage داره ارث بری میکنه از Image و فیلد `image` خودش رو از دست داده 🔎

برای بررسی بیشتر ، یخورده باهاش بازی میکنیم:

```python
print(*Image._meta.get_fields(), sep='\n')
```

```python
<OneToOneRel: blog.bannerimage>
app.Image.id
app.Image.src
app.Image.alt
```

```python
print(*BannerImage._meta.get_fields(), sep='\n')
```

```python
app.Image.id
app.Image.src
app.Image.alt
app.BannerImage.image_ptr
app.BannerImage.available
app.BannerImage.priority
app.BannerImage.link
app.BannerImage.expired
```

چقدر جالب 💫 با همین روش ساده و کارآمد ما به یکباره ، با تعریف BannerImage ، هم Image ساخته میشه و هم ارتباط OneToOne براش درست میشه و این رابطه خاصیت معکوس برای هر دو مدل را داره ، مثال‌ :

```python
banner = BannerImage.objects.create(src="multi-table.png", alt="multi-table", priority=5)

print(banner, banner.image_ptr, sep='\n')
```

```python
<BannerImage: BannerImage object (1)>
<Image: Image object (1)>
```

```python
image = Image.objects.get(pk=1)
print(hasattr(Image, 'bannerimage'), image.bannerimage)
```

```python
True <BannerImage: BannerImage object (1)>
```

موفق باشید . ❤️
