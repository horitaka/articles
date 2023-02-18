---
title: "Pythonの型ヒント(typing)とdataclassでチームのコード品質と開発効率を上げる"
emoji: "💡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [python, チーム開発, lint]
published: true
---

Pythonの[型ヒント(typing)](https://docs.python.org/ja/3/library/typing.html)と[dataclass](https://docs.python.org/ja/3/library/dataclasses.html)をプロジェクトに導入したという記事です。
私が担当しているプロジェクトでは、それまで外注で作っていたプロダクトを内製開発に切り替えることになりました。外注をしていたためコードは外部のベンダーで作っていたのですが、テストをすると実行時エラーが発生したり、コードの内容を理解するのに時間がかかるという課題が発生していました。
また、いわゆるBIツールのbackendのAPIサーバーとして機能するもので、複雑なデータを扱うものでした。それにも関わらず、データの型が定義されておらず非常にコードが理解しづらい状況でした。

そこで、型ヒント(typing)とdataclassを導入することでチームのコード品質と開発効率化につながったことから、どのような取り組みをしてきたかを紹介させていただきます。


# プロジェクトを引き継いだ時の状態

- コードは数万行
- APIエンドポイントは50〜60程度
- 関数の戻り値がdictやtupleになっていて要素数が多いと10以上というものもある
- 今後も継続的に改修や機能追加が発生する

上記のような状態で、改修をするにも都度戻り値をログで出力してみないと実装がわからないという状況でした。


# 最初に型ヒントを導入する

まず最初に簡単にできることとして型ヒントを導入することとしました。

## 型ヒントとは

型ヒントはPython3.5から導入された機能です。変数や関数の引数・戻り値に型を付けることができます。

```python
def greeting(name: str) -> str:
    return 'Hello ' + name
```

https://docs.python.org/ja/3/library/typing.html


## チームへの導入方針

すでに数万行程度のコードがある中で既存コード含めてすべてに型ヒントを導入することは現実的ではありませんでした。
そこで以下のような方針を決めてチームに導入することとしました。

- Publicな関数(外部から呼び出される関数)には型ヒントをつける
- 型の付け方がわからない場合は一旦型ヒント無しでPull requestを出すことも許容する
- 上記を新規に作成するコードや改修するコードに適用する

チーム内にPythonの経験者が少なかったことからまずは簡単にできるように上記の方針としました。


# 次にdataclassを導入する

次に、dataclassを導入することとしました。
複雑なデータを多数扱うことからdataclassを導入することはチームにとっても理にかなったものでした。

## dataclassとは

dataclassとはPython3.7から追加された機能です。
`@dataclass`というデコレーターを追加することで、classに対して`__init__()` や` __repr__()` のようなメソッドを自動で追加してくれる機能です。
dataclassとして定義しておくことでどういった値が入ってくるのかが理解しやすくなる他にエディタの補完が効くというメリットもあります。

```python
from dataclasses import dataclass

@dataclass
class User:
    first_name: str
    last_name: str
    age: int

    def get_full_name(self) -> str:
        return f"{self.first_name} {self.last_name}"
```

これは以下と同様です。

```python
class User:
    first_name: str
    last_name: str
    age: int

    def __init__(first_name: str, last_name: str, age: int):
        self.first_name = first_name
        self.last_name = last_name
        self.age = age

    def get_full_name(self) -> str:
        return f"{self.first_name} {self.last_name}"
```

https://docs.python.org/ja/3/library/dataclasses.html

## チームへの導入方針

dataclassは以下のような方針で導入しました。

- 単純な組み込み型(intやstrなど)以外を返す関数の戻り値はdataclassで定義する
- 上記を新規に作成するコードや改修するコードに適用する


# mypyを導入し静的解析を実行する

型ヒントとdataclassを導入すれば、エディタの機能だけでもある程度はエラーや問題のあるコードが混入することは防げます。
mypyを導入することでこのチェックをより強化することにしました。


## mypyとは

mypyとは型チェックを行ってくれるツールです。

https://github.com/python/mypy
> Mypy is a static type checker for Python.
>
> Type checkers help ensure that you're using variables and functions in your code correctly. With mypy, add type hints (PEP 484) to your Python programs, and mypy will warn you when you use those types incorrectly.


## チームへの導入方針

- pre-commitを使ってコミット前にmypyでの型チェックを実行する
- CIでmypyを実行する

pre-commitというのは、Git Hooksを利用してコミット前に特定の処理を追加できるものです。また(名前はpre-commitですが)コミット以外にもプッシュやマージの前後などにも特定の処理を追加できます。
https://pre-commit.com/

コミット前にLinterやFormatterを実行したり、プッシュ前にローカルでテストを実行するといった使われ方が多いと思います。
私のチームでもコミット前にmypyを実行することで開発者が早期にエラーに気づけるようにしました。

あわせて、Pull request作成時にもmypyを実行するようにしてエラーがコードベースに混入しないようにしました。


また、エディタと組み合わせてファイル保存時に自動でチェックするということも任意で実施しました。
PyCharmへの導入方法は以下の記事も参考にしていただければと思います。
https://zenn.dev/horitaka/articles/4fc4a5c19bee22


# 取り組みの結果

現在も改修は続いていますが、これまでの経験として以下のような効果がありました。

## 開発者向けの効果
- 改修時にコードを読むだけでどういった値が入ってくるのかがある程度理解できる
- 型が一致しないような処理を書いたときに早期に問題に気づける
  - これまでは実行してみないと気づけなかった
- エディタの補完が効く

## レビューアー向けの効果
- 関数の定義を見ればどんな値が入力と出力になるのかがわかり、レビューしやすくなった
- 設計の誤りにもレビュー段階で早期に気づけるようになった
