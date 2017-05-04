# **Pengenalan Docker**

Apa sih docker itu? Docker adalah salah satu platform yang dibangun berdasarkan teknologi container. Docker ini bersifat open source dan dibuat untuk mempermudah developer atau sysadmin dalam membangun, mengemas, dan menjalankan aplikasi dalam sebuah container. Docker berperan sebagai virtualisasi sebuah server, web server, ataupun database server. Jadi developer dapat membuat aplikasi sesuai spesifikasi yang telah di tetapkan sebelumnya. Hal ini juga memudahkan pengerjaan suatu projek yang di kerjakan oleh beberapa orang(team) karena environment sudah di tetapkan dari awal.

Docker dilengkapi dengan fitur sandbox yang menjamin pengerjaan pengembang dan sysadmin tidak terganggu. Sandbox adalah mekanisme pemisahan aplikasi atau program tanpa mengganggu host. Bagi developer, sandbox menjamin aplikasinya dapat berjalan tanpa ada gangguan atas perubahan lingkungan host.Berikut ini cara untuk instalasi docker. Disini saya menggunakan OS ubuntu 16.04.

Pertama, update pakage database ubuntu dengan mengetik :
```
$ sudo apt-get update
```
Kedua, tambahkan key GPG dari official docker :
```
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80  --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```
Ketiga, tambahkan repositori docker :
```
$ sudo apt-add-repository 'deb https://apt.dockerproject.org/repo ubuntu-xenial main'
```
Keempat, update pakage database dengan repositori yang sudah di tambahkan :
```
$ sudo apt-get update
```
Kelima, cek apakah docker yang akan diinstall adalah default repo ubuntu 16.04 :
```
$ apt-cache policy docker-engine
```

    Lalu akan muncul tulisan berikut di terminal
    
    docker-engine:
        Installed: 17.04.0~ce-0~ubuntu-xenial
        Candidate: 17.04.0~ce-0~ubuntu-xenial
    
Keenam, install docker engine

```
$ sudo apt-get install -y docker-engine
```

Setelah selesai menginstal kita bisa cek apakah docker sudah terinstall, yaitu dengan mengetik :
```
$ sudo systemctl status docker.
```
Berikutnya saya akan membuat pengaturan untuk mendevelopment aplikasi menggunakan docker. Disini yang saya gunakan adalah laravel dan server nginx.

Pertama-tama kita harus menginstal laravel terlebih dahulu, bisa menggunakan composer ataupun download manual. Disini saya menginstalnya langsung dari composer. Perlu diingat ketika kalian mengunduh laravel dengan cara manual, kalian harus melakukan composer update.

Untuk download manual kalian bisa mengetik pada cmd :
```
$ curl -L https://github.com/laravel/laravel/archive/v5.3.16.tar.gz | tar xz
```
Untuk download laravel menggunakan composer :
```
$ composer create-project --prefer-dist laravel/laravel blog
```

Kemudia kita akan menggunakan composer/composer image dari docker
```
$ docker run --rm -v $(pwd):/app composer/composer install
```

Kita buat file `docker-compose.yml` untuk mengkonfigurasi docker. Lalu edit file tersebut, bisa menggunakan `$ sudo nano docker-compose.yml` atau menggunakan editor lainnya. Lalu ketik :
```
version: '2'
services:

  #Application
  app:
    build:
      context: ./
      dockerfile: app.dockerfile
    working_dir: /var/www
    volumes:
      - ./:/var/www
    environment:
      - "DB_PORT=3306"
      - "DB_HOST=database"

  #Web Server
  web:
    build:
      context: ./
      dockerfile: web.dockerfile
    working_dir: /var/www
    volumes_from:
      - app
    ports:
      - 1111:80

  #Database
  database:
    image: mysql:5.6
    volumes:
      - dbdata:/var/lib/mysql
    environment:
      - "MYSQL_DATABASE=homestead"
      - "MYSQL_USER=homestead"
      - "MYSQL_PASSWORD=secret"
      - "MYSQL_ROOT_PASSWORD=secret"
    ports:
        - "33066:3306"

  volumes:
    dbdata:
```

Pada web: ports: di atas biasanya menggunakan  8080:80, saya sengaja menggunakan 1111:80 karena port 8080 sudah digunakan.

Kita buat file dengan nama `app.dockerfile` dan isikan :
```
    FROM php:7.0.4-fpm

    RUN apt-get update && apt-get install -y libmcrypt-dev \
        mysql-client libmagickwand-dev --no-install-recommends \
        && pecl install imagick \
        && docker-php-ext-enable imagick \
        && docker-php-ext-install mcrypt pdo_mysql
```
Lalu buat file dengan nama `web.dockerfile` untuk konfigurasi nginx dan isikan :
```
    FROM nginx:1.10

    ADD vhost.conf /etc/nginx/conf.d/default.conf
```
Untuk `vhost.conf` diatas kita harus membuat filenya terlebih dahulu lalu isikan :
```
    server {
        listen 80;
        index index.php index.html;
        root /var/www/public;

        location / {
            try_files $uri /index.php?$args;
        }

        location ~ \.php$ {
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass app:9000;
            fastcgi_index index.php;
            include fastcgi_params;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_path_info;
        }
    }
```

Setelah semua konfigurasi selesai, jalankan `docker-composer build` dan untuk menjalankannya kita bisa mengetik `docker-composer up`. Setelah selesai loading kita tinggal membuka web kita tadi di browser dengan url `localhost:1111`.

![alt text](https://doc-0c-7k-docs.googleusercontent.com/docs/securesc/c80u2j8rspp7k99kb306llscrl71l3jn/ipe9rppv7i3je13dnq1gsi7a3krg98ph/1493856000000/11285658977785115420/11285658977785115420/0B85NQzrRozwwdXBYSDRxR3Fabjg?e=view&nonce=a4rtcpeh1q3pi&user=11285658977785115420&hash=fk15o8jjo1sk9g08bo7g0fqa7ua8fd4q "laravel")

Sumber referensi :
1. https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04
2. https://medium.com/@shakyShane/laravel-docker-part-1-setup-for-development-e3daaefaf3c
