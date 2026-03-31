# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. Dalam kasus BambangShop, sebuah struct tunggal sebenarnya cukup jika kita hanya memandang subscriber sebagai entitas data (menyimpan url dan name). Saat ini hanya mengirim notifikasi via HTTP POST ke URL tertentu yang dimana perilakunya seragam. Namun, secara arsitektur, kita tetap membutuhkan abstraksi (di Rust disebut Trait) untuk fleksibilitas jangka panjang jika kita ingin mendukung berbagai jenis subscriber di masa depan. Jika ke depannya aplikasi ingin mendukung jenis notifikasi lain seperti pengiriman melalui Email atau SMS, kita butuh Trait Observer dengan fungsi update(). Dengan Trait, Publisher tidak perlu peduli bagaimana cara notifikasi dikirim, ia hanya perlu memanggil fungsi update() tersebut.

2. Penggunaan DashMap dalam menyimpan data Subscriber jauh lebih efisien dibandingkan dengan Vec atau list biasa karena kebutuhan akan keunikan data dan kecepatan akses. Mengingat url harus bersifat unik untuk setiap Subscriber, DashMap memungkinkan kita untuk melakukan pencarian, pembaruan, dan penghapusan data dengan kompleksitas waktu rata-rata $O(1)$ tanpa perlu menyisir seluruh elemen satu per satu seperti pada Vec yang memiliki kompleksitas $O(n)$. Selain itu, struktur data map yang dipetakan berdasarkan product_type mempermudah pengelompokan Subscriber sehingga proses distribusi notifikasi menjadi lebih terarah dan sistematis sesuai dengan kategori produk.

3. Meskipun pola Singleton yang kita buat menggunakan lazy_static! sudah menjamin bahwa hanya ada satu instansi daftar subscriber di seluruh program, pola tersebut tidak secara otomatis menjamin keamanan saat data diakses oleh banyak pengguna sekaligus (thread-safety). Dalam lingkungan multithreading seperti aplikasi web ini, beberapa proses bisa saja mencoba membaca atau mengubah data di waktu yang bersamaan, yang berisiko menyebabkan race condition atau error. Oleh karena itu, kita tetap membutuhkan DashMap karena ia dirancang khusus untuk menangani akses bersamaan tersebut secara aman tanpa perlu mengelola penguncian (locking) manual yang rumit. Jadi, Singleton dan DashMap bekerja saling melengkapi: Singleton memastikan objeknya cuma satu, sementara DashMap memastikan objek yang satu itu aman digunakan bersama-sama.

#### Reflection Publisher-2
1. Pemisahan antara Service dan Repository dari Model sangat penting untuk menjaga prinsip Single Responsibility Principle (SRP). Dalam arsitektur modern, Model idealnya hanya bertugas sebagai representasi data atau struktur objek murni(DTO). Dengan memisahkan Repository, kita mengisolasi logika akses data dan bisnis, seperti bagaimana data disimpan ke dalam DashMap atau database. Sementara itu, Service bertindak sebagai jembatan yang mengatur alur logika bisnis dan koordinasi antar-objek. Pemisahan ini memudahkan pengujian kode (unit testing), karena kita bisa menguji logika bisnis di Service tanpa harus bergantung pada implementasi penyimpanan data di Repository. Hal ini memungkinkan pengembangan yang lebih fleksibel dan mudah untuk dimaintain apabila terdapat perubahan pada kode tanpa mengganggu bagian kode yang lain.

2. Jika kita hanya menggunakan Model untuk menangani segalanya, kode akan menjadi sangat kompleks dan sulit dikelola menyebabkan Fat Model. Jika model Product harus menyimpan daftar Subscriber, sekaligus memiliki logika untuk mengirimkan Notification ke setiap URL. Hal ini akan menyebabkan ketergantungan yang sangat tinggi (high coupling) antar-model. Perubahan kecil pada struktur data notifikasi bisa merusak logika penyimpanan produk. Kompleksitas akan meningkat secara eksponensial karena setiap model memikul beban tanggung jawab yang tumpang tindih, sehingga sulit untuk melakukan perubahan fitur atau melacak sumber kesalahan (bug) tanpa memengaruhi seluruh sistem.

3. Eksplorasi terhadap Postman menunjukkan bahwa alat ini sangat krusial dalam mempercepat siklus pengembangan perangkat lunak, terutama dalam memvalidasi logika backend secara terisolasi tanpa harus menunggu implementasi frontend selesai. Fitur seperti Collections sangat membantu dalam mengorganisir berbagai request berdasarkan modul proyek, sementara kemampuan untuk mengontrol Headers, Body Request, dan Cookies memberikan fleksibilitas penuh dalam mensimulasikan berbagai skenario pengguna secara cepat dan efisien. Untuk pengerjaan proyek di masa depan, saya sangat tertarik dengan fitur API Documentation dan Tests untuk skrip validasi otomatis yang sangat mendukung kolaborasi tim dalam menjaga konsistensi kontrak API yang telah disepakati.

#### Reflection Publisher-3
1. Berdasarkan implementasi kode pada tutorial ini, variasi pola Observer yang kita gunakan adalah Push Model. Hal ini terlihat pada metode notify di NotificationService dan metode update di model Subscriber, di mana Publisher secara aktif mengirimkan payload data lengkap (berisi nama produk, tipe, URL, hingga status) kepada setiap subscriber melalui permintaan HTTP POST segera setelah terjadi perubahan status produk (seperti saat produk dibuat, dipromosikan, atau dihapus). Dalam model ini, subscriber tidak perlu mengetahui kondisi internal publisher atau meminta data(pull), mereka cukup menunggu dan menerima informasi dari publisher.

2. Jika kita menggunakan variasi Pull Model, keuntungannya adalah subscriber memiliki kendali penuh atas kapan mereka ingin mengambil data, sehingga beban jaringan bisa lebih terdistribusi jika subscriber sedang sibuk. Namun, kerugiannya sangat signifikan untuk kasus ini, subscriber harus terus-menerus melakukan polling (bertanya berulang kali) ke publisher untuk mengecek apakah ada pembaruan, yang berisiko membuang-buang sumber daya bandwidth dan CPU jika ternyata tidak ada perubahan. Selain itu, informasi yang diterima subscriber mungkin tidak lagi real-time karena ada jeda waktu antara terjadinya perubahan di publisher dengan waktu polling berikutnya yang dilakukan oleh subscriber.

3. Apabila kita memutuskan untuk tidak menggunakan multi-threading dalam proses pengiriman notifikasi, program akan menjalankan proses tersebut secara sekuensial atau berurutan. Dampaknya, jika terdapat banyak subscriber atau jika salah satu server subscriber merespon sangat lambat (mengalami latency tinggi), proses utama di publisher akan tertahan sampai semua notifikasi selesai dikirim. Hal ini akan memperburuk pengalaman pengguna karena operasi sederhana seperti membuat atau menghapus produk akan memakan waktu yang sangat lama, sementara dengan multi-threading, setiap notifikasi dikirim di latar belakang sehingga publisher bisa langsung memberikan respon kepada pengguna tanpa harus menunggu proses pengiriman notifikasi selesai.