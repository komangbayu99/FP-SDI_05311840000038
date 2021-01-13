# Software Deteksi Intrusi

<hr>


## Libraries Yang Digunakan

- Numpy
- cv2

## Bagaimana Cara Menjalankannya

jalankan file Change detection.py untuk melihat cara kerjanya pada contoh video

## Deskripsi

sistem perangkat lunak yang, berdasarkan analisis video otomatis, dapat mendeteksi objek (penyusup) yang tidak termasuk dalam adegan referensi statis (latar belakang) dan menentukan objek mana yang merupakan orang.


![image](https://user-images.githubusercontent.com/56583448/90742914-de113a00-e2cf-11ea-8c91-fd3e114d277b.png)


Untuk setiap frame video, program menyediakan yang berikut ini:

- Nomor frame
- Jumlah objek (penyusup) yang terdeteksi

Untuk setiap objek:

- Area
- Perimeter
- Klasifikasi (orang, Benda mati(objek))

Semua fitur ini diberikan sebagai output file teks.



![image](https://user-images.githubusercontent.com/56583448/90743618-0e58d880-e2d0-11ea-8630-b9613076b422.png)
