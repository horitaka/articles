---
title: "Vue Fes Japan Online 2022 レポート - プロダクト開発を止めずにCompositionAPIとTypeScript"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Vuejs, typescript]
published: true
published_at: 2022-11-16 10:00
---

:::message
> 引用文で書かれている箇所が講演の内容です。
> (一言一句あっているわけではないので正確な内容は動画をご覧ください)
:::


# 概要
Vue Fes Japan Online 2022の視聴レポートです。

**プロダクト開発を止めずに Composition API と TypeScript に最速で移行するための戦い[(動画)](https://www.youtube.com/watch?v=HsBTx36c_kA&t=10817s)**
https://vuefes.jp/2022/sponsor-sessions/stores

> 弊社のプロダクト、ネットショップ開設サービス「STORES」は Nuxt.js で開発を行なっており、多くのページやコンポーネントは Composition API と TypeScript で書かれています。
しかし、ストアデザイン機能など、一部のUIが複雑なページは Options API と JavaScript のままであり、リファクタリングもコストが高く、機能追加や改善の手が入れづらい状態が続いていました。
本セッションでは複雑で手がつけずらいソースコードをプロダクト開発を止めることなく、且つ安全に最速で移行するために考えた戦術をご紹介します。

# 講演メモ&感想

## 技術的な課題
> - Options APIとmixinを入れないと型アサーションを入れないとTypeScriptに移行できない
> - 型アサーションを入れるのは面倒なのでOptions API + mixin を Composition API + composableにする
> - mixinをcomposableに変換して出力するツールを作成
>   - https://github.com/wattanx/vue-mixins-converter
>   - https://vue-mixins-converter.vercel.app/
> - 手動変換だと1ファイル1時間かかるものがツールでは1ファイル20秒に

Vue3に移行するならTypeScriptにも対応していきたいですが、mixinを使っているとそんな苦労があるんですねえ。
幸いにも私が今担当しているプロダクトではmixinをあまり使用していなかったです。
一定以上の規模のプロダクトの場合は、やっぱりVue3移行には何らか自動ツールが必要かなと思いました。


## 運用時の課題
> - プロダクト開発もある中で並行してVue3移行も必要
> - PdM、スクラムチームにも理解してもらう
> - 移行の準備と計画をしっかりたてる
> - issueに着手しやすいように(機能を小さくするなど)
> - モチベーション維持のためにDatadogで進捗を数値で確認できるようにした

チームの理解を得るというのはやっぱり重要ですね。
進捗を数値で可視化するというのは基本的なところではありますが、実際のプロジェクトではなかなかやっていないこともあるので重要かなと思いました。メンバーのモチベーション維持のためにやられていたとのことですが、プロジェクト管理のためにも良いなと思いました。


## まとめ
> - 準備と計画が大事
> - 1ヶ月2名で35%の機能のComposition APIへの移行が完了

全体の規模がどれぐらいなのかも気になるところですが、2名でやっても1ヶ月で35%ぐらいなんですねえ。
すべての移行には3ヶ月ぐらいということなのでしょうか。そこそこ大変ですね。
