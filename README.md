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
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Subscriber model struct.`
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [x] Commit: `Implement add function in Subscriber repository.`
    -   [x] Commit: `Implement list_all function in Subscriber repository.`
    -   [x] Commit: `Implement delete function in Subscriber repository.`
    -   [x] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [x] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [x] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [x] Commit: `Implement publish function in Program service and Program controller.`
    -   [x] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [x] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

1. Berdasarkan pemahaman saya mengenai Observer design pattern, penggunaan sebuah **Model struct** tunggal dalam kasus BambangShop ini sudah cukup memadai. Hal ini dikarenakan saat ini seluruh Subscriber memiliki struktur data dan perilaku yang seragam, yaitu hanya memerlukan `url` dan `name` untuk menerima notifikasi melalui HTTP request. Penggunaan *interface* atau *trait* di Rust baru akan menjadi krusial jika ke depannya terdapat berbagai jenis Subscriber dengan mekanisme pengiriman yang berbeda-beda (misalnya ada yang via Email, SMS, atau protokol lain), sehingga kita perlu melakukan abstraksi pada fungsi pemberitahuannya.

2. Penggunaan **DashMap** jauh lebih tepat dan diperlukan dibandingkan hanya menggunakan `Vec` (list) untuk menjamin keunikan `id` atau `url`. Dengan `DashMap`, kita bisa melakukan pencarian, penambahan, dan penghapusan data dengan kompleksitas waktu rata-rata $O(1)$. Jika kita hanya menggunakan `Vec`, kita harus melakukan iterasi secara sekuensial (O(n)) setiap kali ingin memastikan keunikan data atau menghapus subscriber tertentu, yang mana akan menjadi sangat tidak efisien seiring bertambahnya jumlah data di dalam sistem.

3. Kita tetap memerlukan **DashMap** meskipun sudah menerapkan Singleton pattern melalui `lazy_static`. Dalam ekosistem Rust yang sangat ketat terhadap keamanan memori, variabel statis yang bersifat *mutable* harus dikelola dengan mekanisme *concurrency control* agar tidak terjadi *data race*. Singleton hanya menjamin bahwa hanya ada satu *instance* database yang digunakan di seluruh aplikasi, namun tidak menjamin keamanan saat beberapa *thread* mencoba mengakses atau mengubah data secara bersamaan. `DashMap` menyediakan *thread-safe* HashMap secara *built-in*, sehingga kita tidak perlu lagi membungkus struktur data secara manual dengan `Arc<Mutex<HashMap>>` demi menjaga keamanan *multi-threading*.

#### Reflection Publisher-2

1. Pemisahan antara **Service** dan **Repository** dari Model sangat krusial untuk menjaga prinsip *Single Responsibility Principle* (SRP). Dalam MVC tradisional, Model seringkali menjadi "gemuk" (*Fat Model*) karena harus menangani penyimpanan data sekaligus logika bisnis. Dengan memisahkannya, Repository fokus sepenuhnya pada abstraksi akses data (seperti operasi `DashMap` tadi), sedangkan Service fokus pada koordinasi logika bisnis (seperti validasi atau transformasi data). Hal ini membuat kode kita lebih modular, lebih mudah diuji (*testable*), dan jika suatu saat kita ingin mengganti mekanisme database, kita hanya perlu mengubah lapisan Repository tanpa menyentuh logika bisnis di Service.

2. Jika kita hanya menggunakan Model untuk menangani semuanya, kompleksitas kode akan meningkat secara drastis (*Spaghetti Code*). Model `Program` akan terbebani dengan logika notifikasi, Model `Subscriber` harus mengerti cara menyimpan dirinya sendiri, dan Model `Notification` mungkin harus tahu cara mengirim dirinya sendiri melalui HTTP. Interaksi antar model akan menjadi sangat terikat satu sama lain (*high coupling*). Bayangkan jika setiap kali ada perubahan produk, Model `Product` harus memanggil daftar `Subscriber` dan melakukan loop untuk mengirim `Notification`. ini akan membuat satu class memiliki ketergantungan yang terlalu banyak dan sangat sulit untuk di-maintain atau di-*refactor*.

3. **Postman** sangat membantu dalam proses pengembangan fitur ini karena memungkinkan saya untuk menguji endpoint API secara mandiri tanpa harus menunggu front-end atau aplikasi penerima (Receiver) selesai dibuat. Saya bisa mensimulasikan berbagai skenario, seperti melakukan `POST` subscription dengan data valid maupun invalid, serta memverifikasi status code (seperti `201 Created` atau `404 Not Found`) secara langsung. Fitur yang sangat bermanfaat untuk proyek ke depannya adalah *Collections* untuk mengelompokkan request, *Environment Variables* untuk berpindah antar environment (lokal vs produksi), serta *Automated Test Scripts* yang bisa memastikan respons API selalu sesuai kontrak yang diharapkan secara otomatis.

#### Reflection Publisher-3

1. Dalam tutorial BambangShop ini, variasi Observer Pattern yang kita gunakan adalah **Push model**. Hal ini terlihat jelas pada implementasi fungsi `notify()` di `NotificationService`, di mana Main App (Publisher) secara langsung mengirimkan data payload `Notification` secara lengkap kepada setiap Subscriber melalui HTTP POST request segera setelah sebuah event terjadi. Subscriber tidak perlu bertanya atau meminta data secara manual, mereka tinggal menerima kiriman data dari Publisher.

2. Jika kita menggunakan **Pull model**, keuntungannya adalah Subscriber memiliki kendali penuh atas kapan mereka ingin mengambil data, sehingga tidak akan terjadi *overload* jika Publisher mengirim terlalu banyak notifikasi dalam waktu singkat. Namun, kekurangannya sangat signifikan untuk kasus BambangShop: Subscriber tidak akan mendapatkan informasi secara *real-time* karena mereka harus melakukan *polling* secara berkala. Hal ini juga akan memboroskan sumber daya jaringan dan CPU Publisher jika banyak Subscriber terus-menerus melakukan request hanya untuk mengecek apakah ada produk baru, padahal statusnya mungkin belum berubah.

3. Jika kita memutuskan untuk tidak menggunakan *multi-threading* (alias dijalankan secara sekuensial) dalam proses notifikasi, maka aplikasi akan mengalami *blocking* yang parah. Setiap kali ada event seperti `create` atau `publish`, Main App harus menunggu satu HTTP request ke Subscriber selesai (termasuk menunggu timeout jika URL Subscriber mati) sebelum bisa lanjut mengirim ke Subscriber berikutnya. Hal ini akan membuat performa aplikasi terasa sangat lambat bagi pengguna, karena aksi sederhana seperti menambah produk bisa memakan waktu berdetik-detik hanya untuk menyelesaikan antrean pengiriman notifikasi secara berurutan.