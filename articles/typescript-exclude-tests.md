---
title: "TypeScriptでビルドをするときにテストファイルを除く方法"
emoji: "🧹"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [typescript, test, jest]
published: true
---

TypeScriptでビルドをするときにテストファイルをビルド結果から取り除く方法です。


# プロジェクトの構成
以下のような構成を参考にして説明します。
(testsフォルダはsrcの外側にありますが、srcの中にある場合も設定方法は同じです)

```bash
.
├── src
│   ├── index.ts
│   ├── lib
│   └── (略)
├── tests
└── tsconfig.json
```

# tsconfigの最初の設定内容と実行結果
tsconfigの設定は以下のようにしておりました。
```json:tsconfig.json
{
  "compilerOptions": {
    ・・・
    "rootDir": "." ,
    "outDir": "./dist",
    ・・・
  }
}
```

package.jsonには以下のようなscriptsが設定してあります。
```json:package.json
{
  "scripts": {
    "build": "tsc",
  }
}
```

これでビルドをすると
```bash
npm run build
```

このようにtestsフォルダも一緒にビルドされてしまいます。
```bash
.
├── dist
│   ├── src
│   │   ├── index.js
│   │   ├── lib
│   │   └── (略)
│   └── tests # testsフォルダもビルド結果に含まれてしまう
├── src
│   ├── index.ts
│   ├── lib
│   └── (略)
├── tests
└── tsconfig.json
```




# 解決方法(ボツ)
tsconfigを以下のように設定すれば、ビルド結果からtestsは取り除かれます。
ただ、testファイルを.tsで書いていた場合に今度はテストが実行されなくなってしまいますのでこの方法はだめでした。
```json:tsconfig.json
{
  "compilerOptions": {
    ・・・
    "rootDir": "." ,
    "outDir": "./dist",
    ・・・
  },
  "include": ["./src/**/*.ts"]
}
```

# 解決方法(採用)
ビルド時と開発時でtsconfigの設定をわけることでこの問題を解決できます。
また、ビルド時と開発時で設定内容はほぼ同じになると思いますので、共通部分をtsconfig.jsonで定義しておきビルド時の差分のみを上書きするようにします。tsconfigでは[extends](https://www.typescriptlang.org/tsconfig#extends)を使って設定内容を上書きできます。

**tsconfig.json**
```json:tsconfig.json
{
  "compilerOptions": {
    ・・・
    "rootDir": "." ,
    "outDir": "./dist",
    ・・・
  }
}
```

**tsconfig.build.json**
```json:tsconfig.build.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "rootDir": "./src" 
  },
  "include": ["./src/**/*.ts"]
}
```

また、ビルド時にはtsconfig.build.jsonを使用するように設定します。
```json:package.json
{
  "scripts": {
    "build": "tsc --project tsconfig.build.json",
  }
}
```

これでビルドをすると
```bash
.
├── dist
│   └── src
│       ├── index.js
│       ├── lib
│       └── (略)   
├── src
│   ├── index.ts
│   ├── lib
│   └── (略)
├── tests
├── tsconfig.json
└── tsconfig.build.json
```

このような結果になり、ビルド結果からtestsが除かれるようになりました。
