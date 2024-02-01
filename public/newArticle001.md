---
title: 【docker初心者】1つのdocker-compose.ymlで複数サイトのローカル環境を作成
tags:
  - Docker
  - docker-compose
  - 新人プログラマ応援
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

私は最近やっとDockerでローカル環境を構築するのが増えた、Docker初心者です。
こんな私でも簡単に、複数サイトのローカル環境をDockerで構築できる方法をまとめました。

## 環境

- Nginx: 1.25
- MySQL: 8.0
- PHP: 8.1

## ディレクトリ構成

```
project/
 ├─ db_datta/
 ├─ nginx/
 │  └─ default.conf
 ├─ php/
 │  ├─ Dockerfile
 │  └─ php.ini
 │
 ├─ site1/
 ├─ site2/
 └─ docker-compose.yml

```

## 参考元

https://qiita.com/shir01earn/items/f236c8280bb745dd6fb4

https://jp-terrace.com/blog/2023/09/11/docker-nginx-php/
