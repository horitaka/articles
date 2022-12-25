---
title: "OSSにコントリビュートした(Vue I18n) - APIリファレンスの修正"
emoji: "🌎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Vue.js, JavaScript, VueI18n, OSS]
published: true
---

# 概要

Vue.jsの多言語化のためのライブラリであるVue I18nにコントリビュートをしました。
OSSコントリビュートをやってみようとなるとハードルは高いと感じていらっしゃる方もいると思いますが、ドキュメント修正であれば比較的簡単にできますので最初にやるものとしてはお勧めです。
OSSコントリビュートに興味がある方の参考になれば幸いです。


## 対象のライブラリ
[Vue-I18n](https://github.com/kazupon/vue-i18n)

> Internationalization plugin for Vue.js

## issueの内容
https://github.com/kazupon/vue-i18n/issues/1528

> availableLocales option is wrong docs


# コントリビュートまでの実施内容

## 問題内容と期待動作を把握する

まず、問題の内容と期待動作を確認します。
issueを見ると、

> docs say availableLocales can be passed to the constructor option.
That is incorrect documentation
We need to fix it.

と書いてあります。

[APIリファレンス](https://kazupon.github.io/vue-i18n/api/)を見るとavailableLocalesがconstruction optionsに入っており、これが間違っているようです。
![修正前のドキュメント](https://storage.googleapis.com/zenn-user-upload/8651b0d01cad-20221225.png)


ではどう修正するのが良いかもう少しissueを見て見ますと別のissueへのリンクが貼ってあり、そちらには以下のように記述されております。

https://github.com/kazupon/vue-i18n/issues/1523#issuecomment-1185475275

> availableLocales is read-only property on the VueI18n instance.

read-onlyということですので、APIリファレンスのconstruction optionではなく、propertiesの方に移動すれば良さそうです。


## 開発環境を構築する


### ローカルに環境を構築する
まずローカルで動作できるように環境を構築します。
Vue I18nへのコントリビュートは以前にも実施したことがあるため、以下の**1.開発用にビルドする〜　5.すべてのテストを実行する**に記載されている方法と同じ手順で構築します。
https://zenn.dev/horitaka/articles/41f325f4b2c20e


### ドキュメントの修正方法を確認する
今回はドキュメントの修正になるため、ドキュメントの修正対象ファイルと生成方法を確認します。

ディレクトリを見ると[vuepress](https://github.com/kazupon/vue-i18n/tree/v8.x/vuepress)というディレクトリがあり、この中にmdファイルが多数あります。
いくつかファイルを見てみるとこの中の[api/README.md](https://github.com/kazupon/vue-i18n/blob/v8.x/vuepress/api/README.md)が該当のファイルのようです。

次に、ドキュメントの生成方法を確認します。
package.jsonのscriptsを確認すると、
```json:package.json
npm run docs:dev
```
という記述がありますので、おそらくこれが該当しそうです。

試しに、README.mdを何か適当に修正してローカルで起動したhtmlを見てみると、修正が反映されていることが確認できました。


## 修正方針を決める
開発環境が整いましたのでどのように修正をしてくか考えます。

今回の修正は、availableLocalesの記述場所が間違っているというだけですので、以下の記述をconstructor opsionsからpropertiesのセクションに移動するのみで良さそうです。
```md:README.md
#### availableLocales
> :new: 8.9.0+
  * **Type:** `Locale[]`
  * **Read only**
The list of available locales in `messages` in lexical order.
```

## 修正する

修正方針通りに修正して、無事にpropertiesに移動されていることが確認できました。
![修正後のドキュメント](https://storage.googleapis.com/zenn-user-upload/76d7463edfbe-20221225.png)


## PRを出す

これで修正が一通り終わったので、[Pull Request Guidelines](https://github.com/kazupon/vue-i18n/blob/dev/CONTRIBUTING.md#pull-request-guidelines)を見て、書いてあるとおりにPRを出します。

ドキュメントの修正なのであまり詳しい説明もいらないかとは思いましたが、修正前後のドキュメントのスクリーンショットを貼ってわかりやすくしておきました。
https://github.com/kazupon/vue-i18n/pull/1577

1週間ほどでレビューいただいて無事マージされました！