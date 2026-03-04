# Django ORM and Queries - Complete Guide

## Table of Contents
1. [ORM and Query Data Fetching Introduction](#1-orm-and-query-data-fetching-introduction)
2. [Different Ways to Select All Records](#2-different-ways-to-select-all-records)
3. [Different Ways to Select Single Record](#3-different-ways-to-select-single-record)
4. [First and Last Record Query](#4-first-and-last-record-query)
5. [Filtering, Excluding, Sorting, Limiting, Range and Count](#5-filtering-excluding-sorting-limiting-range-and-count)
6. [Query Method Chaining](#6-query-method-chaining)
7. [Raw SQL Queries](#7-raw-sql-queries)
8. [Query Debugging](#8-query-debugging)
9. [Aggregation Operator Queries](#9-aggregation-operator-queries)
10. [Comparison Operator Queries](#10-comparison-operator-queries)
11. [String Operator Queries](#11-string-operator-queries)
12. [Range and Membership Operator Queries](#12-range-and-membership-operator-queries)
13. [Insert and Delete Operations](#13-insert-and-delete-operations)
14. [Update and Upsert Operations](#14-update-and-upsert-operations)
15. [Relationship Joins, Reverse and Direct Relationships](#15-relationship-joins-reverse-and-direct-relationships)
16. [INNER JOIN and LEFT JOIN Queries](#16-inner-join-and-left-join-queries)

---

## 1. ORM and Query Data Fetching Introduction

Django ORM (Object-Relational Mapping) provides a powerful and intuitive way to interact with databases using Python code instead of raw SQL. It translates Python objects into database tables and vice versa.

### What is ORM?
- **ORM** maps database tables to Python classes
- Each model class represents a database table
- Each instance of the model represents a row in the table
- Model attributes represent table columns

### Basic Query Concepts
- **QuerySet**: A collection of database queries
- **Lazy Evaluation**: Queries are not executed until needed
- **Caching**: QuerySets cache their results

### Example Models
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

## 2. Different Ways to Select All Records

### Method 1: Using `.all()`
```python
def home(request):
    # Get all records as QuerySet
    customers = Customer.objects.all()
    return JsonResponse({'data': list(customers.values())})
```

### Method 2: Using `.all().values()`
```python
def home(request):
    # Get all records with specific fields
    result = Customer.objects.all().values('id', 'name', 'mobile')
    return JsonResponse({'data': list(result)})
```

### Method 3: Using `.all().values_list()`
```python
def home(request):
    # Get all records as tuples
    result = Customer.objects.all().values_list('id', 'name', 'mobile')
    return JsonResponse({'data': list(result)})
```

### Method 4: Using `.filter()` without conditions
```python
def home(request):
    # Get all records using filter
    result = Customer.objects.filter().values()
    return JsonResponse({'data': list(result)})
```

### Method 5: Using iteration
```python
def home(request):
    # Get all records and iterate
    customers = Customer.objects.all()
    data = [{'id': c.id, 'name': c.name, 'mobile': c.mobile} for c in customers]
    return JsonResponse({'data': data})
```

---

## 3. Different Ways to Select Single Record

### Method 1: Using `.get()` with ID
```python
def home(request):
    # Get single record by ID (raises exception if not found)
    result = Customer.objects.get(id=1)
    return JsonResponse(model_to_dict(result))
```

### Method 2: Using `.filter().first()`
```python
def home(request):
    # Get first matching record (returns None if not found)
    customer = Customer.objects.filter(id=1).first()
    if customer:
        return JsonResponse({'id': customer.id, 'name': customer.name})
    return JsonResponse({'error': 'Not found'})
```

### Method 3: Using `.get()` with conditions
```python
def home(request):
    try:
        # Get single record with multiple conditions
        customer = Customer.objects.get(name='John', mobile='1234567890')
        return JsonResponse(model_to_dict(customer))
    except Customer.DoesNotExist:
        return JsonResponse({'error': 'Customer not found'})
    except Customer.MultipleObjectsReturned:
        return JsonResponse({'error': 'Multiple customers found'})
```

### Method 4: Using `.filter()` with slicing
```python
def home(request):
    # Get first record using slicing
    customer = Customer.objects.filter(name='John')[0:1].first()
    if customer:
        return JsonResponse({'id': customer.id, 'name': customer.name})
    return JsonResponse({'error': 'Not found'})
```

---

## 4. First and Last Record Query

### Get First Record
```python
def home(request):
    # Get the first record from the table
    customer = Customer.objects.first()
    if customer:
        return JsonResponse({'id': customer.id, 'name': customer.name})
    return JsonResponse({'error': 'No records found'})
```

### Get First Record with Ordering
```python
def home(request):
    # Get first record ordered by name
    customer = Customer.objects.order_by('name').first()
    return JsonResponse({'id': customer.id, 'name': customer.name})
```

### Get Last Record
```python
def home(request):
    # Get the last record from the table
    customer = Customer.objects.last()
    if customer:
        return JsonResponse({'id': customer.id, 'name': customer.name})
    return JsonResponse({'error': 'No records found'})
```

### Get Last Record with Ordering
```python
def home(request):
    # Get last record ordered by name
    customer = Customer.objects.order_by('name').last()
    return JsonResponse({'id': customer.id, 'name': customer.name})
```

### Get Latest Record by Date
```python
def home(request):
    # Get latest record by created_at field
    customer = Customer.objects.latest('created_at')
    return JsonResponse({'id': customer.id, 'name': customer.name})
```

### Get Earliest Record by Date
```python
def home(request):
    # Get earliest record by created_at field
    customer = Customer.objects.earliest('created_at')
    return JsonResponse({'id': customer.id, 'name': customer.name})
```

---

## 5. Filtering, Excluding, Sorting, Limiting, Range and Count

### Filtering Records
```python
def home(request):
    # Filter records with case-insensitive search
    customer = Customer.objects.filter(name__icontains='john').values()
    return JsonResponse({'data': list(customer)})
```

```python
def home(request):
    # Filter records with case-sensitive search
    customer = Customer.objects.filter(name__contains='john').values()
    return JsonResponse({'data': list(customer)})
```

### Excluding Records
```python
def home(request):
    # Exclude records that match condition
    customer = Customer.objects.exclude(name__icontains='john').values()
    return JsonResponse({'data': list(customer)})
```

### Sorting (Ordering)
```python
def home(request):
    # Sort in ascending order
    customer = Customer.objects.all().order_by('name').values()
    return JsonResponse({'data': list(customer)})
```

```python
def home(request):
    # Sort in descending order
    customer = Customer.objects.all().order_by('-name').values()
    return JsonResponse({'data': list(customer)})
```

```python
def home(request):
    # Multiple column sorting
    customer = Customer.objects.all().order_by('name', '-created_at').values()
    return JsonResponse({'data': list(customer)})
```

### Limiting Records (Slicing)
```python
def home(request):
    # Get first 5 records
    customer = Customer.objects.all()[0:5].values()
    return JsonResponse({'data': list(customer)})
```

```python
def home(request):
    # Get records from index 3 to 5
    customer = Customer.objects.all()[3:5].values()
    return JsonResponse({'data': list(customer)})
```

### Counting Records
```python
def home(request):
    # Count total records
    customer = Customer.objects.count()
    return JsonResponse({'data': customer})
```

```python
def home(request):
    # Count filtered records
    count = Customer.objects.filter(name__icontains='john').count()
    return JsonResponse({'count': count})
```

---

## 6. Query Method Chaining

Method chaining allows you to combine multiple QuerySet methods to create complex queries.

### Basic Method Chaining
```python
def home(request):
    # Chain filter, order_by, and values
    customer = Customer.objects.filter(
        name__icontains='john'
    ).order_by('-created_at').values()
    
    return JsonResponse({'data': list(customer)})
```

### Complex Method Chaining
```python
def home(request):
    # Multiple filters, exclude, order_by, and limit
    products = Product.objects.filter(
        price__gte=100
    ).exclude(
        name__icontains='old'
    ).order_by('-price', 'name')[0:10].values()
    
    return JsonResponse({'data': list(products)})
```

### Method Chaining with Aggregation
```python
from django.db.models import Count, Avg

def home(request):
    # Chain filter with aggregation
    result = Product.objects.filter(
        price__gt=100
    ).values('category').annotate(
        total=Count('id'),
        avg_price=Avg('price')
    ).order_by('-total')
    
    return JsonResponse({'data': list(result)})
```

### Method Chaining with Q Objects
```python
from django.db.models import Q

def home(request):
    # Complex filtering with method chaining
    products = Product.objects.filter(
        Q(price__lt=500) | Q(name__icontains='laptop')
    ).exclude(
        price__lt=100
    ).order_by('-created_at').values()[:20]
    
    return JsonResponse({'data': list(products)})
```

---

## 7. Raw SQL Queries

Sometimes you need to execute raw SQL queries for complex operations.

### Method 1: Using `.raw()`
```python
def home(request):
    # Execute raw SQL query
    customers = Customer.objects.raw('SELECT * FROM myapp_customer WHERE name LIKE %s', ['%john%'])
    data = [{'id': c.id, 'name': c.name, 'mobile': c.mobile} for c in customers]
    return JsonResponse({'data': data})
```

### Method 2: Using `connection.cursor()`
```python
from django.db import connection

def home(request):
    # Execute raw SQL with cursor
    with connection.cursor() as cursor:
        cursor.execute("SELECT id, name, mobile FROM myapp_customer WHERE name LIKE %s", ['%john%'])
        columns = [col[0] for col in cursor.description]
        rows = cursor.fetchall()
        data = [dict(zip(columns, row)) for row in rows]
    
    return JsonResponse({'data': data})
```

### Method 3: Complex Raw Query
```python
def home(request):
    # Complex JOIN query
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

### Method 4: Using `.extra()`
```python
def home(request):
    # Add custom SQL to QuerySet
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

### Method 1: Print SQL Query
```python
def home(request):
    # Get QuerySet
    queryset = Customer.objects.filter(name__icontains='john')
    
    # Print the SQL query
    query = str(queryset.query)
    print(query)
    
    # Execute and get results
    data = list(queryset.values())
    return JsonResponse({'query': query, 'data': data})
```

### Method 2: Using Django Debug Toolbar
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

### Method 3: Enable SQL Logging
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

### Method 4: Using connection.queries
```python
from django.db import connection, reset_queries

def home(request):
    reset_queries()  # Clear previous queries
    
    # Execute queries
    customers = Customer.objects.filter(name__icontains='john').values()
    products = Product.objects.filter(price__gt=100).values()
    
    # Get all executed queries
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

### Method 5: Explain Query Plan
```python
def home(request):
    # Get query execution plan
    queryset = Product.objects.filter(price__gt=100).order_by('-price')
    
    # For PostgreSQL
    explain = queryset.explain(verbose=True, analyze=True)
    
    return JsonResponse({'explain': explain})
```

---

## 9. Aggregation Operator Queries

Aggregation functions perform calculations on a set of values and return a single value.

### Average (Avg)
```python
from django.db.models import Avg

def home(request):
    # Calculate average price
    product = Product.objects.aggregate(Avg('price'))
    return HttpResponse(product['price__avg'])
```

### Minimum (Min)
```python
from django.db.models import Min

def home(request):
    # Get minimum price
    product = Product.objects.aggregate(Min('price'))
    return HttpResponse(product['price__min'])
```

### Maximum (Max)
```python
from django.db.models import Max

def home(request):
    # Get maximum price
    product = Product.objects.aggregate(Max('price'))
    return HttpResponse(product['price__max'])
```

### Sum
```python
from django.db.models import Sum

def home(request):
    # Calculate sum of all prices
    product = Product.objects.aggregate(Sum('price'))
    return HttpResponse(product['price__sum'])
```

### Count
```python
from django.db.models import Count

def home(request):
    # Count total products
    product = Product.objects.aggregate(Count('price'))
    return HttpResponse(product['price__count'])
```

### Multiple Aggregations
```python
from django.db.models import Avg, Max, Min, Sum, Count

def home(request):
    # Perform multiple aggregations at once
    product = Product.objects.aggregate(
        product_avg=Avg('price'),
        product_max=Max('price'),
        product_min=Min('price'),
        product_sum=Sum('price'),
        product_count=Count('price'),
    )
    return JsonResponse(product)
```

### Aggregation with Filtering
```python
def home(request):
    # Aggregate filtered records
    result = Product.objects.filter(
        price__gt=100
    ).aggregate(
        avg_price=Avg('price'),
        total_products=Count('id')
    )
    return JsonResponse(result)
```

### Group By with Aggregation (annotate)
```python
def home(request):
    # Group by category and calculate aggregates
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

Comparison operators allow you to filter records based on numeric or date comparisons.

### Equal (exact)
```python
def home(request):
    # Find products with exact price
    product = Product.objects.filter(price=500).values()
    return JsonResponse({'data': list(product)})
```

### Less Than (lt)
```python
def home(request):
    # Find products with price less than 500
    product = Product.objects.filter(price__lt=500).values()
    return JsonResponse({'data': list(product)})
```

### Greater Than (gt)
```python
def home(request):
    # Find products with price greater than 200
    product = Product.objects.filter(price__gt=200).values()
    return JsonResponse({'data': list(product)})
```

### Less Than or Equal (lte)
```python
def home(request):
    # Find products with price less than or equal to 200
    product = Product.objects.filter(price__lte=200).values()
    return JsonResponse({'data': list(product)})
```

### Greater Than or Equal (gte)
```python
def home(request):
    # Find products with price greater than or equal to 200
    product = Product.objects.filter(price__gte=200).values()
    return JsonResponse({'data': list(product)})
```

### Not Equal (exclude)
```python
def home(request):
    # Find products not equal to specific price
    product = Product.objects.exclude(price=500).values()
    return JsonResponse({'data': list(product)})
```

### Using Q Objects with OR
```python
from django.db.models import Q

def home(request):
    # Complex OR conditions
    product = Product.objects.filter(
        Q(name="Laptop") |
        Q(price__lt=2000) |
        Q(price__gt=50)
    ).values()
    
    return JsonResponse({'data': list(product)})
```

### Using Q Objects with AND
```python
from django.db.models import Q

def home(request):
    # Complex AND conditions
    product = Product.objects.filter(
        Q(name="Laptop") &
        Q(price__lt=2000) &
        Q(price__gt=50)
    ).values()
    
    return JsonResponse({'data': list(product)})
```

### Complex Q Object Combinations
```python
def home(request):
    # Combine AND and OR conditions
    product = Product.objects.filter(
        (Q(price__gte=100) & Q(price__lte=500)) |
        Q(name__icontains='premium')
    ).values()
    
    return JsonResponse({'data': list(product)})
```

### Negation with Q Objects
```python
def home(request):
    # Use NOT with Q objects
    product = Product.objects.filter(
        ~Q(price__lt=100) & Q(name__icontains='laptop')
    ).values()
    
    return JsonResponse({'data': list(product)})
```

---

## 11. String Operator Queries

String operators allow you to perform pattern matching and text searches.

### Contains (Case-Sensitive)
```python
def home(request):
    # Find products containing "pp" (case-sensitive)
    product = Product.objects.filter(name__contains="pp").values()
    return JsonResponse({'data': list(product)})
```

### iContains (Case-Insensitive)
```python
def home(request):
    # Find products containing "pp" (case-insensitive)
    product = Product.objects.filter(name__icontains="pp").values()
    return JsonResponse({'data': list(product)})
```

### Starts With
```python
def home(request):
    # Find products starting with "p"
    product = Product.objects.filter(name__startswith="p").values()
    return JsonResponse({'data': list(product)})
```

### iStarts With (Case-Insensitive)
```python
def home(request):
    # Find products starting with "p" (case-insensitive)
    product = Product.objects.filter(name__istartswith="p").values()
    return JsonResponse({'data': list(product)})
```

### Ends With
```python
def home(request):
    # Find products ending with "t"
    product = Product.objects.filter(name__endswith="t").values()
    return JsonResponse({'data': list(product)})
```

### iEnds With (Case-Insensitive)
```python
def home(request):
    # Find products ending with "t" (case-insensitive)
    product = Product.objects.filter(name__iendswith="t").values()
    return JsonResponse({'data': list(product)})
```

### String Operators with OR
```python
from django.db.models import Q

def home(request):
    # Combine string operators with OR
    product = Product.objects.filter(
        Q(name__startswith="p") |
        Q(name__endswith="t")
    ).values()
    return JsonResponse({'data': list(product)})
```

### String Operators with AND
```python
from django.db.models import Q

def home(request):
    # Combine string operators with AND
    product = Product.objects.filter(
        Q(name__startswith="p") &
        Q(name__endswith="t")
    ).values()
    return JsonResponse({'data': list(product)})
```

### Exact Match
```python
def home(request):
    # Exact string match
    product = Product.objects.filter(name__exact="Laptop").values()
    return JsonResponse({'data': list(product)})
```

### iExact (Case-Insensitive Exact)
```python
def home(request):
    # Case-insensitive exact match
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

## 12. Range and Membership Operator Queries

### Range Query (Inclusive)
```python
def home(request):
    # Find products with price between 10 and 100 (inclusive)
    product = Product.objects.filter(price__range=(10, 100)).values()
    return JsonResponse({'data': list(product)})
```

### Exclude Range
```python
def home(request):
    # Find products with price NOT between 100 and 400
    product = Product.objects.exclude(price__range=(100, 400)).values()
    return JsonResponse({'data': list(product)})
```

### IN Operator
```python
def home(request):
    # Find products with specific prices
    product = Product.objects.filter(price__in=[100, 200, 300]).values()
    return JsonResponse({'data': list(product)})
```

### Exclude IN
```python
def home(request):
    # Find products excluding specific prices
    product = Product.objects.exclude(price__in=[100, 200, 300]).values()
    return JsonResponse({'data': list(product)})
```

### IN with List of IDs
```python
def home(request):
    # Find products by list of IDs
    product_ids = [1, 5, 10, 15, 20]
    products = Product.objects.filter(id__in=product_ids).values()
    return JsonResponse({'data': list(products)})
```

### IN with Subquery
```python
def home(request):
    # Find products in specific categories
    category_ids = Category.objects.filter(name__icontains='electronics').values_list('id', flat=True)
    products = Product.objects.filter(category_id__in=category_ids).values()
    return JsonResponse({'data': list(products)})
```

### Date Range Query
```python
from datetime import datetime, timedelta

def home(request):
    # Find products created in last 7 days
    seven_days_ago = datetime.now() - timedelta(days=7)
    products = Product.objects.filter(
        created_at__range=(seven_days_ago, datetime.now())
    ).values()
    return JsonResponse({'data': list(products)})
```

### Complex Range with Q Objects
```python
from django.db.models import Q

def home(request):
    # Multiple range conditions
    products = Product.objects.filter(
        Q(price__range=(100, 500)) |
        Q(price__range=(1000, 2000))
    ).values()
    return JsonResponse({'data': list(products)})
```

---

## 13. Insert and Delete Operations

### Single Insert with create()
```python
def home(request):
    # Create single product
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

### Single Insert with save()
```python
def home(request):
    # Create product instance and save
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
    # Prepare list of products
    products = [
        Product(name='OpenAI', price=500, unit='per user', img_url='http://example.com/openai.png', category_id=1, user_id=1),
        Product(name='ChatGPT', price=600, unit='per user', img_url='http://example.com/chatgpt.png', category_id=1, user_id=1),
        Product(name='DALL-E', price=400, unit='per user', img_url='http://example.com/dalle.png', category_id=1, user_id=1),
        Product(name='Midjourney', price=350, unit='per user', img_url='http://example.com/midjourney.png', category_id=1, user_id=1),
    ]
    
    # Bulk create (much faster for multiple records)
    Product.objects.bulk_create(products)
    return JsonResponse({'message': 'Products Uploaded Successfully'})
```

### Insert with get_or_create()
```python
def home(request):
    # Create if not exists
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

### Delete Single Record
```python
def home(request):
    try:
        # Get and delete specific product
        product = Product.objects.get(id=26)
        product.delete()
        return JsonResponse({'message': 'Product Deleted Successfully'})
    except Product.DoesNotExist:
        return JsonResponse({'message': 'Product Not Found'}, status=404)
```

### Delete Multiple Records
```python
def home(request):
    # Delete all products with price less than 100
    deleted_count, _ = Product.objects.filter(price__lt=100).delete()
    return JsonResponse({
        'message': 'Products Deleted Successfully',
        'count': deleted_count
    })
```

### Delete All Records
```python
def home(request):
    # Delete all products
    deleted_count, _ = Product.objects.all().delete()
    return JsonResponse({
        'message': 'All Products Deleted',
        'count': deleted_count
    })
```

---

## 14. Update and Upsert Operations

### Update Single Record
```python
def home(request):
    try:
        # Get and update specific product
        product = Product.objects.get(id=28)
        product.name = "New_Name"
        product.price = 120
        product.unit = "Per Hour"
        product.save()
        
        return JsonResponse({'message': 'Product Updated Successfully'})
    except Product.DoesNotExist:
        return JsonResponse({'message': 'Product Not Found'}, status=404)
```

### Update Multiple Records
```python
def home(request):
    # Update all products in a category
    updated_count = Product.objects.filter(
        category_id=1
    ).update(
        price=F('price') * 1.1  # Increase price by 10%
    )
    
    return JsonResponse({
        'message': 'Products Updated Successfully',
        'count': updated_count
    })
```

### Update with F() Expression
```python
from django.db.models import F

def home(request):
    # Update price using current value
    Product.objects.filter(id=10).update(
        price=F('price') + 50  # Add 50 to current price
    )
    return JsonResponse({'message': 'Price Updated Successfully'})
```

### Conditional Update
```python
from django.db.models import Case, When, Value, IntegerField

def home(request):
    # Update based on conditions
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

### Upsert (Update or Create)
```python
def home(request):
    try:
        # Update if exists, create if not
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
    # Update multiple records efficiently
    products = Product.objects.filter(category_id=1)
    
    for product in products:
        product.price = product.price * 1.2
    
    # Bulk update (Django 2.2+)
    Product.objects.bulk_update(products, ['price'])
    
    return JsonResponse({'message': 'Bulk Update Completed'})
```

### Update with select_for_update() (Locking)
```python
from django.db import transaction

def home(request):
    # Lock row during update to prevent race conditions
    with transaction.atomic():
        product = Product.objects.select_for_update().get(id=10)
        product.price = 500
        product.save()
    
    return JsonResponse({'message': 'Product Updated with Lock'})
```

---

## 15. Relationship Joins, Reverse and Direct Relationships

### Understanding Relationships

#### Direct Relationship (Forward)
When a model has a ForeignKey to another model, you can access the related object directly.

```python
# Product has ForeignKey to Category
product = Product.objects.get(id=1)
category_name = product.category.name  # Direct access
```

#### Reverse Relationship (Backward)
When accessing related objects from the "other side" of the relationship.

```python
# Category to Products (reverse)
category = Category.objects.get(id=1)
products = category.product_set.all()  # Reverse access
```

### Direct Relationship Query
```python
def home(request):
    # Access related object (ForeignKey)
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
    # Access related objects from parent
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
# In models.py
class Product(models.Model):
    category = models.ForeignKey(
        Category,
        on_delete=models.CASCADE,
        related_name='products'  # Custom name instead of product_set
    )

# In views.py
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
    # Get student with all courses
    student = Student.objects.prefetch_related('courses').get(id=1)
    courses = student.courses.all()
    
    # Get course with all students (reverse)
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
    # Access nested relationships
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

## 16. INNER JOIN and LEFT JOIN Queries

### INNER JOIN with select_related()
```python
def home(request):
    # INNER JOIN: Only products with category
    user = User.objects.filter().select_related('categories').values(
        'id', 'username',
        'categories__id',
        'categories__name'
    )
    
    # Get SQL query
    query = str(user.query)
    
    return JsonResponse({
        'query': query,
        'data': list(user)
    })
```

### SQL Generated by select_related():
```sql
SELECT 
    user.id,
    user.username,
    categories.id,
    categories.name
FROM myapp_user AS user
INNER JOIN myapp_category AS categories ON (user.categories_id = categories.id)
```

### LEFT JOIN with Annotations
```python
from django.db.models import Count, OuterRef, Subquery

def home(request):
    # LEFT JOIN: All categories, even without products
    categories = Category.objects.annotate(
        product_count=Count('product')
    ).values('id', 'name', 'product_count')
    
    return JsonResponse({'data': list(categories)})
```

### Multiple JOINs
```python
def home(request):
    # Multiple INNER JOINs
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

### LEFT JOIN with prefetch_related()
```python
def home(request):
    # LEFT JOIN for reverse relationships
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

### Custom JOIN with extra()
```python
def home(request):
    # Custom JOIN condition
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
    # Get categories that have products
    has_products = Product.objects.filter(category_id=OuterRef('id'))
    
    categories = Category.objects.annotate(
        has_products=Exists(has_products)
    ).filter(has_products=True).values()
    
    return JsonResponse({'data': list(categories)})
```

### Performance: select_related vs prefetch_related

#### select_related (INNER JOIN)
- Use for ForeignKey and OneToOne relationships
- Creates SQL JOIN
- Single database query
- More efficient for small related sets

```python
# One query with JOIN
products = Product.objects.select_related('category').all()
for product in products:
    print(product.category.name)  # No additional query
```

#### prefetch_related (Separate queries)
- Use for ManyToMany and reverse ForeignKey relationships
- Creates separate queries and joins in Python
- Multiple database queries
- More efficient for large related sets

```python
# Two queries: one for categories, one for products
categories = Category.objects.prefetch_related('product_set').all()
for category in categories:
    for product in category.product_set.all():  # No additional query per product
        print(product.name)
```

### Complex JOIN Example
```python
from django.db.models import Prefetch

def home(request):
    # Complex prefetch with filtering
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

## Best Practices

### 1. Use select_related and prefetch_related
Avoid N+1 query problems by using these methods appropriately.

### 2. Use values() and values_list() for Large Datasets
When you don't need full model instances, use these methods to reduce memory usage.

### 3. Use exists() Instead of count()
When checking if records exist:
```python
# Bad
if Product.objects.filter(name='Laptop').count() > 0:
    pass

# Good
if Product.objects.filter(name='Laptop').exists():
    pass
```

### 4. Use iterator() for Large QuerySets
```python
for product in Product.objects.all().iterator():
    # Process product
    pass
```

### 5. Use bulk_create and bulk_update
For multiple records, always use bulk operations.

### 6. Use F() for Database-Level Operations
```python
# Update in database, not Python
Product.objects.filter(id=1).update(price=F('price') * 1.1)
```

### 7. Use Q Objects for Complex Queries
Combine multiple conditions cleanly with Q objects.

### 8. Index Foreign Keys and Frequently Queried Fields
```python
class Product(models.Model):
    name = models.CharField(max_length=200, db_index=True)
```

### 9. Use only() and defer() to Limit Fields
```python
# Load only specific fields
products = Product.objects.only('name', 'price')

# Load all except specific fields
products = Product.objects.defer('description', 'img_url')
```

### 10. Monitor Query Performance
Use Django Debug Toolbar and database query logs to identify slow queries.

---

## Common Pitfalls

### 1. N+1 Query Problem
```python
# Bad - Creates N+1 queries
products = Product.objects.all()
for product in products:
    print(product.category.name)  # Additional query per product

# Good - Single query with JOIN
products = Product.objects.select_related('category').all()
for product in products:
    print(product.category.name)
```

### 2. Avoiding get() Exceptions
```python
# Bad - Can raise exception
product = Product.objects.get(id=999)

# Good - Use try-except or filter().first()
try:
    product = Product.objects.get(id=999)
except Product.DoesNotExist:
    product = None

# Or
product = Product.objects.filter(id=999).first()
```

### 3. Using count() When Not Needed
```python
# Bad - Executes query twice
if Product.objects.filter(price__gt=100).count() > 0:
    products = Product.objects.filter(price__gt=100)

# Good - Execute once
products = Product.objects.filter(price__gt=100)
if products.exists():
    # Use products
```

---

## Conclusion

Django ORM provides a powerful, Pythonic way to interact with databases. By mastering these patterns and following best practices, you can write efficient, maintainable database queries without writing raw SQL.

### Key Takeaways:
- Use QuerySet methods for cleaner code
- Leverage select_related and prefetch_related for performance
- Understand when to use aggregation, annotation, and filtering
- Monitor query performance and optimize when needed
- Follow Django conventions for better maintainability

### Further Learning:
- Django Official Documentation: https://docs.djangoproject.com/en/stable/topics/db/queries/
- Django ORM Cookbook: https://books.agiliq.com/projects/django-orm-cookbook/
- Django Debug Toolbar: https://django-debug-toolbar.readthedocs.io/

---

**Created with ❤️ for Django Developers**