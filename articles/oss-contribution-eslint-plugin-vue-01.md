---
title: "OSSにコントリビュートした(eslint-plugin-vue) - component-name-in-template-casing"
emoji: "🔨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vue, JavaScript, ESLint, OSS]
published: true
---

# 概要

Vue.js用のESLint pluginであるeslint-plugin-vueにコントリビュートしました。
issueの修正からPull requestを出すまでの流れを解説します。
OSSコントリビュートに興味がある方やeslintの仕組みに興味がある方などの参考になれば幸いです。


## 対象のライブラリ
eslint-plugin-vue
https://github.com/vuejs/eslint-plugin-vue

> Official ESLint plugin for Vue.js.
> This plugin allows us to check the <template> and <script> of .vue files with ESLint, as well as Vue code in .js files.

## issueの内容
https://github.com/vuejs/eslint-plugin-vue/issues/2048

> [vue/component-name-in-template-casing] False positive for dynamic component in template


# コントリビュートまでの実施内容

## 問題内容を把握する

まず、問題の内容を確認します。
issueを見ると、

> I've found a false positive scenario when TypeScript is enabled and type Component is imported from the vue library.

と書いてあります。

False positiveですので、本来はエラーではないものがエラーとして検知されてしまっているということのようです。


もう少し詳しくissueを見てみますと以下のように記載されております。

**What did you do?**
```vue
<template>
  <component />
</template>

<script lang="ts" setup>
  import { type Component } from 'vue';
</script>
```

**What actually happened?**
```bash
App.vue
  2:3  error  Component name "component" is not PascalCase  vue/component-name-in-template-casing

✖ 1 problem (1 error, 0 warnings)
  1 error and 0 warnings potentially fixable with the `--fix` option.
```

`<component />`はエラーとして検出するべきではないということのようです。


## eslintの仕組みを理解する

