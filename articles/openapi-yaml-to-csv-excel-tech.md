---
title: "OpenAPIのYAMLファイルをCSVに変換するCLI(Node.js)をTypeScriptで作成した"
emoji: "🧑‍💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [nodejs, openapi, cli, typescript, oss]
published: true
---

OpenAPIのYAMLファイルをCSVに変換するCLIを作成しました。
Node.jsがインストールされていれば実行できます。TypeScriptで作成しました。

この記事では、このCLIを作成するにあたっての開発環境の構築からnpmライブラリとして公開するまでの手順をご紹介します。
プロダクトの概要やどういったプロセスで実施したかも記事にしておりますのであわせてご覧ください。
https://zenn.dev/horitaka/articles/openapi-yaml-to-csv-excel-product
https://zenn.dev/horitaka/articles/openapi-yaml-to-csv-excel-process


# 開発環境の構築

## Node.js + TypeScriptの環境構築

### 1. Node.jsプロジェクトを作成する
npm initを実行します。
```
npm init

package name: (oapi-yaml-to-csv-excel) 
version: (1.0.0) 
description: CLI tool to convert OpenAPI yaml file to CSV/Excel file.
entry point: (index.js) dist/index.js
test command: 
git repository: 
keywords: openapi converter yaml csv excel
author: horitaka
license: (ISC) MIT

About to write to /Users/takahiro/Downloads/oapi-yaml-to-csv-excel/package.json:
...
```
自分だけで使うものであれば`npm init -y`としてデフォルトの値を入れることも多いのですが、今回はOSSとしてnpmリポジトリに公開する予定のためすべての質問に答えていきます。

次に、TypeScriptをインストールします。
```
npm install typescript --save-dev
```
そしてtsconfigの設定を行います。
```
npx tsc --init
```

Node.js + TypeScriptの構築手順はこちらを参考にしました。
https://typescript-jp.gitbook.io/deep-dive/nodejs

### 2. importパスのaliasを設定する

このままでも開発できますが、importのパスをシンプルにするためにaliasを設定します。

例えば、相対パスでモジュールをimportするときに、以下のように階層が深くなってしまうことがあります。
```ts
import { someModule } from "../../lib/someModule"
```
これを以下のようにsrcからのパスを指定してimportできるようにします。
```ts
import { someModule } from "@/lib/someModule"
```

tsconfigのpathsに設定をすれば可能です。
```json:tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```
https://www.typescriptlang.org/docs/handbook/module-resolution.html#path-mapping


ただ、このままですとビルドしたあとにパスのaliasがうまく解決できずにビルド後の実行でエラーになってしまいます。
これを解決するためにtsc-aliasを導入します。
https://github.com/justkey007/tsc-alias

```
npm install --save-dev tsc-alias
```

package.jsonのbuild scriptも更新します。
```diff:package.json
{
  "scripts": {
-   "build": "tsc",
+   "build": "tsc && tsc-alias",
  }
}
```

## commitlintとCommitizenの導入

コミットメッセージの入力を補助するためのツールとしてcommitlintとCommitizeを導入します。
詳しい手順は下記の記事をご覧ください。
https://zenn.dev/horitaka/articles/commit-message-rules


## Jestの導入
Unit testのフレームワークとしてJestを導入します。

```
npm install --save-dev jest
```

TypeScriptで実行できるように関連ライブラリを追加します。
```
npm install --save-dev ts-jest
npm install --save-dev @types/jest
```

Jestの設定ファイルを追加します。
最後の`moduleNameMapper`は先ほど`tsconfig.json`設定したimportパスのaliasをJestでも解釈できるようにするためのものです。
```js:jest.config.js
/** @type {import('ts-jest').JestConfigWithTsJest} */
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
}
```

package.jsonのtest scriptも更新しておきます。
```diff:package.json
{
  "scripts": {
+   "test": "jest",
+   "test:watch": "jest --watch",
  }
}
```

# Node.jsでのCLIコマンド実行

