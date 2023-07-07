![enter image description here](https://github.com/code-barg/daily-django-tips/blob/data/img/01-model-inheritance.jpeg?raw=true)

<h1 align="center"> ارث بری مدل ها در جنگو - قسمت دوم 💡</h1>

 به نام خدا ، سلام سلام 🖐️ اومدیم با قسمت دوم ارث بری در مدل های فریمورک محبوب جنگو ، در قسمت قبل از مدل های انتزاعی و ارث بری ازشون گفتیم حالا بریم سراغ نوع دوم ارث بری ها که به پراکسی (proxy) معروفه بپردازیم 🧐

#### مانند قبل ، بریم تعریف این نوع از مدل رو از خود جنگو ببینیم ❓

> Sometimes, however, you only want to change the **Python behavior** of a model – perhaps to change the default manager, or add a new method.
> 
> This is what proxy model inheritance is for: creating a proxy for the original model. You can create, delete and update instances of the proxy model and all the data will be saved as if you were using the original (non-proxied) model. The difference is that you can change things like the default model ordering or the default manager in the proxy, without having to alter the original.

همانطور که میبینید ، جنگو داره بهمون میگه: گاهی وقت ها شما میخواهید فقط رفتار پایتونی مدلتان متفاوت باشند یا به زبان آدمیزاد ، ممکنه وقت هایی بشه که برای یک جدول ما بصورت های متفاوتی قصد دریافت یا ارسال داده رو داشته باشیم ، اینجاست proxy مدل ها بکمکمون میان. 🤪 



به مثال زیر توجه کنید :

```python
from django.db import models
from django.utils import timezone


class Product(models.Model):
    title = models.CharField(max_length=100)
    created = models.DateTimeField(auto_now_add=True)
    
    
class OrderedProduct(Product):
    class Meta:
        proxy = True
        ordering = ['created']

    def created_delta(self):
        return timezone.now() - self.created
```



خوب که بهش نگاه کنیم میبینیم هیچی نیست ! 🤕

خب ما یک مدل Product داریم که تونستیم با ارث بری ازش یک مدل رفتاری (proxy) ازش بسازیم ، این مدل جدید با نام OrderedProduct نه ماهیت دیتابیسی داره (یعنی داخل دیتابیس چنین جدولی نیست) و نه اجازه این رو داره که تغییر دیتابیسی در مدل اصلی ایجاد کنه !



#### سوال : پس فایدش چیه ؟ (به چه دردی میخوره):❓

در پاسخ به این سوال میشه اینطور گفت که این مدل ها ، زمانی که شما قصد فیلتر کردن داده های اصلی (Product) را بصورت سطحی داشته باشید یا نسبت به خاصیتی (مثلا اگر این محصول تخفیف داشت !) متد هایی نیاز داشته باشید عملا بیفایده است و میتوان راهکار های ساده تری استفاده کرد ، ولی اگر مقصودتان فقط بصورت سطحی و کلی نباشد ، قصد جداسازی به همراه استفاده از الگو های بهینه سازی کوئری ها یا مثلا استفاده از Caching بطور جزئی و حرفه ای تر را داشته باشید استفاده از مدل های پراکسی شده بسیار عالی است !



#### نتیجه :

اول) ارث بری بصورت پراکسی مدل امکان رفتار پایتونیک بهتر و مقیاس پذیر تر را به ما میدهد. ✅

دوم) در این نوع ارث بری ، ما نمیتوانیم تغییری در باطن یا ساختار اصلی مدل ایجاد کنیم. ❎

<br>



### مثال عملی و باحال از جلسه امروز (اهل دلاش اینجاشو دوست دارن 😉)

فایل ‍‍`models.py‍` ما اینشکلیه :

```python
from django.db import models
from django.contrib.auth.models import Group, Permission, User


class SecurityFilter(models.Model):
    """
    Security filter for a permission and a group by filter conditions.
    The filter conditions are evaluated in the context of the user.

    For example, if the filter conditions are "user.is_staff == True",
    then the filter is applied only if the user is staff.
    """
    filter_conditions = models.TextField(verbose_name='filter conditions')
    group = models.ForeignKey(to=Group, related_name='filters', on_delete=models.CASCADE)
    permission = models.ForeignKey(to=Permission, related_name='filters', on_delete=models.CASCADE)

    def __unicode__(self):
        return "%s for %s" % (self.permission, self.group)


class SecurityUserManager(models.Manager):
    """
    Manager for User model.

    This manager provides a method to get a queryset of available security filters
    """

    @staticmethod
    def _get_filters(user, permission):
        """
        Get security filters for a user and a permission.

        :param user: auth.models.User instance
        :param permission: auth.models.Permission instance
        :return: list of SecurityFilter instances
        """
        return SecurityFilter.objects.filter(permission=permission, group__in=user.groups.all())

    @staticmethod
    def _build_query(filters):
        """
        Build a query from a list of security filters.

        :param filters: list of SecurityFilter instances
        :return: a query object
        """
        if not filters:
            return None

        query = models.Q()
        for conditions_set in (f.filter_conditions for f in filters):
            query |= models.Q(**eval(conditions_set))

        return query

    def available(self, user, permission):
        """
        Get a queryset of available security filters for a user and a permission.

        :param user: auth.models.User instance
        :param permission: auth.models.Permission instance
        :return: queryset of SecurityFilter instances
        """
        filters = self._get_filters(user=user, permission=permission)
        query = self._build_query(filters)

        queryset = self.get_queryset()
        if query is None:
            return queryset.none()
        else:
            return queryset.filter(query)


class UserProxy(User):
    """
    Proxy class for User model.
    """
    objects = SecurityUserManager()

    class Meta:
        proxy = True

```