eslintはアプリケーション開発には導入はしているのですが自分でルールを作ったことはないため、まずはeslintの仕組みを理解する必要がありました。
まずは[Contribution Guide](https://github.com/vuejs/eslint-plugin-vue#beers-contribution-guide)を見ます。

> Be sure to read the [official ESLint guide](https://eslint.org/docs/latest/developer-guide/working-with-rules) before you start writing a new rule.

とのことですので、[official ESLint guide](https://eslint.org/docs/latest/developer-guide/working-with-rules)を早速見てみます。


> Each rule in ESLint has three files named with its identifier (for example, no-extra-semi).
> - in the lib/rules directory: a source file (for example, no-extra-semi.js)
> - in the tests/lib/rules directory: a test file (for example, no-extra-semi.js)
> - in the docs/src/rules directory: a Markdown documentation file (for example, no-extra-semi.md)

`lib/rules`の中にルール名.jsのファイルがあるようです。`tests/lib/rules`、`docs/src/rules`にも同様にルール名.jsでファイルがあるようです。今回は`component-name-in-template-casing`のバグですので、`component-name-in-template-casing.js`を修正すれば良さそうです。


ESLintの公式ドキュメントをもう少し見てみますと、

> create (function) returns an object with methods that ESLint calls to “visit” nodes while traversing the abstract syntax tree (AST as defined by ESTree) of JavaScript code:

と記述されており、どうやらcreateの中にチェックを行う処理を書くようになっているようです。


eslint-plugin-vueのContribution Guideの方に戻りますと、

> To see what an abstract syntax tree (AST) of your code looks like, you may use AST Explorer. After opening AST Explorer, select Vue as the syntax and vue-eslint-parser as the parser.

と書かれており、ASTについても知っておく必要があるようです。
ASTというのはソースコードを解析してtree状で表現したもののようです。
[AST Explorer](https://astexplorer.net/)というものがあり、こちらに実際のJavaScriptコードをいれてみるとどのように分解されるのかがイメージ付きます。


なんとなくeslintの仕組みは理解できて、`component-name-in-template-casing.js`の`create`を修正すれば良さそうということはわかりました。ただ、具体的にどこをどう修正すれば良さそうかがまだイメージがつかなかったのでもう少し調査を進めます。


eslintではカスタムルールを作成できるということは知っていたので、カスタムルールを試しに作成してみて学習することにしました。
カスタムルールの作成に関しては、以下の2つの記事が参考になりました。

https://qiita.com/mysticatea/items/cc3f648e11368799e66c
https://techblog.yahoo.co.jp/javascript/how-to-create-eslint-rules/

こちらの記事を参考にして実際に簡単なカスタムルールを作ってみることで概ねESLintの仕組みが理解できました。


## 開発環境を構築する

ESLintの仕組みが概ね理解できたので、ローカルに開発環境を構築します。

[eslint-plugin-vue](https://github.com/vuejs/eslint-plugin-vue)をforkしてローカルにソースコードをcloneします。
`npm install`を実行して、とりあえずテストが動くかを確認します。
`package.json`を見ますと、

```json:package.json
"scripts": {
  ・・・
  "start": "npm run test:base -- --watch --growl",
  "test:base": "mocha \"tests/lib/**/*.js\" --reporter dot",
  "test": "nyc npm run test:base -- \"tests/integrations/*.js\" --timeout 60000",
  ・・・
}  
```

とあるので、`npm run test`を実行しておけば良さそうです。

```bash
❯ npm run test  

> eslint-plugin-vue@9.8.0 test
> nyc npm run test:base -- "tests/integrations/*.js" --timeout 60000

・・・

  9367 passing (1m)
```

テストは問題なく実行できました。
これで開発ができる状態になりました。


## 修正する

開発環境が整いましたのでどのように修正をしてくか考えます。

### 修正方針を決める

#### 期待動作を確認する

eslint-plugin-vueのメンテナーの方から、

> type-only imports should always be ignored in <script setup> for this rule.

というコメントをいただいており、type-only importの場合はlinterでのチェックをしないようにするのが良さそうです。
今回の修正対象のルールである[component-name-in-template-casing](https://eslint.vuejs.org/rules/component-name-in-template-casing.html)を見ると、optionに`registeredComponentsOnly`という設定があるため、これも考慮しておく必要がありそうです。
まとめると、以下の動作となるように修正する必要があることがわかりました。
- type-only import かつ `registeredComponentsOnly=false`の場合: linterでチェックしない
- type-only import かつ `registeredComponentsOnly=true`の場合: linterでチェックする

#### 修正箇所のあたりを付ける

次にソースコードのどのあたりを修正するのが良いかあたりを付けます。
`/lib/rules/component-name-in-template-casing.js`のコードを見ると、以下でregisteredComponentsにコンポーネント名と思われるものを追加している箇所があります。

```js:component-name-in-template-casing.js
if (utils.isScriptSetup(context)) {
  ・・・
    for (const variable of (moduleScope && moduleScope.variables) || []) {
      registeredComponents.add(variable.name)
    }
  ・・・
}
```

さらにもう少しコードを読み進めますと以下でチェック対象かどうかを判定しているところがあります。
```js:component-name-in-template-casing.js
function isVerifyTarget(node) {
  ・・・
  // We only verify the registered components.
  return registeredComponents.has(casing.pascalCase(node.rawName))
}
```

修正方法は`registeredComponents.add`の前でtype-only importかどうかをチェックして、type-only importの場合はaddしないようにするという方針が良さそうです。


#### 修正方法を検討する

対象のimport文がtype-only importかどうかをチェックするロジックが必要になるため、次にこのチェック方法を検討します。

AST Explorerで`import { type Component } from 'vue'`を入力して解析結果を見てみると

![](https://storage.googleapis.com/zenn-user-upload/ddbdee16d2ae-20230108.png)

となっており、`importKind: type`でチェックできそうです。


`component-name-in-template-casing.js`のソースコードを再び見てみますと、contextオブジェクトから`getSourceCode()`で対象のコードを取得して、いくつかのプロパティにアクセスして`variable.name`でコンポーネント名を取得しています。

```js:component-name-in-template-casing.js
if (utils.isScriptSetup(context)) {
  // For <script setup>
  const globalScope = context.getSourceCode().scopeManager.globalScope
  if (globalScope) {
    // Only check find the import module
    const moduleScope = globalScope.childScopes.find(
      (scope) => scope.type === 'module'
    )
    for (const variable of (moduleScope && moduleScope.variables) || []) {
      registeredComponents.add(variable.name)
    }
  }
}
```

variableからどういう情報が取得できるか[Variable interfaceのドキュメント](https://eslint.org/docs/latest/developer-guide/scope-manager-interface#variable-interface)を見てみます。
`Variable.def`が`Definition[]`型となっており、さらに[Definition interfaceのドキュメント](https://eslint.org/docs/latest/developer-guide/scope-manager-interface#definition-interface)にimportに関する記述があるのでここでtype-only importかどうかの判定ができそうです。

| type | node |
| ---- | ---- | 
| "ImportBinding" |	ImportSpecifier, ImportDefaultSpecifier, or ImportNamespaceSpecifier |


ImportBindingあたりを使ってtype-only importかどうかを判定できそうということはわかりましたが、既存のeslint-plugin-vueのルールにも同様の箇所があるのではないかと思い、念のため他のルールを見てみました。すると、[no-undef-componentsのコード](https://github.com/vuejs/eslint-plugin-vue/blob/master/lib/rules/no-undef-components.js)に以下のコードがあり、このロジックを使えば良いことがわかりました。

```js:no-undef-components.js
(variable.defs.length > 0 &&
  variable.defs.every((def) => {
    if (def.type !== 'ImportBinding') {
      return false
    }
    if (def.parent.importKind === 'type') {
      // check for `import type Foo from './xxx'`
      return true
    }
    if (
      def.node.type === 'ImportSpecifier' &&
      def.node.importKind === 'type'
    ) {
      // check for `import { type Foo } from './xxx'`
      return true
    }
    return false
  }))
```



### 修正する

修正方法がわかりましたので、コードの修正をします。

#### テストコードを修正する

先にまずはテストコードを修正して、unit testがエラーになることを確認します。
テストコードに以下のようなテストケースを追加しました。

```diff:tests/lib/rules/component-name-in-template-casing.js
  valid: [
    ・・・
+   {
+     code: `
+       <script setup lang="ts">
+         import type Foo from './Foo.vue'
+         import type { HelloWorld1 } from './components/HelloWorld'
+         import { type HelloWorld2 } from './components/HelloWorld2'
+         import type { HelloWorld as HelloWorld3 } from './components/HelloWorld3'
+         import { type HelloWorld as HelloWorld4 } from './components/HelloWorld4';
+         import { type default as HelloWorld5 } from './components/HelloWorld5';
+         import { type Component } from 'vue';
+       </script>
+       <template>
+         <foo />
+         <hello-world1 />
+         <hello-world2 />
+         <hello-world3 />
+         <hello-world4 />
+         <hello-world5 />
+         <component />
+       </template>
+     `,
+     options: ['PascalCase', { registeredComponentsOnly: true }],
+     parserOptions: {
+       parser: require.resolve('@typescript-eslint/parser')
+     }
+   }
  　・・・
  ]

```

#### バグを修正する

そしてルールのバグを以下のように修正します。

```diff:lib/rules/component-name-in-template-casing.js
+ function isTypeOnlyImport(variable) {
+   if (variable.defs.length === 0) return false
+ 
+   return variable.defs.every((def) => {
+     if (def.type !== 'ImportBinding') {
+       return false
+     }
+     if (def.parent.importKind === 'type') {
+       // check for `import type Foo from './xxx'`
+       return true
+     }
+     if (def.node.type === 'ImportSpecifier' && def.node.importKind === 'type') {
+       // check for `import { type Foo } from './xxx'`
+       return true
+     }
+     return false
+   })
+ }
  ・・・
  module.exports = {
    ・・・
    create(context) {
      if (utils.isScriptSetup(context)) {
        // For <script setup>
        const globalScope = context.getSourceCode().scopeManager.globalScope
        if (globalScope) {
          // Only check find the import module
          const moduleScope = globalScope.childScopes.find(
            (scope) => scope.type === 'module'
          )
          for (const variable of (moduleScope && moduleScope.variables) || []) {
-           registeredComponents.add(variable.name)
+           if (!isTypeOnlyImport(variable)) {
+             registeredComponents.add(variable.name)
+           }
          }
        }
      }  
・・・
```

修正後にunit testを実行して無事にpassすることを確認できました！

### PRを出す

最後にPull requestを出して完了です。
すぐにレビューをしてもらえて無事にマージされました！
https://github.com/vuejs/eslint-plugin-vue/pull/2069


# まとめ

OSSコントリビュートのメリットの1つとしてライブラリに詳しくなれることがあると思います。今回のeslint-plugin-vueも正にそのとおりで、issueの修正を通してeslint-plugin-vueの仕組みやカスタムルールの作り方を習得することができました。
簡単なカスタムルールなら作れそうな気がしたので、実際のプロジェクトでもカスタムルールの作成にチャレンジしてみようと思います。