## yargsの導入
Node.jsのCLIコマンドの引数をパースするためのライブラリとしてyargsを導入します。これがなくてもCLIは作れるのですが、yargsを使ったほうが圧倒的に簡単にNode.jsでのCLIを作ることができるのでお勧めです。
https://github.com/yargs/yargs

yargsの導入にあたってはこちらの記事も参考にしました。
https://qiita.com/toshi-toma/items/ea76b8894e7771d47e10
https://zenn.dev/ryo_kawamata/articles/introduce-yargs

## package.jsonの設定
package.jsonのbinに設定を追加します。
binフィールドはコマンド名と実行するファイルの紐付けを行ってくれます。

```diff:package.json
{
+  "bin": {
+    "@horitaka/oapi-convert": "dist/index.js",
+    "oapi-convert": "dist/index.js"
+  }
}
```

https://docs.npmjs.com/cli/v9/configuring-npm/package-json#bin
> supply a bin field in your package.json which is a map of command name to local file name.



# npmリポジトリへの公開

ここまでの設定をして一通り開発をしたら、あとはnpmリポジトリに公開するのみです。

ビルドします。
```
npm run build
```

package.jsonのfilesフィールドに追記します。
```diff:package.json
{
+  "files": [
+    "dist"
+  ],
}
```

filesフィールドを指定することでnpmライブラリをインストールする時にパッケージに含まれる対象を設定することができます。今回はビルドした結果がdistに出力されるためdistのみがパッケージに含まれていれば良いのでfilesにはdistを指定します。

https://docs.npmjs.com/cli/v9/configuring-npm/package-json#files
> The optional files field is an array of file patterns that describes the entries to be included when your package is installed as a dependency. 


npmアカウントアカウントを作成したあと、ログインをして、
```
npm login
・・・
```

npmに公開します。
```
npm run publish
```

これで無事に公開できました。

# CI/CD環境の整備
このままでも利用できるのですが、今後、パッケージへ機能追加やバグ修正などもしていく予定ですので、CIとCDの設定もしておきます。
CI/CDにはGitHub Actionsを使用しました。

## GitHub ActionsでのLinterとUnit testの実行
mainブランチにマージされたときにLinterとUnit testを実行するようにします。

```yaml:.github/workflows/ci.yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js 18.x
        uses: actions/setup-node@v3
        with:
          node-version: 18.x
          cache: npm
      - name: Install
        run: npm ci
      - name: Lint
        run: npm run lint

  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [14.x, 16.x, 18.x]

    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: npm
      - name: Install
        run: npm ci
      - name: Test
        run: npm test
```

設定は以下のサンプルをほぼそのまま使用しました。
https://docs.github.com/ja/actions/automating-builds-and-tests/building-and-testing-nodejs


## GitHub Actionsでの自動デプロイの実行
自動でnpmにpublishできるようにも設定します。
mainブランチにマージされたあとで`npm run publish`を実行しても良いのですが、[semantic-release](https://github.com/semantic-release/semantic-release)という便利なライブラリがあるのでこれを使用します。
semantic-releaseを利用することで、コミットメッセージを読み取って自動でバージョン管理やnpmへのpublishを実行してくれます。

semantic-releaseを追加します。
```
npm install --save-dev semantic-release
```

package.jsonに設定を追加します。
```package.json
  "release": {
    "plugins": [
      "@semantic-release/commit-analyzer",
      "@semantic-release/release-notes-generator",
      "@semantic-release/github",
      "@semantic-release/npm"
    ],
    "branches": [
      "main"
    ]
  }
```

GitHub Actionsにも追加します。

```yaml:.github/workflows/release.yaml
name: Release

on:
  workflow_dispatch:

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18.x
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: npx semantic-release
```


設定ファイルはsemantic-releaseのドキュメントに書いてある以下の内容をほぼそのまま利用しました。

https://github.com/semantic-release/semantic-release/blob/master/docs/recipes/ci-configurations/github-actions.md#using-semantic-release-with-github-actions


----

これで無事に完了です。
もしよければぜひ使っていただけるとうれしいです。
https://github.com/horitaka/openapi-yaml-to-csv-excel