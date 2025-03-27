# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2025 - Faculty of Computer Science, Universitas Indonesia

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

You can download the Postman Collection JSON from the official documentation.

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
-   [✅] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [✅] Commit: `Create Subscriber model struct.`
    -   [✅] Commit: `Create Notification model struct.`
    -   [✅] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [✅] Commit: `Implement add function in Subscriber repository.`
    -   [✅] Commit: `Implement list_all function in Subscriber repository.`
    -   [✅] Commit: `Implement delete function in Subscriber repository.`
    -   [✅] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [✅] Commit: `Create Notification service struct skeleton.`
    -   [✅] Commit: `Implement subscribe function in Notification service.`
    -   [✅] Commit: `Implement subscribe function in Notification controller.`
    -   [✅] Commit: `Implement unsubscribe function in Notification service.`
    -   [✅] Commit: `Implement unsubscribe function in Notification controller.`
    -   [✅] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [✅] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [✅] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [✅] Commit: `Implement publish function in Program service and Program controller.`
    -   [✅] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [✅] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## My Reflections
#### Reflection Publisher-1
1. In the Observer pattern diagram explained by the Head First Design Pattern book, Subscriber is defined as an interface. Explain based on your understanding of Observer design patterns, do we still need an interface (or `trait` in Rust) in this BambangShop case, or a single Model `struct` is enough?

    Untuk kasus kita sekarang, penggunaan single `struct` (tanpa `trait`) sudah cukup. Alasannya adalah:
    - Saat ini hanya ada satu tipe Observer, yakni `Subscriber` sehingga belum ada urgensi untuk abstraksi/polimorfisme melalui `trait`
    - Semisal di masa yang akan datang ada jenis Observer baru , barulah `trait` diperlukan untuk menerapkan Open-Closed Principle
    - Penggunaan `trait` baru diperlukan ketika ada variasi perilaku Observer yang harus di-handle secara generik 
2. `id` in `Program` and `url` in `Subscriber` is intended to be unique. Explain based on your understanding, is using `Vec` (list) sufficient or using `DashMap` (map/dictionary) like we currently use is necessary for this case?

    Penggunaan `Dashmap` menurut saya lebih tepat dibanding `Vec` karena:
    - `Dashmap` memungkinkan pencarian langsung berdasarkan `id` atau `url` (complexity of `O(1)`), sedangkan `Vec` memiliki complexity of `O(n)` untuk mencari pasangan data
    - Dengan `Dashmap`, data terkait dapat disimpan dalam satu struktur, sementara `Vec` memaksa penggunaan dua struktur terpisah
    - `DashMap` mendukung akses concurrent secara built-in yang cocok digunakan untuk sistem multi-threaded seperti kasus BambangShop
3. When programming using Rust, we are enforced by rigorous compiler constraints to make a thread-safe program. In the case of the List of Subscribers (`SUBSCRIBERS`) static variable, we used the `DashMap` external library for thread safe `HashMap`. Explain based on your understanding of design patterns, do we still need `DashMap` or we can implement Singleton pattern instead?

    Menurut saya, karena:
    - `DashMap` menyediakan akses concurrent yang aman tanpa perlu implementasi manual locking (seperti `RwLock` atau `Mutex`) yang berisiko deadlock
    - Meskipun `Singleton` sudah memastikan satu instance, tetapi dia tidak otomatis menjamin keamanan thread. Untuk `Singleton` dengan `HashMap` biasa, operasi read/write harus dilindungi lock yang berpengaruh pada performa dan kompleksitas yang ada
    - `DashMap` sudah mengintegrasikan thread safety dan efisiensi sehingga lebih cocok untuk kasus ini ketimbang yang lain


