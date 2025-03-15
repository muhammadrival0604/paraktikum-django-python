
## **1. Install Django dan MySQL Client**  
Pastikan Python sudah terinstall. Jika belum, unduh dari [python.org](https://www.python.org/downloads/).

Lalu, buat dan aktifkan virtual environment (opsional, tapi disarankan):  
```sh
python -m venv venv
source venv/bin/activate  # Mac/Linux
venv\Scripts\activate  # Windows
```

Instal **Django** dan **MySQL client**:
```sh
pip install django mysqlclient
```

---

## **2. Buat Project Django**  
Buat project baru:  
```sh
django-admin startproject mycrud_django
cd mycrud_django
```

Buat aplikasi dalam proyek:  
```sh
python manage.py startapp mycrud_app
```

---

## **3. Konfigurasi `settings.py`**
Buka file `settings.py` dalam folder `mycrud_django` dan ubah beberapa bagian:

### **A. Tambahkan `mycrud_app` di `INSTALLED_APPS`**  
Tambahkan aplikasi yang baru dibuat:
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'mycrud_app',  # Tambahkan ini
]
```

### **B. Konfigurasi Database MySQL**
Ganti pengaturan database di `settings.py`:
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'db_siswa',  # Ganti dengan nama database
        'USER': 'root',       # Sesuaikan dengan user MySQL
        'PASSWORD': '',       # Masukkan password MySQL
        'HOST': 'localhost',  # Default: localhost
        'PORT': '3306',       # Default: 3306
    }
}
```

> **Pastikan database `db_siswa` sudah dibuat di MySQL sebelum menjalankan migrasi.**

### **C. Konfigurasi Template**
Tambahkan pengaturan folder **templates** agar Django bisa menemukan file HTML:  
```python
import os

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'mycrud_app', 'templates')],  # Tambah ini
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

---

## **4. Buat Model untuk `Siswa`**
Buka `models.py` di `mycrud_app`, lalu buat model `Siswa`:
```python
from django.db import models

class Siswa(models.Model):
    nisn = models.CharField(max_length=10, unique=True)
    nama = models.CharField(max_length=100)
    kelas = models.CharField(max_length=10)
    
    def __str__(self):
        return self.nama
```
Jalankan migrasi:
```sh
python manage.py makemigrations
python manage.py migrate
```

---

## **5. Buat Views untuk CRUD**
Buka `views.py` di `mycrud_app` dan tambahkan fungsi berikut:

```python
from django.shortcuts import render, redirect
from .models import Siswa

def index(request):
    siswa = Siswa.objects.all()
    return render(request, 'index.html', {'siswa': siswa})

def tambah(request):
    if request.method == "POST":
        nisn = request.POST['nisn']
        nama = request.POST['nama']
        kelas = request.POST['kelas']
        Siswa.objects.create(nisn=nisn, nama=nama, kelas=kelas)
        return redirect('/')
    return render(request, 'tambah.html')

def hapus(request, id):
    siswa = Siswa.objects.get(id=id)
    siswa.delete()
    return redirect('/')

def edit(request, id):
    siswa = Siswa.objects.get(id=id)
    if request.method == "POST":
        siswa.nisn = request.POST['nisn']
        siswa.nama = request.POST['nama']
        siswa.kelas = request.POST['kelas']
        siswa.save()
        return redirect('/')
    return render(request, 'edit.html', {'siswa': siswa})
```

---

## **6. Buat Routing di `urls.py`**
Buka `mycrud_app/urls.py`, lalu tambahkan:
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name="index"),
    path('tambah/', views.tambah, name="tambah"),
    path('hapus/<int:id>/', views.hapus, name="hapus"),
    path('edit/<int:id>/', views.edit, name="edit"),
]
```
Lalu, tambahkan routing ini ke `mycrud_django/urls.py`:
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('mycrud_app.urls')),
]
```

---

## **7. Buat Folder `templates` dan File HTML**
Buat folder `templates` di dalam `mycrud_app`, lalu buat tiga file berikut:

### **A. `index.html` (Menampilkan Data)**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Data Siswa</title>
</head>
<body>
    <h1>Daftar Siswa</h1>
    <a href="/tambah/">Tambah Siswa</a>
    <table border="1">
        <tr>
            <th>NISN</th>
            <th>Nama</th>
            <th>Kelas</th>
            <th>Aksi</th>
        </tr>
        {% for s in siswa %}
        <tr>
            <td>{{ s.nisn }}</td>
            <td>{{ s.nama }}</td>
            <td>{{ s.kelas }}</td>
            <td>
                <a href="/edit/{{ s.id }}">Edit</a> |
                <a href="/hapus/{{ s.id }}">Hapus</a>
            </td>
        </tr>
        {% endfor %}
    </table>
</body>
</html>
```

### **B. `tambah.html` (Tambah Data)**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Tambah Siswa</title>
</head>
<body>
    <h1>Tambah Siswa</h1>
    <form method="post">
        {% csrf_token %}
        NISN: <input type="text" name="nisn"><br>
        Nama: <input type="text" name="nama"><br>
        Kelas: <input type="text" name="kelas"><br>
        <button type="submit">Simpan</button>
    </form>
</body>
</html>
```

### **C. `edit.html` (Edit Data)**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Edit Siswa</title>
</head>
<body>
    <h1>Edit Siswa</h1>
    <form method="post">
        {% csrf_token %}
        NISN: <input type="text" name="nisn" value="{{ siswa.nisn }}"><br>
        Nama: <input type="text" name="nama" value="{{ siswa.nama }}"><br>
        Kelas: <input type="text" name="kelas" value="{{ siswa.kelas }}"><br>
        <button type="submit">Update</button>
    </form>
</body>
</html>
```

---

## **8. Jalankan Server**
```sh
python manage.py runserver
```
Buka di browser:  
👉 `http://127.0.0.1:8000/`

---
