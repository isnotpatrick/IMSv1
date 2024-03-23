# Inventory Management System v1

<sub>*last updated: 2024-03-22*</sub>
<br>
<sub>*comments: Django with SQLite*</sub>

A simplified step-by-step guide to building a basic inventory management system as a web application using Python and the Django framework. This example will cover the setup of a Django project, creating models for inventory management, implementing CRUD operations, and setting up basic views for managing inventory items.

Please note that this example will be quite basic and may not cover all possible features or edge cases. Additionally, it's assumed that you have Python and Django installed on your system. If not, you can install them using pip:

```
pip install django
```

Let's get started:

Step 1: Create a Django Project

```bash
django-admin startproject inventory_management
cd inventory_management
```

Step 2: Create a Django App for Inventory

```bash
python manage.py startapp inventory
```

Step 3: Define Models for Inventory Management
<br>
Edit **'inventory/models.py'**:

```python
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField(blank=True)
    quantity = models.PositiveIntegerField(default=0)
    unit_price = models.DecimalField(max_digits=10, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        app_label = 'inventory'

    def __str__(self):
        return self.name
```

Step 4: Register the Model in Admin Panel
<br>
Edit **'inventory/admin.py'**:

```python
from django.contrib import admin
from .models import Product

admin.site.register(Product)
```

Step 5: Set Up URL Routing
<br>
Edit **'inventory_management/urls.py'**:

```python
from django.contrib import admin
from django.urls import path, include, re_path
from django.http import HttpResponse

def ignore_favicon(request):
    return HttpResponse(status=204)

urlpatterns = [
    path("admin/", admin.site.urls),
    path('', include('inventory.urls')),
    # Ignore favicon requests in URLs
    re_path(r'^favicon\.ico$', ignore_favicon),
]
```

Step 6: Define URLs for Inventory App
<br>
Create **'inventory/urls.py'**:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.product_list, name='product_list'),
    path('product/<int:pk>/', views.product_detail, name='product_detail'),
    path('product/new/', views.product_create, name='product_create'),
    path('product/<int:pk>/edit/', views.product_edit, name='product_edit'),
    path('product/<int:pk>/delete/', views.product_delete, name='product_delete'),
]
```

Step 7: Implement Views for CRUD Operations
<br>
Edit **'inventory/views.py'**:

```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Product
from .forms import ProductForm

def product_list(request):
    products = Product.objects.all()
    return render(request, 'inventory/product_list.html', {'products': products})

def product_detail(request, pk):
    product = get_object_or_404(Product, pk=pk)
    return render(request, 'inventory/product_detail.html', {'product': product})

def product_create(request):
    if request.method == 'POST':
        form = ProductForm(request.POST)
        if form.is_valid():
            product = form.save()
            return redirect('product_detail', pk=product.pk)
    else:
        form = ProductForm()
    return render(request, 'inventory/product_form.html', {'form': form})

def product_edit(request, pk):
    product = get_object_or_404(Product, pk=pk)
    if request.method == 'POST':
        form = ProductForm(request.POST, instance=product)
        if form.is_valid():
            product = form.save()
            return redirect('product_detail', pk=product.pk)
    else:
        form = ProductForm(instance=product)
    return render(request, 'inventory/product_form.html', {'form': form})

def product_delete(request, pk):
    product = get_object_or_404(Product, pk=pk)
    if request.method == 'POST':
        product.delete()
        return redirect('product_list')
    return render(request, 'inventory/product_confirm_delete.html', {'product': product})
```

Step 8: Create HTML Templates
<br>
HTML templates will be saved in **'inventory/templates/inventory directory'**:

Create **'product_list.html'**:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Product List</title>
</head>
<body>
    <h1>Product List</h1>
    <a href="{% url 'product_create' %}">Add Product</a>
    <table border="1">
        <thead>
            <tr>
                <th>Name</th>
                <th>Description</th>
                <th>Quantity</th>
                <th>Unit Price</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            {% for product in products %}
            <tr>
                <td>{{ product.name }}</td>
                <td>{{ product.description }}</td>
                <td>{{ product.quantity }}</td>
                <td>{{ product.unit_price }}</td>
                <td>
                    <a href="{% url 'product_detail' pk=product.pk %}">View</a>
                    <a href="{% url 'product_edit' pk=product.pk %}">Edit</a>
                    <a href="{% url 'product_delete' pk=product.pk %}">Delete</a>
                </td>
            </tr>
            {% empty %}
            <tr>
                <td colspan="5">No products available.</td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
</body>
</html>
```

Create **'product_detail.html'**:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Product Detail</title>
</head>
<body>
    <h1>Product Detail</h1>
    <p><strong>Name:</strong> {{ product.name }}</p>
    <p><strong>Description:</strong> {{ product.description }}</p>
    <p><strong>Quantity:</strong> {{ product.quantity }}</p>
    <p><strong>Unit Price:</strong> {{ product.unit_price }}</p>
    <p><strong>Created At:</strong> {{ product.created_at }}</p>
    <p><strong>Updated At:</strong> {{ product.updated_at }}</p>
    <a href="{% url 'product_list' %}">Back to Product List</a>
</body>
</html>
```

Create **'product_form.html'**:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% if form.instance.pk %}Edit Product{% else %}Add Product{% endif %}</title>
</head>
<body>
    <h1>{% if form.instance.pk %}Edit Product{% else %}Add Product{% endif %}</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Save</button>
    </form>
</body>
</html>
```

Create **'product_confirm_delete.html'**:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Confirm Deletion</title>
</head>
<body>
    <h1>Confirm Deletion</h1>
    <p>Are you sure you want to delete the product "{{ product.name }}"?</p>
    <form method="post">
        {% csrf_token %}
        <button type="submit">Confirm Delete</button>
        <a href="{% url 'product_detail' pk=product.pk %}">Cancel</a>
    </form>
</body>
</html>
```

Step 9: Implement Forms for Product
<br>
Create **'inventory/forms.py'**:

```python
from django import forms
from .models import Product

class ProductForm(forms.ModelForm):
    class Meta:
        model = Product
        fields = ['name', 'description', 'quantity', 'unit_price']
```

Step 10: Include app in project settings
<br>
Update **'inventory_management/settings.py'**

```python
INSTALLED_APPS = [
    ...
    'inventory',
]
```

Step 11: Migrate Database Changes

```bash
python manage.py makemigrations
python manage.py migrate
```

Step 11: Run Development Server

```bash
python manage.py runserver
```

Now, you should be able to access the inventory management system in your web browser at 'http://127.0.0.1:8000/'. You can perform CRUD operations on products through the provided views and templates.

Please note that this is a basic example, and you may need to customize and expand upon it to fit your specific requirements and use cases. Additionally, it's important to consider security, validation, and error handling when developing a production-level application.