#### Reflection Publisher-2
1. In the Model-View Controller (MVC) compound pattern, there is no “Service” and “Repository”. Model in MVC covers both data storage and business logic. Explain based on your understanding of design principles, why we need to separate “Service” and “Repository” from a Model?

    Menurut saya, pemisahan ini didasarkan pada Single Responsibility Principle (SRP), di mana:
    - Repository bertanggung jawab pada pengelolaan akses data (misal CRUD ke database), sementara Service lebih fokus pada logika bisnis (misal validasi)
    - Dengan memisahkan kedua layer tersebut, kode menjadi lebih modular, testable, dan mengurangi coupling. Misalnya perubahan pada struktur database tidak akan memengaruhi logika bisnis di Service atau sebaliknya
    - Pemisahan ini juga meningkatkan scalability karena memungkinkan pengembangan/penggantian teknologi tanpa mengganggu alur bisnis utama

2. What happens if we only use the Model? Explain your imagination on how the interactions between each model (`Program`, `Subscriber`, `Notification`) affect the code complexity for each model?

    - Kompleksitas kode akan meningkat, model akan menjadi rumit sehingga sulit dipahami, dimodifikasi, dan bahkan di-debug karena menggabungkan banyak fungsi sekaligus
    - Coupling tinggi, perubahan pada suatu bagian dapat memengaruhi bagian lain yang seharusnya tidak ada hubungannya
    - Unit testing menjadi tidak efektif karena tidak ada isolasi antara logika bisnis dan akses data
3. Have you explored more about Postman? Tell us how this tool helps you to test your current work. You might want to also list which features in Postman you are interested in or feel like it is helpful to help your Group Project or any of your future software engineering projects.

    Ya, saya sempat eksplor-eksplor Postman sedikit. Menurut saya, fitur yang menarik adalah:
    - HTTP Request Testing, yang bisa mengirim berbagai jenis request dengan parameter, header, dan body yang fleksibel
    - Collections, yang bisa mengelompokkan endpoint dalam folder untuk pengujian yang lebih terstruktur, yang akan dibutuhkan ketika pengerjaan dilakukan dalam skala yang lebih besar (misal dokumentasi API)
    - Response Validation, yang memeriksa response API untuk memastikan keluaran sesuai dengan ekspektasi
    - Environment Variables, menyimpan variabel global yang dapat digunakan di seluruh request, mempermudah pengujian di berbagai environment (dev, stage, prod, dsb.)


#### Reflection Publisher-3
1. Observer Pattern has two variations: Push model (publisher pushes data to subscribers) and Pull model (subscribers pull data from publisher). In this tutorial case, which variation of Observer Pattern that we use?
    
    Di tutorial ini, variasi yang digunakan adalah Push model karena Publisher, yakni `NotificationService` secara aktif mengirimkan notifikasi ke semua Subscriber secara langsung saat terjadi perubahan data (CRUD)

2. What are the advantages and disadvantages of using the other variation of Observer Pattern for this tutorial case? (example: if you answer Q1 with Push, then imagine if we used Pull)
    
    Jika menggunakan Pull model, kelebihannya adalah:
    - Subscriber bisa memilih data spesifik yang dibutuhkan sehingga akan mengurangi transfer data yang irrelevant
    - Lebih efisien jika Subscriber memiliki kebutuhan data yang bervariasi
    - Subscriber memiliki kontrol penuh atas waktu dan frekuensi pengambilan data
    
    Sementara itu, kekurangannya adalah sebagai berikut.
    - Subscriber harus memahami struktur data Publisher untuk melakukan pull
    - Jika Subscriber tidak aktif melakukan pull, notifikasi bisa terlewat
    - Subscriber perlu secara manual memantau perubahan yang menyebabkan overhead tambahan

3. Explain what will happen to the program if we decide to not use multi-threading in the notification process.

    - Notifikasi akan diproses secara sekuensial menyebabkan antrean panjang
    - Bottleneck pada server, terutama ketika Subscriber banyak atau proses pengiriman lama
    - Proses CRUD product akan terhambat karena sistem harus menunggu semua notifikasi selesai dikirim sebelum melanjutkan operasi lain
    - Skalabilitas menurun karena tiap penambahan Subscriber akan memperparah delay yang terjadi