چقدر ساده و پر استفاده ولی باحال 😇

 برای تست و استفادش هم خب این هم فایل `tests.py` ما میتونه راهنمای کاملتون باشه :

```python
from django.test import TestCase
from django.contrib.auth.models import User, Permission, Group
from django.contrib.contenttypes.models import ContentType

from .models import SecurityFilter, UserProxy


class SecurityFilterTestCase(TestCase):
    def setUp(self):
        self.joker = User.objects.create_user(username='joker')
        self.victim_1 = User.objects.create_user(username='victim_1')
        self.victim_2 = User.objects.create_user(username='victim_2')
        self.batman = User.objects.create_user(username='batman')

        self.villains = Group.objects.create(name='villains')
        self.villains.user_set.add(self.joker)

        self.kill_and_rob = Permission.objects.create(
            name='kill and rob',
            codename='kill_and_rob',
            content_type=ContentType.objects.get(app_label='auth', model='user')
        )

        SecurityFilter.objects.create(
            filter_conditions="{'username__startswith': 'victim'}",
            permission=self.kill_and_rob,
            group=self.villains,
        )

    def test_basic_filtering(self):
        condition_string = SecurityFilter.objects.get().filter_conditions
        filter_query = eval(condition_string)

        users = User.objects.filter(**filter_query)
        victims = User.objects.filter(pk__in=[self.victim_1.pk, self.victim_2.pk])

        self.assertEqual(set(users), set(victims))

    def test_security_manager(self):
        users = UserProxy.objects.available(user=self.joker, permission=self.kill_and_rob)
        victims = User.objects.filter(pk__in=[self.victim_1.pk, self.victim_2.pk])

        self.assertEqual(set(users.values_list('pk', flat=True)), set(victims.values_list('pk', flat=True)))

    def test_security_manager_two_filters(self):
        SecurityFilter.objects.create(
            filter_conditions="{'username': 'batman'}",
            permission=self.kill_and_rob,
            group=self.villains,
        )

        users = UserProxy.objects.available(user=self.joker, permission=self.kill_and_rob)
        victims = User.objects.filter(pk__in=[self.victim_1.pk, self.victim_2.pk, self.batman.pk])

        self.assertEqual(set(users.values_list('pk', flat=True)), set(victims.values_list('pk', flat=True)))

    def test_security_manager_two_groups(self):
        archvillains = Group.objects.create(name='archvillains')
        archvillains.user_set.add(self.joker)

        SecurityFilter.objects.create(
            filter_conditions="{'username': 'batman'}",
            permission=self.kill_and_rob,
            group=archvillains,
        )

        users = UserProxy.objects.available(user=self.joker, permission=self.kill_and_rob)
        victims = User.objects.filter(pk__in=[self.victim_1.pk, self.victim_2.pk, self.batman.pk])

        self.assertEqual(set(users.values_list('pk', flat=True)), set(victims.values_list('pk', flat=True)))

    def test_security_manager_two_perms(self):
        make_joke = Permission.objects.create(
            name='make joke',
            codename='make_joke',
            content_type=ContentType.objects.get(app_label='auth', model='user')
        )
        SecurityFilter.objects.create(
            filter_conditions="{'username': 'batman'}",
            permission=make_joke,
            group=self.villains,
        )

        users = UserProxy.objects.available(user=self.joker, permission=make_joke)
        victims = User.objects.filter(pk__in=[self.batman.pk])

        self.assertEqual(set(users.values_list('pk', flat=True)), set(victims.values_list('pk', flat=True)))

    def test_security_manager_no_perms(self):
        SecurityFilter.objects.all().delete()

        users = UserProxy.objects.available(user=self.joker, permission=self.kill_and_rob)

        self.assertEqual(set(users.values_list('pk', flat=True)), set())

```

 موفق و پیروز باشید . ❤️
