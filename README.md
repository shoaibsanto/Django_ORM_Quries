# Django ORM এবং Queries - সম্পূর্ণ গাইড

## সূচিপত্র
1. [ORM এবং Query Data Fetching পরিচিতি](#1-orm-এবং-query-data-fetching-পরিচিতি)
2. [সমস্ত রেকর্ড সিলেক্ট করার বিভিন্ন উপায়](#2-সমস্ত-রেকর্ড-সিলেক্ট-করার-বিভিন্ন-উপায়)
3. [একক রেকর্ড সিলেক্ট করার বিভিন্ন উপায়](#3-একক-রেকর্ড-সিলেক্ট-করার-বিভিন্ন-উপায়)
4. [প্রথম এবং শেষ রেকর্ড কুয়েরি](#4-প্রথম-এবং-শেষ-রেকর্ড-কুয়েরি)
5. [ফিল্টারিং, বাদ দেওয়া, সর্টিং, লিমিটিং, রেঞ্জ এবং কাউন্ট](#5-ফিল্টারিং-বাদ-দেওয়া-সর্টিং-লিমিটিং-রেঞ্জ-এবং-কাউন্ট)
6. [Query Method Chaining](#6-query-method-chaining)
7. [সরাসরি Raw SQL Queries](#7-সরাসরি-raw-sql-queries)
8. [Query Debugging](#8-query-debugging)
9. [Aggregation Operator Queries](#9-aggregation-operator-queries)
10. [Comparison Operator Queries](#10-comparison-operator-queries)
11. [String Operator Queries](#11-string-operator-queries)
12. [Range এবং Membership Operator Queries](#12-range-এবং-membership-operator-queries)
13. [Insert এবং Delete Operations](#13-insert-এবং-delete-operations)
14. [Update এবং Upsert Operations](#14-update-এবং-upsert-operations)
15. [Relationship Joins, Reverse এবং Direct Relationships](#15-relationship-joins-reverse-এবং-direct-relationships)
16. [INNER JOIN এবং LEFT JOIN Queries](#16-inner-join-এবং-left-join-queries)

---

## 1. ORM এবং Query Data Fetching পরিচিতি

Django ORM (Object-Relational Mapping) একটি শক্তিশালী এবং সহজ পদ্ধতি যা আপনাকে raw SQL এর পরিবর্তে Python code ব্যবহার করে ডাটাবেসের সাথে কাজ করতে দেয়। এটি Python objects কে database tables এ রূপান্তরিত করে এবং এর বিপরীতে।

### ORM কি?
- **ORM** database tables কে Python classes এ map করে
- প্রতিটি model class একটি database table প্রতিনিধিত্ব করে
- model এর প্রতিটি instance table এর একটি row প্রতিনিধিত্ব করে
- Model attributes table এর columns প্রতিনিধিত্ব করে

### মৌলিক Query ধারণা
- **QuerySet**: Database queries এর একটি সংগ্রহ
- **Lazy Evaluation**: Queries প্রয়োজন না হওয়া পর্যন্ত execute হয় না
- **Caching**: QuerySets তাদের results cache করে

### উদাহরণ Models
```python
from django.db import models

class Category(models.Model):
    name = models.CharField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)

class User(models.Model):
    username = models.CharField(max_length=100)
    email = models.EmailField()
    categories = models.ForeignKey(Category, on_delete=models.CASCADE)

class Customer(models.Model):
    name = models.CharField(max_length=100)
    mobile = models.CharField(max_length=15)
    created_at = models.DateTimeField(auto_now_add=True)

class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    unit = models.CharField(max_length=50)
    img_url = models.URLField()
    category = models.ForeignKey(Category, on_delete=models.CASCADE)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
```

---

## 2. সমস্ত রেকর্ড সিলেক্ট করার বিভিন্ন উপায়

### পদ্ধতি ১: `.all()` ব্যবহার করে
```python
def home(request):
    # সমস্ত রেকর্ড QuerySet হিসেবে পাওয়া
    customers = Customer.objects.all()
    return JsonResponse({'data': list(customers.values())})
```

### পদ্ধতি ২: `.all().values()` ব্যবহার করে
```python
def home(request):
    # নির্দিষ্ট ফিল্ড সহ সমস্ত রেকর্ড পাওয়া
    result = Customer.objects.all().values('id', 'name', 'mobile')
    return JsonResponse({'data': list(result)})
```

### পদ্ধতি ৩: `.all().values_list()` ব্যবহার করে
```python
def home(request):
    # সমস্ত রেকর্ড tuples হিসেবে পাওয়া
    result = Customer.objects.all().values_list('id', 'name', 'mobile')
    return JsonResponse({'data': list(result)})
```

### পদ্ধতি ৪: শর্ত ছাড়া `.filter()` ব্যবহার করে
```python
def home(request):
    # filter ব্যবহার করে সমস্ত রেকর্ড পাওয়া
    result = Customer.objects.filter().values()
    return JsonResponse({'data': list(result)})
```

### পদ্ধতি ৫: Iteration ব্যবহার করে
```python
def home(request):
    # সমস্ত রেকর্ড পেয়ে iterate করা
    customers = Customer.objects.all()
    data = [{'id': c.id, 'name': c.name, 'mobile': c.mobile} for c in customers]
    return JsonResponse({'data': data})
```

---

## 3. একক রেকর্ড সিলেক্ট করার বিভিন্ন উপায়

### পদ্ধতি ১: ID দিয়ে `.get()` ব্যবহার করে
```python
def home(request):
    # ID দিয়ে একটি রেকর্ড পাওয়া (না পেলে exception raise করে)
    result = Customer.objects.get(id=1)
    return JsonResponse(model_to_dict(result))
```

### পদ্ধতি ২: `.filter().first()` ব্যবহার করে
```python
def home(request):
    # প্রথম matching রেকর্ড পাওয়া (না পেলে None return করে)
    customer = Customer.objects.filter(id=1).first()
    if customer:
        return JsonResponse({'id': customer.id, 'name': customer.name})
    return JsonResponse({'error': 'Not found'})
```

### পদ্ধতি ৩: শর্ত সহ `.get()` ব্যবহার করে
```python
def home(request):
    try:
        # একাধিক শর্ত দিয়ে একটি রেকর্ড পাওয়া
        customer = Customer.objects.get(name='John', mobile='1234567890')
        return JsonResponse(model_to_dict(customer))
    except Customer.DoesNotExist:
        return JsonResponse({'error': 'Customer not found'})
    except Customer.MultipleObjectsReturned:
        return JsonResponse({'error': 'Multiple customers found'})
```

### পদ্ধতি ৪: Slicing সহ `.filter()` ব্যবহার করে
```python
def home(request):
    # Slicing ব্যবহার করে প্রথম রেকর্ড পাওয়া
    customer = Customer.objects.filter(name='John')[0:1].first()
    if customer:
        return JsonResponse({'id': customer.id, 'name': customer.name})
    return JsonResponse({'error': 'Not found'})
```

---

## 4. প্রথম এবং শেষ রেকর্ড কুয়েরি

### প্রথম রেকর্ড পাওয়া
```python
def home(request):
    # Table থেকে প্রথম রেকর্ড পাওয়া
    customer = Customer.objects.first()
    if customer:
        return JsonResponse({'id': customer.id, 'name': customer.name})
    return JsonResponse({'error': 'No records found'})
```

### Ordering সহ প্রথম রেকর্ড পাওয়া
```python
def home(request):
    # নাম অনুযায়ী সাজিয়ে প্রথম রেকর্ড পাওয়া
    customer = Customer.objects.order_by('name').first()
    return JsonResponse({'id': customer.id, 'name': customer.name})
```

### শেষ রেকর্ড পাওয়া
```python
def home(request):
    # Table থেকে শেষ রেকর্ড পাওয়া
    customer = Customer.objects.last()
    if customer:
        return JsonResponse({'id': customer.id, 'name': customer.name})
    return JsonResponse({'error': 'No records found'})
```

### Ordering সহ শেষ রেকর্ড পাওয়া
```python
def home(request):
    # নাম অনুযায়ী সাজিয়ে শেষ রেকর্ড পাওয়া
    customer = Customer.objects.order_by('name').last()
    return JsonResponse({'id': customer.id, 'name': customer.name})
```

### তারিখ অনুযায়ী সর্বশেষ রেকর্ড পাওয়া
```python
def home(request):
    # created_at field অনুযায়ী সর্বশেষ রেকর্ড পাওয়া
    customer = Customer.objects.latest('created_at')
    return JsonResponse({'id': customer.id, 'name': customer.name})
```

### তারিখ অনুযায়ী সবচেয়ে পুরাতন রেকর্ড পাওয়া
```python
def home(request):
    # created_at field অনুযায়ী সবচেয়ে পুরাতন রেকর্ড পাওয়া
    customer = Customer.objects.earliest('created_at')
    return JsonResponse({'id': customer.id, 'name': customer.name})
```

---

## 5. ফিল্টারিং, বাদ দেওয়া, সর্টিং, লিমিটিং, রেঞ্জ এবং কাউন্ট

### রেকর্ড ফিল্টার করা
```python
def home(request):
    # Case-insensitive search করে রেকর্ড ফিল্টার করা
    customer = Customer.objects.filter(name__icontains='john').values()
    return JsonResponse({'data': list(customer)})
```

```python
def home(request):
    # Case-sensitive search করে রেকর্ড ফিল্টার করা
    customer = Customer.objects.filter(name__contains='john').values()
    return JsonResponse({'data': list(customer)})
```

### রেকর্ড বাদ দেওয়া
```python
def home(request):
    # শর্ত match করে এমন রেকর্ড বাদ দেওয়া
    customer = Customer.objects.exclude(name__icontains='john').values()
    return JsonResponse({'data': list(customer)})
```

### সর্টিং (Ordering)
```python
def home(request):
    # আরোহী ক্রমে সাজানো
    customer = Customer.objects.all().order_by('name').values()
    return JsonResponse({'data': list(customer)})
```

```python
def home(request):
    # অবরোহী ক্রমে সাজানো
    customer = Customer.objects.all().order_by('-name').values()
    return JsonResponse({'data': list(customer)})
```

```python
def home(request):
    # একাধিক column অনুযায়ী সাজানো
    customer = Customer.objects.all().order_by('name', '-created_at').values()
    return JsonResponse({'data': list(customer)})
```

### রেকর্ড লিমিট করা (Slicing)
```python
def home(request):
    # প্রথম ৫টি রেকর্ড পাওয়া
    customer = Customer.objects.all()[0:5].values()
    return JsonResponse({'data': list(customer)})
```

```python
def home(request):
    # Index ৩ থেকে ৫ পর্যন্ত রেকর্ড পাওয়া
    customer = Customer.objects.all()[3:5].values()
    return JsonResponse({'data': list(customer)})
```

### রেকর্ড গণনা করা
```python
def home(request):
    # মোট রেকর্ড গণনা করা
    customer = Customer.objects.count()
    return JsonResponse({'data': customer})
```

```python
def home(request):
    # ফিল্টার করা রেকর্ড গণনা করা
    count = Customer.objects.filter(name__icontains='john').count()
    return JsonResponse({'count': count})
```

---

## 6. Query Method Chaining

Method chaining আপনাকে একাধিক QuerySet methods একসাথে ব্যবহার করে জটিল queries তৈরি করতে দেয়।

### মৌলিক Method Chaining
```python
def home(request):
    # Filter, order_by, এবং values chain করা
    customer = Customer.objects.filter(
        name__icontains='john'
    ).order_by('-created_at').values()
    
    return JsonResponse({'data': list(customer)})
```

### জটিল Method Chaining
```python
def home(request):
    # একাধিক filters, exclude, order_by, এবং limit
    products = Product.objects.filter(
        price__gte=100
    ).exclude(
        name__icontains='old'
    ).order_by('-price', 'name')[0:10].values()
    
    return JsonResponse({'data': list(products)})
```

### Aggregation সহ Method Chaining
```python
from django.db.models import Count, Avg

def home(request):
    # Filter এর সাথে aggregation chain করা
    result = Product.objects.filter(
        price__gt=100
    ).values('category').annotate(
        total=Count('id'),
        avg_price=Avg('price')
    ).order_by('-total')
    
    return JsonResponse({'data': list(result)})
```

### Q Objects সহ Method Chaining
```python
from django.db.models import Q

def home(request):
    # Method chaining এর সাথে জটিল filtering
    products = Product.objects.filter(
        Q(price__lt=500) | Q(name__icontains='laptop')
    ).exclude(
        price__lt=100
    ).order_by('-created_at').values()[:20]
    
    return JsonResponse({'data': list(products)})
```

---

## 7. সরাসরি Raw SQL Queries

কখনও কখনও জটিল operations এর জন্য আপনাকে raw SQL queries execute করতে হতে পারে।

### পদ্ধতি ১: `.raw()` ব্যবহার করে
```python
def home(request):
    # Raw SQL query execute করা
    customers = Customer.objects.raw('SELECT * FROM myapp_customer WHERE name LIKE %s', ['%john%'])
    data = [{'id': c.id, 'name': c.name, 'mobile': c.mobile} for c in customers]
    return JsonResponse({'data': data})
```

### পদ্ধতি ২: `connection.cursor()` ব্যবহার করে
```python
from django.db import connection

def home(request):
    # Cursor দিয়ে raw SQL execute করা
    with connection.cursor() as cursor:
        cursor.execute("SELECT id, name, mobile FROM myapp_customer WHERE name LIKE %s", ['%john%'])
        columns = [col[0] for col in cursor.description]
        rows = cursor.fetchall()
        data = [dict(zip(columns, row)) for row in rows]
    
    return JsonResponse({'data': data})
```

### পদ্ধতি ৩: জটিল Raw Query
```python
def home(request):
    # জটিল JOIN query
    with connection.cursor() as cursor:
        cursor.execute("""
            SELECT p.id, p.name, p.price, c.name as category_name
            FROM myapp_product p
            INNER JOIN myapp_category c ON p.category_id = c.id
            WHERE p.price > %s
            ORDER BY p.price DESC
        """, [100])
        
        columns = [col[0] for col in cursor.description]
        rows = cursor.fetchall()
        data = [dict(zip(columns, row)) for row in rows]
    
    return JsonResponse({'data': data})
```

### পদ্ধতি ৪: `.extra()` ব্যবহার করে
```python
def home(request):
    # QuerySet এ custom SQL যোগ করা
    products = Product.objects.extra(
        select={'price_with_tax': 'price * 1.15'},
        where=['price > %s'],
        params=[100],
        order_by=['-price']
    ).values('id', 'name', 'price', 'price_with_tax')
    
    return JsonResponse({'data': list(products)})
```

---

## 8. Query Debugging

### পদ্ধতি ১: SQL Query Print করা
```python
def home(request):
    # QuerySet পাওয়া
    queryset = Customer.objects.filter(name__icontains='john')
    
    # SQL query print করা
    query = str(queryset.query)
    print(query)
    
    # Execute করে results পাওয়া
    data = list(queryset.values())
    return JsonResponse({'query': query, 'data': data})
```

### পদ্ধতি ২: Django Debug Toolbar ব্যবহার করা
```python
# Install: pip install django-debug-toolbar

# settings.py
INSTALLED_APPS = [
    # ...
    'debug_toolbar',
]

MIDDLEWARE = [
    # ...
    'debug_toolbar.middleware.DebugToolbarMiddleware',
]

INTERNAL_IPS = ['127.0.0.1']
```

### পদ্ধতি ৩: SQL Logging Enable করা
```python
# settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'level': 'DEBUG',
        },
    },
}
```

### পদ্ধতি ৪: connection.queries ব্যবহার করা
```python
from django.db import connection, reset_queries

def home(request):
    reset_queries()  # পূর্ববর্তী queries clear করা
    
    # Queries execute করা
    customers = Customer.objects.filter(name__icontains='john').values()
    products = Product.objects.filter(price__gt=100).values()
    
    # সব executed queries পাওয়া
    queries = connection.queries
    
    return JsonResponse({
        'total_queries': len(queries),
        'queries': queries,
        'data': {
            'customers': list(customers),
            'products': list(products)
        }
    })
```

### পদ্ধতি ৫: Query Execution Plan দেখা
```python
def home(request):
    # Query execution plan পাওয়া
    queryset = Product.objects.filter(price__gt=100).order_by('-price')
    
    # PostgreSQL এর জন্য
    explain = queryset.explain(verbose=True, analyze=True)
    
    return JsonResponse({'explain': explain})
```

---

## 9. Aggregation Operator Queries

Aggregation functions একাধিক values এর উপর calculation করে একটি single value return করে।

### গড় (Avg)
```python
from django.db.models import Avg

def home(request):
    # গড় মূল্য হিসাব করা
    product = Product.objects.aggregate(Avg('price'))
    return HttpResponse(product['price__avg'])
```

### সর্বনিম্ন (Min)
```python
from django.db.models import Min

def home(request):
    # সর্বনিম্ন মূল্য পাওয়া
    product = Product.objects.aggregate(Min('price'))
    return HttpResponse(product['price__min'])
```

### সর্বোচ্চ (Max)
```python
from django.db.models import Max

def home(request):
    # সর্বোচ্চ মূল্য পাওয়া
    product = Product.objects.aggregate(Max('price'))
    return HttpResponse(product['price__max'])
```

### যোগফল (Sum)
```python
from django.db.models import Sum

def home(request):
    # সব মূল্যের যোগফল হিসাব করা
    product = Product.objects.aggregate(Sum('price'))
    return HttpResponse(product['price__sum'])
```

### গণনা (Count)
```python
from django.db.models import Count

def home(request):
    # মোট products গণনা করা
    product = Product.objects.aggregate(Count('price'))
    return HttpResponse(product['price__count'])
```

### একাধিক Aggregations
```python
from django.db.models import Avg, Max, Min, Sum, Count

def home(request):
    # একসাথে একাধিক aggregations করা
    product = Product.objects.aggregate(
        product_avg=Avg('price'),
        product_max=Max('price'),
        product_min=Min('price'),
        product_sum=Sum('price'),
        product_count=Count('price'),
    )
    return JsonResponse(product)
```

### Filtering সহ Aggregation
```python
def home(request):
    # ফিল্টার করা রেকর্ডের aggregate
    result = Product.objects.filter(
        price__gt=100
    ).aggregate(
        avg_price=Avg('price'),
        total_products=Count('id')
    )
    return JsonResponse(result)
```

### Aggregation সহ Group By (annotate)
```python
def home(request):
    # Category অনুযায়ী group করে aggregates হিসাব করা
    result = Product.objects.values('category__name').annotate(
        total_products=Count('id'),
        avg_price=Avg('price'),
        max_price=Max('price'),
        min_price=Min('price')
    ).order_by('-total_products')
    
    return JsonResponse({'data': list(result)})
```

---

## 10. Comparison Operator Queries

Comparison operators আপনাকে numeric বা date comparisons এর ভিত্তিতে রেকর্ড ফিল্টার করতে দেয়।

### সমান (exact)
```python
def home(request):
    # নির্দিষ্ট মূল্যের products খুঁজে বের করা
    product = Product.objects.filter(price=500).values()
    return JsonResponse({'data': list(product)})
```

### এর চেয়ে কম (lt)
```python
def home(request):
    # ৫০০ এর চেয়ে কম মূল্যের products খুঁজে বের করা
    product = Product.objects.filter(price__lt=500).values()
    return JsonResponse({'data': list(product)})
```

### এর চেয়ে বেশি (gt)
```python
def home(request):
    # ২০০ এর চেয়ে বেশি মূল্যের products খুঁজে বের করা
    product = Product.objects.filter(price__gt=200).values()
    return JsonResponse({'data': list(product)})
```

### এর চেয়ে কম বা সমান (lte)
```python
def home(request):
    # ২০০ এর চেয়ে কম বা সমান মূল্যের products খুঁজে বের করা
    product = Product.objects.filter(price__lte=200).values()
    return JsonResponse({'data': list(product)})
```

### এর চেয়ে বেশি বা সমান (gte)
```python
def home(request):
    # ২০০ এর চেয়ে বেশি বা সমান মূল্যের products খুঁজে বের করা
    product = Product.objects.filter(price__gte=200).values()
    return JsonResponse({'data': list(product)})
```

### সমান নয় (exclude)
```python
def home(request):
    # নির্দিষ্ট মূল্যের সমান নয় এমন products খুঁজে বের করা
    product = Product.objects.exclude(price=500).values()
    return JsonResponse({'data': list(product)})
```

### OR সহ Q Objects ব্যবহার করা
```python
from django.db.models import Q

def home(request):
    # জটিল OR শর্ত
    product = Product.objects.filter(
        Q(name="Laptop") |
        Q(price__lt=2000) |
        Q(price__gt=50)
    ).values()
    
    return JsonResponse({'data': list(product)})
```

### AND সহ Q Objects ব্যবহার করা
```python
from django.db.models import Q

def home(request):
    # জটিল AND শর্ত
    product = Product.objects.filter(
        Q(name="Laptop") &
        Q(price__lt=2000) &
        Q(price__gt=50)
    ).values()
    
    return JsonResponse({'data': list(product)})
```

### জটিল Q Object Combinations
```python
def home(request):
    # AND এবং OR শর্ত একসাথে ব্যবহার করা
    product = Product.objects.filter(
        (Q(price__gte=100) & Q(price__lte=500)) |
        Q(name__icontains='premium')
    ).values()
    
    return JsonResponse({'data': list(product)})
```

### Q Objects দিয়ে Negation
```python
def home(request):
    # Q objects এর সাথে NOT ব্যবহার করা
    product = Product.objects.filter(
        ~Q(price__lt=100) & Q(name__icontains='laptop')
    ).values()
    
    return JsonResponse({'data': list(product)})
```

---

## 11. String Operator Queries

String operators আপনাকে pattern matching এবং text searches করতে দেয়।

### Contains (Case-Sensitive)
```python
def home(request):
    # "pp" সম্বলিত products খুঁজে বের করা (case-sensitive)
    product = Product.objects.filter(name__contains="pp").values()
    return JsonResponse({'data': list(product)})
```

### iContains (Case-Insensitive)
```python
def home(request):
    # "pp" সম্বলিত products খুঁজে বের করা (case-insensitive)
    product = Product.objects.filter(name__icontains="pp").values()
    return JsonResponse({'data': list(product)})
```

### Starts With
```python
def home(request):
    # "p" দিয়ে শুরু হয় এমন products খুঁজে বের করা
    product = Product.objects.filter(name__startswith="p").values()
    return JsonResponse({'data': list(product)})
```

### iStarts With (Case-Insensitive)
```python
def home(request):
    # "p" দিয়ে শুরু হয় এমন products খুঁজে বের করা (case-insensitive)
    product = Product.objects.filter(name__istartswith="p").values()
    return JsonResponse({'data': list(product)})
```

### Ends With
```python
def home(request):
    # "t" দিয়ে শেষ হয় এমন products খুঁজে বের করা
    product = Product.objects.filter(name__endswith="t").values()
    return JsonResponse({'data': list(product)})
```

### iEnds With (Case-Insensitive)
```python
def home(request):
    # "t" দিয়ে শেষ হয় এমন products খুঁজে বের করা (case-insensitive)
    product = Product.objects.filter(name__iendswith="t").values()
    return JsonResponse({'data': list(product)})
```

### OR সহ String Operators
```python
from django.db.models import Q

def home(request):
    # OR দিয়ে string operators একসাথে ব্যবহার করা
    product = Product.objects.filter(
        Q(name__startswith="p") |
        Q(name__endswith="t")
    ).values()
    return JsonResponse({'data': list(product)})
```

### AND সহ String Operators
```python
from django.db.models import Q

def home(request):
    # AND দিয়ে string operators একসাথে ব্যবহার করা
    product = Product.objects.filter(
        Q(name__startswith="p") &
        Q(name__endswith="t")
    ).values()
    return JsonResponse({'data': list(product)})
```

### সম্পূর্ণ মিল (Exact Match)
```python
def home(request):
    # সম্পূর্ণ string মিল
    product = Product.objects.filter(name__exact="Laptop").values()
    return JsonResponse({'data': list(product)})
```

### iExact (Case-Insensitive Exact)
```python
def home(request):
    # Case-insensitive সম্পূর্ণ মিল
    product = Product.objects.filter(name__iexact="laptop").values()
    return JsonResponse({'data': list(product)})
```

### Regex Search
```python
def home(request):
    # Regular expression search
    product = Product.objects.filter(name__regex=r'^[Ll]aptop').values()
    return JsonResponse({'data': list(product)})
```

### iRegex (Case-Insensitive Regex)
```python
def home(request):
    # Case-insensitive regex search
    product = Product.objects.filter(name__iregex=r'laptop|computer').values()
    return JsonResponse({'data': list(product)})
```

---

## 12. Range এবং Membership Operator Queries

### Range Query (অন্তর্ভুক্ত)
```python
def home(request):
    # ১০ থেকে ১০০ এর মধ্যে মূল্যের products খুঁজে বের করা (উভয় অন্তর্ভুক্ত)
    product = Product.objects.filter(price__range=(10, 100)).values()
    return JsonResponse({'data': list(product)})
```

### Range বাদ দেওয়া
```python
def home(request):
    # ১০০ থেকে ৪০০ এর মধ্যে নয় এমন মূল্যের products খুঁজে বের করা
    product = Product.objects.exclude(price__range=(100, 400)).values()
    return JsonResponse({'data': list(product)})
```

### IN Operator
```python
def home(request):
    # নির্দিষ্ট মূল্যের products খুঁজে বের করা
    product = Product.objects.filter(price__in=[100, 200, 300]).values()
    return JsonResponse({'data': list(product)})
```

### IN বাদ দেওয়া
```python
def home(request):
    # নির্দিষ্ট মূল্য বাদ দিয়ে products খুঁজে বের করা
    product = Product.objects.exclude(price__in=[100, 200, 300]).values()
    return JsonResponse({'data': list(product)})
```

### IDs এর তালিকা সহ IN
```python
def home(request):
    # IDs এর তালিকা দিয়ে products খুঁজে বের করা
    product_ids = [1, 5, 10, 15, 20]
    products = Product.objects.filter(id__in=product_ids).values()
    return JsonResponse({'data': list(products)})
```

### Subquery সহ IN
```python
def home(request):
    # নির্দিষ্ট categories এ products খুঁজে বের করা
    category_ids = Category.objects.filter(name__icontains='electronics').values_list('id', flat=True)
    products = Product.objects.filter(category_id__in=category_ids).values()
    return JsonResponse({'data': list(products)})
```

### তারিখের Range Query
```python
from datetime import datetime, timedelta

def home(request):
    # গত ৭ দিনে তৈরি products খুঁজে বের করা
    seven_days_ago = datetime.now() - timedelta(days=7)
    products = Product.objects.filter(
        created_at__range=(seven_days_ago, datetime.now())
    ).values()
    return JsonResponse({'data': list(products)})
```

### Q Objects সহ জটিল Range
```python
from django.db.models import Q

def home(request):
    # একাধিক range শর্ত
    products = Product.objects.filter(
        Q(price__range=(100, 500)) |
        Q(price__range=(1000, 2000))
    ).values()
    return JsonResponse({'data': list(products)})
```

---

## 13. Insert এবং Delete Operations

### create() দিয়ে একক Insert
```python
def home(request):
    # একটি product তৈরি করা
    new_product = Product.objects.create(
        name='OpenAI',
        price=500,
        unit='per user',
        img_url='http://example.com/openai.png',
        category_id=1,
        user_id=1
    )
    return JsonResponse({
        'message': 'Product Uploaded Successfully',
        'id': new_product.id
    })
```

### save() দিয়ে একক Insert
```python
def home(request):
    # Product instance তৈরি করে save করা
    product = Product(
        name='Google AI',
        price=300,
        unit='per month',
        img_url='http://example.com/google.png',
        category_id=1,
        user_id=1
    )
    product.save()
    return JsonResponse({
        'message': 'Product Created Successfully',
        'id': product.id
    })
```

### Bulk Insert
```python
def home(request):
    # Products এর তালিকা প্রস্তুত করা
    products = [
        Product(name='OpenAI', price=500, unit='per user', img_url='http://example.com/openai.png', category_id=1, user_id=1),
        Product(name='ChatGPT', price=600, unit='per user', img_url='http://example.com/chatgpt.png', category_id=1, user_id=1),
        Product(name='DALL-E', price=400, unit='per user', img_url='http://example.com/dalle.png', category_id=1, user_id=1),
        Product(name='Midjourney', price=350, unit='per user', img_url='http://example.com/midjourney.png', category_id=1, user_id=1),
    ]
    
    # Bulk create (একাধিক রেকর্ডের জন্য অনেক দ্রুত)
    Product.objects.bulk_create(products)
    return JsonResponse({'message': 'Products Uploaded Successfully'})
```

### get_or_create() দিয়ে Insert
```python
def home(request):
    # না থাকলে তৈরি করা
    product, created = Product.objects.get_or_create(
        name='Laptop_Pro',
        defaults={
            'price': 1500,
            'unit': 'piece',
            'img_url': 'http://example.com/laptop.png',
            'category_id': 1,
            'user_id': 1
        }
    )
    
    if created:
        message = 'Product Created Successfully'
    else:
        message = 'Product Already Exists'
    
    return JsonResponse({'message': message, 'id': product.id})
```

### একক রেকর্ড Delete করা
```python
def home(request):
    try:
        # নির্দিষ্ট product পেয়ে delete করা
        product = Product.objects.get(id=26)
        product.delete()
        return JsonResponse({'message': 'Product Deleted Successfully'})
    except Product.DoesNotExist:
        return JsonResponse({'message': 'Product Not Found'}, status=404)
```

### একাধিক রেকর্ড Delete করা
```python
def home(request):
    # ১০০ এর কম মূল্যের সব products delete করা
    deleted_count, _ = Product.objects.filter(price__lt=100).delete()
    return JsonResponse({
        'message': 'Products Deleted Successfully',
        'count': deleted_count
    })
```

### সব রেকর্ড Delete করা
```python
def home(request):
    # সব products delete করা
    deleted_count, _ = Product.objects.all().delete()
    return JsonResponse({
        'message': 'All Products Deleted',
        'count': deleted_count
    })
```

---

## 14. Update এবং Upsert Operations

### একক রেকর্ড Update করা
```python
def home(request):
    try:
        # নির্দিষ্ট product পেয়ে update করা
        product = Product.objects.get(id=28)
        product.name = "New_Name"
        product.price = 120
        product.unit = "Per Hour"
        product.save()
        
        return JsonResponse({'message': 'Product Updated Successfully'})
    except Product.DoesNotExist:
        return JsonResponse({'message': 'Product Not Found'}, status=404)
```

### একাধিক রেকর্ড Update করা
```python
def home(request):
    # একটি category এর সব products update করা
    updated_count = Product.objects.filter(
        category_id=1
    ).update(
        price=F('price') * 1.1  # মূল্য ১০% বৃদ্ধি করা
    )
    
    return JsonResponse({
        'message': 'Products Updated Successfully',
        'count': updated_count
    })
```

### F() Expression দিয়ে Update
```python
from django.db.models import F

def home(request):
    # বর্তমান মান ব্যবহার করে মূল্য update করা
    Product.objects.filter(id=10).update(
        price=F('price') + 50  # বর্তমান মূল্যে ৫০ যোগ করা
    )
    return JsonResponse({'message': 'Price Updated Successfully'})
```

### শর্তসাপেক্ষ Update
```python
from django.db.models import Case, When, Value, IntegerField

def home(request):
    # শর্ত অনুযায়ী update করা
    Product.objects.all().update(
        price=Case(
            When(price__lt=100, then=Value(100)),
            When(price__gt=1000, then=Value(1000)),
            default=F('price'),
            output_field=IntegerField(),
        )
    )
    return JsonResponse({'message': 'Prices Normalized'})
```

### Upsert (Update অথবা Create)
```python
def home(request):
    try:
        # আছে তো update, নাহলে create করা
        product, created = Product.objects.update_or_create(
            name="Laptop_New",
            defaults={
                "price": 150,
                "unit": "per kg",
                "img_url": "https://example.com/abc.png",
                "category_id": 1,
                "user_id": 1,
            }
        )
        
        if created:
            message = "Product Inserted Successfully"
        else:
            message = "Product Updated Successfully"
        
        return JsonResponse({"message": message, "id": product.id})
    except Exception as e:
        return JsonResponse({'message': str(e)}, status=400)
```

### Bulk Update
```python
def home(request):
    # একাধিক রেকর্ড দক্ষভাবে update করা
    products = Product.objects.filter(category_id=1)
    
    for product in products:
        product.price = product.price * 1.2
    
    # Bulk update (Django 2.2+)
    Product.objects.bulk_update(products, ['price'])
    
    return JsonResponse({'message': 'Bulk Update Completed'})
```

### select_for_update() দিয়ে Update (Locking)
```python
from django.db import transaction

def home(request):
    # Race conditions প্রতিরোধ করতে update এর সময় row lock করা
    with transaction.atomic():
        product = Product.objects.select_for_update().get(id=10)
        product.price = 500
        product.save()
    
    return JsonResponse({'message': 'Product Updated with Lock'})
```

---

## 15. Relationship Joins, Reverse এবং Direct Relationships

### Relationships বোঝা

#### Direct Relationship (Forward)
যখন একটি model এর অন্য model এর সাথে ForeignKey থাকে, তখন আপনি সরাসরি related object access করতে পারেন।

```python
# Product এর Category এর সাথে ForeignKey আছে
product = Product.objects.get(id=1)
category_name = product.category.name  # সরাসরি access
```

#### Reverse Relationship (Backward)
Relationship এর "অন্য দিক" থেকে related objects access করা।

```python
# Category থেকে Products (reverse)
category = Category.objects.get(id=1)
products = category.product_set.all()  # Reverse access
```

### Direct Relationship Query
```python
def home(request):
    # Related object access করা (ForeignKey)
    products = Product.objects.select_related('category', 'user').values(
        'id', 'name', 'price',
        'category__name',
        'user__username'
    )
    return JsonResponse({'data': list(products)})
```

### Reverse Relationship Query
```python
def home(request):
    # Parent থেকে related objects access করা
    categories = Category.objects.prefetch_related('product_set').all()
    
    data = []
    for category in categories:
        data.append({
            'category': category.name,
            'products': list(category.product_set.values('name', 'price'))
        })
    
    return JsonResponse({'data': data})
```

### Custom Related Name
```python
# models.py তে
class Product(models.Model):
    category = models.ForeignKey(
        Category,
        on_delete=models.CASCADE,
        related_name='products'  # product_set এর পরিবর্তে custom নাম
    )

# views.py তে
def home(request):
    categories = Category.objects.prefetch_related('products').all()
    
    data = []
    for category in categories:
        data.append({
            'category': category.name,
            'products': list(category.products.values('name', 'price'))
        })
    
    return JsonResponse({'data': data})
```

### Many-to-Many Relationships
```python
# models.py
class Student(models.Model):
    name = models.CharField(max_length=100)
    courses = models.ManyToManyField('Course', related_name='students')

class Course(models.Model):
    title = models.CharField(max_length=200)

# views.py
def home(request):
    # সব courses সহ student পাওয়া
    student = Student.objects.prefetch_related('courses').get(id=1)
    courses = student.courses.all()
    
    # সব students সহ course পাওয়া (reverse)
    course = Course.objects.prefetch_related('students').get(id=1)
    students = course.students.all()
    
    return JsonResponse({
        'student_courses': list(courses.values()),
        'course_students': list(students.values())
    })
```

### Nested Relationships
```python
def home(request):
    # Nested relationships access করা
    products = Product.objects.select_related(
        'category',
        'user__categories'  # Nested relationship
    ).values(
        'name',
        'category__name',
        'user__username',
        'user__categories__name'
    )
    
    return JsonResponse({'data': list(products)})
```

---

## 16. INNER JOIN এবং LEFT JOIN Queries

### select_related() দিয়ে INNER JOIN
```python
def home(request):
    # INNER JOIN: শুধু category সহ products
    user = User.objects.filter().select_related('categories').values(
        'id', 'username',
        'categories__id',
        'categories__name'
    )
    
    # SQL query পাওয়া
    query = str(user.query)
    
    return JsonResponse({
        'query': query,
        'data': list(user)
    })
```

### select_related() দ্বারা উৎপন্ন SQL:
```sql
SELECT 
    user.id,
    user.username,
    categories.id,
    categories.name
FROM myapp_user AS user
INNER JOIN myapp_category AS categories ON (user.categories_id = categories.id)
```

### Annotations দিয়ে LEFT JOIN
```python
from django.db.models import Count, OuterRef, Subquery

def home(request):
    # LEFT JOIN: সব categories, products ছাড়াও
    categories = Category.objects.annotate(
        product_count=Count('product')
    ).values('id', 'name', 'product_count')
    
    return JsonResponse({'data': list(categories)})
```

### একাধিক JOINs
```python
def home(request):
    # একাধিক INNER JOINs
    products = Product.objects.select_related(
        'category',
        'user',
        'user__categories'
    ).values(
        'id', 'name', 'price',
        'category__name',
        'user__username',
        'user__categories__name'
    )
    
    query = str(products.query)
    return JsonResponse({
        'query': query,
        'data': list(products)
    })
```

### prefetch_related() দিয়ে LEFT JOIN
```python
def home(request):
    # Reverse relationships এর জন্য LEFT JOIN
    category = Category.objects.prefetch_related('product_set').values(
        'name',
        'product__id',
        'product__name'
    )
    
    query = str(category.query)
    return JsonResponse({
        'query': query,
        'data': list(category)
    })
```

### extra() দিয়ে Custom JOIN
```python
def home(request):
    # Custom JOIN শর্ত
    products = Product.objects.extra(
        tables=['myapp_category'],
        where=['myapp_product.category_id = myapp_category.id AND myapp_category.name = %s'],
        params=['Electronics']
    ).values('id', 'name', 'price')
    
    return JsonResponse({'data': list(products)})
```

### Subquery JOIN
```python
from django.db.models import OuterRef, Exists

def home(request):
    # যে categories এ products আছে সেগুলি পাওয়া
    has_products = Product.objects.filter(category_id=OuterRef('id'))
    
    categories = Category.objects.annotate(
        has_products=Exists(has_products)
    ).filter(has_products=True).values()
    
    return JsonResponse({'data': list(categories)})
```

### Performance: select_related বনাম prefetch_related

#### select_related (INNER JOIN)
- ForeignKey এবং OneToOne relationships এর জন্য ব্যবহার করুন
- SQL JOIN তৈরি করে
- একটি database query
- ছোট related sets এর জন্য বেশি দক্ষ

```python
# JOIN সহ একটি query
products = Product.objects.select_related('category').all()
for product in products:
    print(product.category.name)  # কোন অতিরিক্ত query নেই
```

#### prefetch_related (আলাদা queries)
- ManyToMany এবং reverse ForeignKey relationships এর জন্য ব্যবহার করুন
- আলাদা queries তৈরি করে এবং Python এ join করে
- একাধিক database queries
- বড় related sets এর জন্য বেশি দক্ষ

```python
# দুটি queries: একটি categories এর জন্য, একটি products এর জন্য
categories = Category.objects.prefetch_related('product_set').all()
for category in categories:
    for product in category.product_set.all():  # প্রতিটি product এর জন্য অতিরিক্ত query নেই
        print(product.name)
```

### জটিল JOIN উদাহরণ
```python
from django.db.models import Prefetch

def home(request):
    # Filtering সহ জটিল prefetch
    expensive_products = Product.objects.filter(price__gt=500)
    
    categories = Category.objects.prefetch_related(
        Prefetch('product_set', queryset=expensive_products, to_attr='expensive_products')
    ).all()
    
    data = []
    for category in categories:
        data.append({
            'category': category.name,
            'expensive_products': [p.name for p in category.expensive_products]
        })
    
    return JsonResponse({'data': data})
```

---

## সর্বোত্তম অভ্যাস (Best Practices)

### ১. select_related এবং prefetch_related ব্যবহার করুন
N+1 query সমস্যা এড়াতে এই methods যথাযথভাবে ব্যবহার করুন।

### ২. বড় Datasets এর জন্য values() এবং values_list() ব্যবহার করুন
যখন আপনার পূর্ণ model instances এর প্রয়োজন নেই, তখন memory ব্যবহার কমাতে এই methods ব্যবহার করুন।

### ৩. count() এর পরিবর্তে exists() ব্যবহার করুন
রেকর্ড আছে কিনা চেক করার সময়:
```python
# খারাপ
if Product.objects.filter(name='Laptop').count() > 0:
    pass

# ভালো
if Product.objects.filter(name='Laptop').exists():
    pass
```

### ৪. বড় QuerySets এর জন্য iterator() ব্যবহার করুন
```python
for product in Product.objects.all().iterator():
    # product প্রসেস করুন
    pass
```

### ৫. bulk_create এবং bulk_update ব্যবহার করুন
একাধিক রেকর্ডের জন্য, সবসময় bulk operations ব্যবহার করুন।

### ৬. Database-Level Operations এর জন্য F() ব্যবহার করুন
```python
# Database এ update করুন, Python এ নয়
Product.objects.filter(id=1).update(price=F('price') * 1.1)
```

### ৭. জটিল Queries এর জন্য Q Objects ব্যবহার করুন
Q objects দিয়ে একাধিক শর্ত পরিষ্কারভাবে একত্রিত করুন।

### ৮. Foreign Keys এবং ঘন ঘন Query করা Fields Index করুন
```python
class Product(models.Model):
    name = models.CharField(max_length=200, db_index=True)
```

### ৯. Fields সীমিত করতে only() এবং defer() ব্যবহার করুন
```python
# শুধুমাত্র নির্দিষ্ট fields load করুন
products = Product.objects.only('name', 'price')

# নির্দিষ্ট fields বাদে সব load করুন
products = Product.objects.defer('description', 'img_url')
```

### ১০. Query Performance নিরীক্ষণ করুন
ধীর queries চিহ্নিত করতে Django Debug Toolbar এবং database query logs ব্যবহার করুন।

---

## সাধারণ ভুল (Common Pitfalls)

### ১. N+1 Query সমস্যা
```python
# খারাপ - N+1 queries তৈরি করে
products = Product.objects.all()
for product in products:
    print(product.category.name)  # প্রতিটি product এর জন্য অতিরিক্ত query

# ভালো - JOIN সহ একটি query
products = Product.objects.select_related('category').all()
for product in products:
    print(product.category.name)
```

### ২. get() Exceptions এড়ানো
```python
# খারাপ - Exception raise করতে পারে
product = Product.objects.get(id=999)

# ভালো - try-except বা filter().first() ব্যবহার করুন
try:
    product = Product.objects.get(id=999)
except Product.DoesNotExist:
    product = None

# অথবা
product = Product.objects.filter(id=999).first()
```

### ৩. প্রয়োজন না হলে count() ব্যবহার করা
```python
# খারাপ - দুইবার query execute করে
if Product.objects.filter(price__gt=100).count() > 0:
    products = Product.objects.filter(price__gt=100)

# ভালো - একবার execute করুন
products = Product.objects.filter(price__gt=100)
if products.exists():
    # products ব্যবহার করুন
```

---

## উপসংহার

Django ORM databases এর সাথে interact করার একটি শক্তিশালী, Pythonic উপায় প্রদান করে। এই patterns আয়ত্ত করে এবং best practices অনুসরণ করে, আপনি raw SQL না লিখেই দক্ষ, maintainable database queries লিখতে পারবেন।

### মূল বিষয়সমূহ:
- পরিষ্কার কোডের জন্য QuerySet methods ব্যবহার করুন
- Performance এর জন্য select_related এবং prefetch_related লাভজনক
- কখন aggregation, annotation, এবং filtering ব্যবহার করতে হয় তা বুঝুন
- Query performance নিরীক্ষণ করুন এবং প্রয়োজন হলে optimize করুন
- ভালো maintainability এর জন্য Django conventions অনুসরণ করুন

### আরও শেখার জন্য:
- Django Official Documentation: https://docs.djangoproject.com/en/stable/topics/db/queries/
- Django ORM Cookbook: https://books.agiliq.com/projects/django-orm-cookbook/
- Django Debug Toolbar: https://django-debug-toolbar.readthedocs.io/

---

**Django Developers দের জন্য ❤️ সহকারে তৈরি**
