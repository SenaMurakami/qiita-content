---
title: 【Docker初心者】1つのdocker-compose.ymlで複数サイトのローカル環境を作成
tags:
  - Docker
  - docker-compose
  - 新人プログラマ応援
private: false
updated_at: '2024-02-02T11:15:05+09:00'
id: 45f2dd57eabd5c3831c1
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

私は最近やっとDockerでローカル環境を構築するのが増えた、Docker初心者です。
こんな私でも簡単に、複数サイトのローカル環境をDockerで構築できる方法をまとめました。

この記事は [@shir01earn](https://qiita.com/shir01earn) さんの [Docker で PHP + PHP-FPM × Nginx × MySQL の開発環境を構築](https://qiita.com/shir01earn/items/f236c8280bb745dd6fb4)を参考にさせていただきましたので、細かい説明等を知りたい方は是非ご覧ください。

また、`site1`と`site2`は適宜変更してください。

## 環境

- Nginx: 1.25
- MySQL: 8.0
- PHP: 8.1

## ディレクトリ構成

コンテナを起動するとdb_dataは自動で作成されます。
それ以外を作成してください。

```ディレクトリ
project/
 ├─ db_data/ # 自動で作成される
 ├─ mysql/
 │  └─ my.cnf # mysqlの設定
 ├─ nginx/
 │  └─ default.conf # nginxの設定
 ├─ php/
 │  ├─ Dockerfile
 │  └─ php.ini # phpの設定
 ├─ site1/
 ├─ site2/
 └─ docker-compose.yml

```

## docker-compose.yml

```docker-compose.yml
version: '3.7'

services:
  php:
    build: ./php
    volumes:
      - ./site1:/var/www/site1
      - ./site2:/var/www/site2
    networks:
      - webnet

  nginx:
    image: nginx:1.25.0
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./site1:/var/www/site1
      - ./site2:/var/www/site2
    networks:
      - webnet
    depends_on:
      - php
      - mysql
    
  mysql:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: your_mysql_root_password
      MYSQL_DATABASE: your_database_name
      MYSQL_USER: your_mysql_user
      MYSQL_PASSWORD: your_mysql_password
    volumes:
      - ./db_data:/var/lib/mysql
    networks:
      - webnet

networks:
  webnet:

volumes:
  db_data:

```

## MySQL

```my.cnf
[mysqld]
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci

```

## Nginx

```default.conf
server {
    listen 80;
    server_name site1.local;

    root /var/www/site1;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}

server {
    listen 80;
    server_name site2.local;

    root /var/www/site2;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}

```

## PHP

```Dockerfile:Dockerfile
FROM php:8.1.18-fpm
COPY php.ini /usr/local/etc/php/

RUN apt-get update && apt-get install -y \
    git \
    curl \
    zip \
    unzip \
    && docker-php-ext-install pdo_mysql

WORKDIR /var/www
```

```php.ini
[Date]
date.timezone = "Asia/Tokyo"
[mbstring]
mbstring.internal_encoding = "UTF-8"
mbstring.language = "Japanese"
```

## hosts設定

hostsファイルに下記の行を追加してください。

```
127.0.0.1 site1.local site2.local
```

## 最後に

コンテナを起動します。

```ターミナル:ターミナル
docker-compose up -d
```

これで `http://site1.local/` と `http://site2.local/` それぞれ表示されてれば完了です。
コンテナの停止は下記のコマンドです。

```ターミナル:ターミナル
docker-compose down
```

Nginx、PHP、MySQLの設定は全部簡易的なものです。
公式のドキュメントを参照してご自分にあったカスタマイズをお願いします。

## 参考元

https://qiita.com/shir01earn/items/f236c8280bb745dd6fb4

https://jp-terrace.com/blog/2023/09/11/docker-nginx-php/
