# BambangShop Receiver App
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
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create SubscriberRequest model struct.`
    -   [ ] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Notification repository.`
    -   [ ] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Commit: `Implement receive_notification function in Notification service.`
    -   [ ] Commit: `Implement receive function in Notification controller.`
    -   [ ] Commit: `Implement list_messages function in Notification service.`
    -   [ ] Commit: `Implement list function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1
1. Kita membutuhkan sinkronisasi karena variabel global (seperti list notifikasi) akan diakses oleh banyak proses (thread) secara bersamaan dalam lingkungan aplikasi web yang berjalan asinkronus (multi-threading). Tanpa sinkronisasi, bisa terjadi data race (tabrakan data) yang membuat program crash.  
   Kita memilih RwLock<> (Read-Write Lock) dibandingkan Mutex<> karena sifat akses datanya. RwLock<> mengizinkan banyak proses untuk membaca data secara bersamaan, asalkan tidak ada yang sedang menulis. Namun, saat ada yang ingin menulis data baru, ia akan mengunci akses untuk semua proses lain. Sebaliknya, Mutex<> sangat kaku; ia hanya mengizinkan satu proses (baik untuk membaca maupun menulis) pada satu waktu. Karena aplikasi kita akan lebih sering "membaca" daftar notifikasi untuk ditampilkan daripada "menulis" notifikasi baru, RwLock<> jauh lebih efisien secara performa dibandingkan Mutex<>.
2. Rust memiliki aturan yang sangat ketat terkait keamanan memori (memory safety) dan keamanan thread (thread safety) sejak fase kompilasi. Di Rust, variabel statik global yang bisa diubah (mutable global state) dianggap sangat tidak aman karena berpotensi besar memicu data race jika diakses oleh beberapa thread secara bersamaan tanpa pelindung. Berbeda dengan Java yang membiarkan developer mengurus sinkronisasinya sendiri (atau membiarkan error terjadi saat program berjalan), compiler Rust langsung melarang mutasi variabel statik secara langsung. Oleh karena itu, kita harus menggunakan lazy_static! untuk membungkus data tersebut di dalam tipe pelindung yang aman untuk multi-threading (seperti RwLock, Mutex, atau DashMap) sehingga sinkronisasinya terjamin saat program dijalankan.
#### Reflection Subscriber-2
1. Ya, saya mengeksplorasi src/lib.rs. Dari file tersebut, saya belajar bahwa lib.rs bertindak sebagai pusat konfigurasi utama untuk aplikasi Rocket ini. Di dalamnya terdapat inisialisasi AppConfig untuk membaca environment variables (seperti ROCKET_PORT dan URL aplikasi), serta inisialisasi REQWEST_CLIENT secara lazy menggunakan lazy_static yang nantinya dipakai oleh Service untuk mengirim HTTP request. Selain itu, fungsi-fungsi error handling standar seperti compose_error_response juga dipusatkan di sini agar kode lebih bersih.
2. Observer pattern membuat penambahan subscriber baru (Receiver) menjadi sangat mudah (loosely coupled). Publisher tidak peduli bagaimana internal Receiver bekerja; ia hanya butuh URL mereka untuk menembakkan POST request. Jadi, kita bisa menyalakan ratusan Receiver di port berbeda tanpa mengubah satu baris kode pun di Publisher.
   Namun, untuk menjalankan lebih dari satu instance Main app (Publisher), pendekatannya tidak semudah itu. Karena saat ini Publisher menyimpan daftar Subscriber dan Product di in-memory data structure (DashMap lokal), setiap instance Publisher akan memiliki datanya sendiri yang tidak tersinkronisasi. Agar bisa menjalankan banyak Publisher, kita harus mengganti in-memory storage tersebut dengan database sungguhan (seperti PostgreSQL atau Redis) yang bisa diakses secara terpusat oleh semua instance Publisher.
3. Ya, menggunakan Postman collection dan bereksperimen dengan Environment Variables di dalamnya sangat membantu mempercepat pengujian. Kita bisa mengatur variabel seperti {{base_url}} atau {{port}} sehingga jika kita ingin berpindah menguji Receiver di port 8001, 8002, atau 8003, kita tidak perlu mengetik ulang URL secara manual. Fitur ini jelas akan sangat bermanfaat di Group Project nanti untuk memastikan semua API yang dikerjakan oleh backend berjalan sesuai spesifikasi sebelum diintegrasikan dengan frontend.
