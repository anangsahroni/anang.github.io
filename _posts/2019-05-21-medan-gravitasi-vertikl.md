---
layout: post
title: Medan Gravitasi Vertikal akibat Titik Massa
date: 2019-05-21 16:00:00
description: Pengaplikasian Python untuk plot Medan Gravitasi Titik Massa, digenerasi ulang dari Gravity and Magnetic Exploration -  Principles, Practices, and Applications (Hinze et al., 2013)
tags: gravity, python
categories: geophysics
featured: true
related_publications: hinze2013

---
> The post is currently still in Bahasa Indonesia and will be translated to English soon

> Konten ini juga sebagai bahan belajar saya, kalau ada kesalahan bisa dikoreksi, _thanks :)_


Akhirnya setelah sekian lama berkutat dengan web berbagai event/organisasi tanpa punya blog sendiri, sekarang punya _hehehe_... ya semoga konsisten bisa posting macam-macam minimal seminggu sekali. Terima kasih pada GitHub dan Jekyll yang memberikan ruang serta _tools_ yang luar bisa _customable_.

Belakangan ini saya memang tertarik untuk mendalami bahasa pemrograman `Python`, bukan tanpa alasan sih, tuntutan duniawai agar bisa lebih _survive_ di dunia _hahaha_. Bersamaan baca buku [Gravity and Magnetic Exploration: Principles, Practices, and Applications](https://www.cambridge.org/core/books/gravity-and-magnetic-exploration/20BABB4AAFCDF8DC7F4CB36B8359D482) karya William J. Hinze, Ralph R. B. von Frese and Afif H. Saad (2012), saya tertarik untuk belajar cara membuat peta magnitudo medan gravitasi. _Sebenernya_ cukup sederhana, tapi untuk _newbie_ macam saya ya kaya begini cukup menyulitkan, pada akhirnya saya banyak belajar karena _kepo_ sana-sini.

>Berawal dari coba-coba akhirnya keterusan, oposih. skip.

Saya _sih_ berharap ada pembaca yang terinspirasi, terutama pembaca yang _newbie_ `Python` sehingga bisa ikut belajar, buat para _mastah_ juga ikut membantu memberikan saran _hehehe_. Kode-kode di bawah ini sebenarnya sederhana dan banyak simplifikasi, misal dalam penurunan, kedepan semoga bisa saya sempurnakan. Untuk para _geopipisis_ semoga konten saya dapat menginspirasi dan menambah ilmu tentang per-_gravity_-an.  

Oke _to the point_, jadi ini adalah cara menghitung besar medan gravitasi ($ \boldsymbol{g} $) dari titik massa $ m $  kg  di kedalaman $ d $ meter komponen $ x $, $ y $, maupun $ z $ dari penurunan potensial gravitasi $ U $. Rumus potensial gravitasi sendiri [1]:

$$ U = G \frac{m} {r} $$

dengan $ G $ : konstanta gravitasi, $ m $ : massa, dan $ r $ : jarak. Jarak ($ r $) sendiri merupakan $ \sqrt{x^2 + y^2 + z^2} $ , kok malah jadi _sepaneng gini ya wehehehe_. Nah ini contoh untuk komponen $ z $ [2]:

$$ \boldsymbol{g_{z}} = \frac {\partial U}{\partial z} $$

Nah disini saya ingin memberikan cara menghitung penurunan (derivatif) dengan rumus turunan sederhana yaitu [3]:

$$ \boldsymbol{g_{z}} = \lim_{\Delta z \to 0} \frac {U(x,y,z+ \Delta z)-U(x,y,z)}{\Delta z} $$

_(mohon dikoreksi kalo salah ya hehehe)_  
Nah karena ini pake diskrit di `Python` jadi saya pilih saja $ \Delta z $ sekecil mungkin mendekati 0. Ya walaupun sebenarnya langsung bisa pakai rumus [4]:

$$ \boldsymbol{g_{z}} = -G \frac{m \Delta z} {r^3} $$

Tapi kita coba pakai penurunan sendiri biar lebih menantang _hehehe_, semoga _bener_ ya.

Nah langsung saja praktek:
  
#### 1. Impor dulu modul dan konstanta yang dipake.
Disini saya pakai massa dan kedalaman tertera, silahkan kalo mau diganti, _bebas pokoke_.
```python
import numpy as np
import matplotlib.pyplot as plt

# define some constants
G = 6.67408e-11 # gravitational constant
m = 1e12  # mass of a point mass in kg
d = -500 # depth of a point mass in m
```  
Massa yang digunakan ada 10^12 kg dan kedalaman 500 meter, nilai ini saya sesuaikan dengan referensi buku Gravity and Magnetic Exploration, teman-teman bisa ganti _monggo_.
  
#### 2. Selanjutnya adalah membuat sebaran datanya.
Pertama membuat sebaran koordinat dulu:
```python
# create x, y, z sparse
x = np.arange(-1500, 1500, 50)
y = np.arange(-1500, 1500, 50)
z = np.arange(-500, 10, 10)
```  
Sebaran yang saya pakai adalah -1500-1450 meter untuk `x` dan `y`, dan -500-0 meter untuk `z`.
Baru abis itu dibuat _grid_ jadi _3D array_:
```python
# create X, Y, Z grid
X, Y, Z = np.meshgrid(x, y, z)
```  
Buat teman-teman _geopipisis_ pastinya sudah akrab dengan istilah _grdding_.
#### 3. Lanjut mendefinisikan fungsi menghitung potensial $ U $
Membuat fungsi sederhana untuk menghitung potensial gravitasi:
```python
# define function for gravitational potential
def grav_potential(xi, yi, zi, m, d):
 "Calculate grav potential in a point in 3D space due to mass m in d depth, ex: grav_potential(X, Y, Z, 100, 5)"
 r = np.sqrt(xi**2 + yi**2 + (zi-d)**2)
 r[r==0]=np.nan #avoid dividing by zero
 pot = G * m / r
 return pot
```  
Yang perlu diperhatikan adalah adanya `zi-d` pada operasi untuk menghitung jarak dari _point mass_. `r[r==0]=np.nan` digunakan untuk menghilangkan data di `r=0` (di titik massa) agar tidak terjadi pembagian dengan 0 (`ZeroDivisionError`). 
#### 4. Kemudian mendefinisikan fungsi derivatif terhadap $ z $ (menghitung medan)
Menurunkan potensial gravitas menjadi medan gravitasi komponen $ z $ menggunakan persamaan derivatif yang ada di atas [3].
```python
#define function for derivative
dz = 0.000001
def grav_field(Xi, Yi, Zi):
    "Calculate derivative in certain component, ex: grav_field(X, Y, Z+dz) --> Z component"
    field = ((grav_potential(Xi, Yi, Zi, m, d) - grav_potential(X, Y, Z, m, d)) / dz)
    return field
```  
Saya menggunakan nilai $ \Delta z $ (`dz = 0.000001`) yang saya anggap sudah mendekati nilai `0`.
#### 5. Langsung panggil derivatif terhadap $ z $
nb. kalo mau terhadap komponen lain tinggal dz nya dipindah hehew, dikalikan `1e5` hanya biar skala warnanya masuuk..
```python
# call the derivative
grav_z = grav_field(X, Y, Z+dz) * 1e5
```  
Nilai `1e5` nanti akan dikembalikan lagi di keterangan warna _(colorbar)_.
#### 6. Mari diplot di kedalaman=0.
Kalau mau kedalaman lain nilai `-1` tinggal diganti, itu dari -500 sampai 0 lompatan 10. Contoh kalo kedalaman -450 berarti `1`, -500 berarti `0`, -10 berarti `-2` dsb.
```python
plt.contourf(X[:,:,-1], Y[:,:,-1], grav_z[:,:,-1])
```  
  
#### 7. Final, masukkan parametter _plotting_
```python
plt.colorbar()
plt.text(1600, 1600, "x 10^-5 m/s^2")
plt.xlabel("Easting (m)")
plt.ylabel("Northing (m)")
plt.title("Gravitational field in Z comp at 0 m of a point \n mass (1e12 kg) located at 500 m depth")
plt.show()
```  
Dan hasilnya adalah...
![$ g_{z} $](/static/images/posts/grav_z.jpg){: .img-fluid }

Halaman ini akan saya sempurnakan lagi untuk kedepannya, untuk kalibrasi hasil bisa dilakukan dengan memasukkan `m` menjadi massa bumi dan `d` menjadi diameter bumi, walaupun belum memperhitungkan kelengkungan tetapi sudah cukup untuk melihat nilai medan gravitasi. Thank you for reading :)
