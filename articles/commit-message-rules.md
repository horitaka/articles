---
title: "commitlintとCommitizenを使ってチーム開発におけるコミットメッセージを統一する"
emoji: "📗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [git, linter, husky, チーム開発]
published: true
---

チーム開発においてはいろいろと開発ルールを決めることがあります。
今回の記事では、コミットメッセージを統一するための方法を紹介します。
尚、本記事はNode.jsのビルド環境を想定したものです。


# コミットメッセージをどうするか

コミットメッセージをチームで統一する際にまず始めに決めることはどのようなメッセージのルールとするかです。
もちろんゼロから作っても良いのですが世の中には先人たちの知恵が詰まった事例があるのでそれを活用します。
コミットメッセージの規約として、Conventional Commitsというものがあります。
私のチームではコミットメッセージのルールとしてConventional Commitsを採用しています。

https://www.conventionalcommits.org/ja/v1.0.0/

> Conventional Commits の仕様はコミットメッセージのための軽量の規約です。 明示的なコミット履歴を作成するための簡単なルールを提供します。

## Conventional Commitsの主な内容

Conventional Commitsの規約ではコミットメッセージは以下のような形式になっています。

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

typeには、`feat`、`fix`、`docs`などが入ります。

scopeには、対象の機能のスコープを記述します。


**コミットメッセージの例**
```
feat(lang): add polish language
```

その他にも公式サイトにはコミットメッセージの例がたくさん書いてあります。
https://www.conventionalcommits.org/ja/v1.0.0/#%E4%BE%8B


# commitlintで自動でチェックする

コミットメッセージは決めたもののこれを開発者自身やレビューアーがチェックするというのも無駄な作業です。
自動でコミットメッセージが規約に沿っているかをチェックするツールとしてcommitlintがありますのでこれを活用します。

https://commitlint.js.org/#/

## commitlintを導入する

#### 1. インストール
以下のライブラリをインストールします。
```bash
npm install --save-dev @commitlint/cli @commitlint/config-conventional
```

#### 2. 設定追加
commitlintの設定ファイルを追加します。
```js:commitlint.config.js
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```

尚、package.jsonに記述することも可能です。
```json:package.json
{
  ...
  "commitlint": {
    "extends": [
      "@commitlint/config-conventional"
    ]
  },
  ...
}
```

#### 3. 動作チェック
適当なメッセージを入力してコミットします。
```bash
git commit -m "規約に沿っていないコミットメッセージ"
```

commitlintを実行します。
```bash
❯ npx commitlint --from=HEAD~1 
⧗   input: 規約に沿っていないコミットメッセージ
✖   subject may not be empty [subject-empty]
✖   type may not be empty [type-empty]

✖   found 2 problems, 0 warnings
ⓘ   Get help: https://github.com/conventional-changelog/commitlint/#what-is-commitlint
```

無事にエラーが表示されることを確認できました。


## Gitフック`commit-msg`でコミット時に自動でチェックする

毎回、commitlintを実行するのも手間なので、Gitフックの`commit-msg`を使用して、コミット時にcommitlintが毎回実行されるようにします。

https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BA-Git-%E3%83%95%E3%83%83%E3%82%AF

> commit-msg フックは、開発者の書いたコミットメッセージを保存した一時ファイルへのパスをパラメータに取ります。 このスクリプトがゼロ以外の値を返した場合、Git はコミットプロセスを中断します。これを使えば、コミットを許可して処理を進める前に、プロジェクトの状態やコミットメッセージを検査できます。


#### 1. huskyのインストール
```bash
npm install husky --save-dev
```
#### 2. commit-msgフックの導入
```bash
npx husky add .husky/commit-msg  'npx --no -- commitlint --edit ${1}'
```

これでコミット時に自動でcommitlintが実行されるようになりました。


# Commitizenを使ってプロンプトから入力する

このままでも良いのですが、手作業でコミットメッセージを入力する方法では毎回ルールを確認する必要があります。
そこでコミットメッセージの入力を補助するツールとしてCommitizenを使用します。
Commitizenとはプロンプト形式で規約に沿ったコミットメッセージを入力できるようにするものです。

http://commitizen.github.io/cz-cli/

> When you commit with Commitizen, you'll be prompted to fill out any required commit fields at commit time. 

## Commitizenを導入する

#### 1. インストール
```bash
npm install --save-dev commitizen
```

```bash
npx commitizen init cz-conventional-changelog --save-dev --save-exact
```

#### 2. 設定追加
```json:package.json
{
  ...
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    }
  }
  ...
}
```

#### 3. npm scriptsに追加
```json:package.json
{
  ...
  "scripts": {
    ...
    "commit": "cz",
    ...
  }
  ...
}
```

これで`npm run commit`を実行することで以下のようなプロンプトが表示され、案内に従って入力することで規約に沿ったメッセージを入力できます。

```bash
? Select the type of change that you're committing: (Use arrow keys)
❯ feat:     A new feature 
  fix:      A bug fix 
  docs:     Documentation only changes 
  style:    Changes that do not affect the meaning of the code (white-space, formatting, missing semi-colons, etc) 
  refactor: A code change that neither fixes a bug nor adds a feature 
  perf:     A code change that improves performance 
  test:     Adding missing tests or correcting existing tests   

? What is the scope of this change (e.g. component or file name): (press enter to skip) 
? Write a short, imperative tense description of the change (max 94 chars):
...
```

## Gitフック`prepare-commit-msg`でコミット時に自動でチェックする

Gitフックの`prepare-commit-msg`を使用して、`git commit`コマンドを実行したときに毎回commitizenが実行できるようにすることも可能です。

https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BA-Git-%E3%83%95%E3%83%83%E3%82%AF
> prepare-commit-msg フックは、コミットメッセージエディターが起動する直前、デフォルトメッセージが生成された直後に実行されます。 このフックでは、デフォルトメッセージを、コミットの作者の目に触れる前に編集できます。

```bash
npx husky add .husky/prepare-commit-msg  'exec < /dev/tty && node_modules/.bin/cz --hook || true'
```

これで`git commit`を実行するとプロンプトが立ち上がるようになります。
