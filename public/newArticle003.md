---
title: RollupでTypeScriptを使ってみる
tags:
  - Rollup
  - TypeScript
  - 新人プログラマ応援
private: false
updated_at: ""
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

Rollup と TypeScript でライブラリを開発してみたいと考え、まずは Rollup と TypeScript でシンプルな開発環境を構築してみました。  
その時の雑メモを備忘録として残しておきます。  
npm や TypeScript の詳細については割愛しております。

index.html に「Hello, World」と表示できればゴールとします。

## 環境

- Node.js v21.2.0
- npm v10.2.3
- rollup v4.12.0
- @rollup/plugin-typescript v11.1.6
- tslib v2.6.2

## 最終的なディレクトリ構造

```ディレクトリ構造:ディレクトリ構造
rollup-ts-test/
  ├ node_modules/
  ├ src/
  │  └  app.ts
  ├ bundle.js
  ├ index.html
  ├ package-lock.json
  ├ package.json
  ├ rollup.config.js
  └ tsconfig.json
```

## 環境構築

### 作業用ディレクトリと index.html を作成

作業用のディレクトリ`rollup-ts-test`を作成して、作成したディレクトリ内に`index.html`を作成します。

```ターミナル: ターミナル
mkdir rollup-ts-test
cd rollup-ts-test
touch index.html
```

```ディレクトリ構造: ディレクトリ構造
rollup-ts-test/
  └ index.html
```

### 必要なパッケージをインストール

`npm`で必要なパッケージをインストールします。

```ターミナル: ターミナル
npm init -y
npm i -D rollup @rollup/plugin-typescript tslib
```

`rollup`は Rollup のパッケージ、`@rollup/plugin-typescript`と`tslib`は Rollup で TypeScript に対応するためにパッケージです。  
この時、`node_modules`フォルダと`package.json`、`package-lock.json`ファイルが作成されます。

```ディレクトリ構造: ディレクトリ構造
rollup-ts-test/
  ├ node_modules/
  ├ index.html
  ├ package-lock.json
  └ package.json
```

### src フォルダと ts ファイルを準備

`src`フォルダと`app.ts`を用意して、app.ts に「Hello, World」を出力する記述をします。

```ターミナル: ターミナル
mkdir src
cd src
touch app.ts
```

```app.ts
function greeter(person: string): string {
  return 'Hello, ' + person
}

const user: string = 'World';

document.body.textContent = greeter(user);
```

```ディレクトリ構造: ディレクトリ構造
rollup-ts-test/
  ├ node_modules/
  ├ src/
  │  └  app.ts
  ├ index.html
  ├ package-lock.json
  └ package.json
```

### rollup.config.js を作成

Rollup のコンフィグファイル`rollup.config.js`を作成します。

```ターミナル: ターミナル
cd ../
touch rollup.config.js
```

```rollup.config.js
import typescript from '@rollup/plugin-typescript';

export default {
  input: './src/app.ts',
  output: {
    file: 'bundle.js',
    format: 'iife',
  },
  plugins: [
    typescript()
  ]
}
```

```ディレクトリ構造:ディレクトリ構造
rollup-ts-test/
  ├ node_modules/
  ├ src/
  │  └  app.ts
  ├ index.html
  ├ package-lock.json
  ├ package.json
  └ rollup.config.js
```

### tsconfig.json を作成

ts ファイルを js ファイルにコンパイルするための設定ファイル`tsconfig.json`を作成します。

```ターミナル: ターミナル
touch tsconfig.json
```

```tsconfig.json
{
  "compilerOptions": {
    "target": "es5",
    "module": "esnext",
    "strict": true
  }
}
```

```ディレクトリ構造:ディレクトリ構造
rollup-ts-test/
  ├ node_modules/
  ├ src/
  │  └  app.ts
  ├ index.html
  ├ package-lock.json
  ├ package.json
  ├ rollup.config.js
  └ tsconfig.json
```

### コンパイルしてみる

`rollup -c`コマンドでコンパイルしてみると下記のエラーが出ます。

```ターミナル: ターミナル
[!] RollupError: Node tried to load your configuration file as CommonJS even though it is likely an ES module. To resolve this, change the extension of your configuration to ".mjs", set "type": "module" in your package.json file or pass the "--bundleConfigAsCjs" flag.
```

下記のどれかで対応することで解決します。

1. rollup.config.jsの拡張子を .mjs に変更する`rollup.config.mjs`
2. `package.json` ファイルに `"type": "module"` を追加する
3. `--bundleConfigAsCjs` フラグを Rollup コマンドに追加する

2の `package.json` ファイルに `"type": "module"` を追加する で対応します。

```package.json
{
  "name": "rollup-test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@rollup/plugin-typescript": "^11.1.6",
    "rollup": "^4.12.0",
    "tslib": "^2.6.2"
  }
}

```

これでもう一度`rollup -c`コマンドでkコンパイルしてみると`bundle.js`がルートディレクトリに作成されます。

```ディレクトリ構造:ディレクトリ構造
rollup-ts-test/
  ├ node_modules/
  ├ src/
  │  └  app.ts
  ├ bundle.js
  ├ index.html
  ├ package-lock.json
  ├ package.json
  ├ rollup.config.js
  └ tsconfig.json
```

### 表示確認

index.htmlでbundle.jsを読み込んで「Hello, World」と表示されれば完成です。

```index.html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>rollup-test</title>
</head>
<body>
  <script src="./bundle.js"></script>
</body>
</html>
```

## 最後に

実務で使うにはまだまだ設定が足りないですが、
webpackよりも簡単にTypeScriptの環境構築ができた気がします。  

## 参考元

https://zenn.dev/no4_dev/articles/74f80c4243919ea2a247-2

https://qiita.com/yutasuzuki/items/96f3dcb01b8e52c49aed

https://qiita.com/Futo_Horio/items/de182639b7e4261c9606

https://zenn.dev/s_takashi/scraps/ee10b6d8a6a937