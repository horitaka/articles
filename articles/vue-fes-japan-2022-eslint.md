---
title: "Vue Fes Japan Online 2022 レポート - eslint-plugin-vueを使用して継続的にVue3移行する"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vue, eslint]
published: true
---
:::message
> 引用文で書かれている箇所が講演の内容です。
> (一言一句あっているわけではないので正確な内容は動画をご覧ください)
:::

# 概要

Vue Fes Japan Online 2022の視聴レポートです。

**eslint-plugin-vueを使用して継続的にVue3移行する[(動画)](https://www.youtube.com/watch?v=dtD4p89ogKM&t=21916s)**
https://vuefes.jp/2022/sessions/ota-meshi

> Vue2のアプリケーションをVue3に移行するニーズは増えてきていると思いますが、複数のアプリケーションを移行する場合、手動で実施するのはコストもかかりますし、Vue2をメインに使用してきたエンジニアが、新しいVue3でのアプリケーション開発で古い機能を使ってしてしまうというリスクもあります。
eslint-plugin-vueは古い書き方を検知し、場合によって自動的にVue3の新しい書き方に書き換えることで、継続的にVue3移行を助けます。

こちらに講演資料も公開されています。
https://ota-meshi.github.io/vue-fes-japan-online-2022-slide/1

# 講演メモ&感想

## Vue3のBreaking Changeについて
> - vue3のBreaking Changeは38項目
> - https://v3-migration.vuejs.org/breaking-changes/
> - Vue3のBraking ChangeをESLintでチェックできる

Breaking Changeが多いのでVue3移行は大変ですね。
業務で開発しているコードもeslint-plugin-vueでチェックをしていて普段からお世話になっています。


## eslint-plugin-vueとは
> - ESLintはJavaScriptのコードをチェックするリンター
> - eslint-plugin-vueは.vueファイルの解析、Vueに特化した検証ルールを提供
> - Vue3移行に便利なルールもいくつか提供

チームで開発するならVueならeslint-plugin-vueは必須ですね。


## eslint-plugin-vueを利用してVue3移行してみた
> - [eslint-plugin-vue](https://eslint.vuejs.org/)を使えばVue3のBreaking Changeの半分ぐらいがチェック可能
> - ESLint本体に付属しているルールを使用すればさらに5項目のチェックが可能
> - [eslint-plugin-vue-scoped-css](https://future-architect.github.io/eslint-plugin-vue-scoped-css/)を使えばさらに1項目のチェックが可能
> - それ以外は社内用のESLintプラグインやすぐに見つけられるものだったのでESLintルール無しで対応

eslint-plugin-vueのVue3でdeprecatedになった機能をチェックするルールは知っていましたが、ESLint本体も使えるとのことです。
ESLint本体も使えば自動チェックできる部分も結構カバーできるんですね。


## Vue3移行にESLintを使うメリット
> - 自動でチェックできる
> - 別のプロジェクトにも流用できる
> - Vue3移行後にVue2でしか動かないコードを書かれない
> - 社内専用のルールはカスタムルールを作れば対応できる

自動でチェックできるとか別のプロジェクトでもルールを共通にできるというのはメリットとして思っていましたが、Vue3移行後にもVue2のコードを書かれないようにするのはたしかにそうだなあと思いました。
ブログやネット上の記事を見ても古いものであればVue2でしか動かないコードが書かれていたりするので、あまり経験のないメンバーがそういうコードを書いてしまうのを防げると思いました。

## まとめ
> - Breaking Changeの65%(26/40項目)はESLintでチェック可能
> - 移行作業の95%ぐらいは自動でチェック
> - カスタムルールを作ればほぼ自動チェックできる

Vue3移行作業の95%ぐらいの作業がカバーできるというのは嬉しいですね。
業務でメンテしているプロジェクトの中にもまだVue2のものがあるので、これまでに知らなかったものもlintルール含めて活用していきたいと思います。
カスタムルールを作るのはちょっとハードルありそうですが、自動でチェックできるところが多そうであればチャレンジしてみようかなと思います。