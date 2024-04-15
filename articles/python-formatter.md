---
title: "PythonのFormatterとLinterの設定(Flake8, Black, mypy, isort) + VSCodeの設定"
emoji: "🛺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [python, vscode, linter, formatter]
published: true
---


## 概要
PythonのFormatterとLinterの紹介とサンプルコードです。
VSCodeでの設定方法も紹介します。
この記事で紹介するコードは下記のリポジトリにあります。
https://github.com/horitaka/python-formatter


## Formatter/Linterの説明
FormatterとLinterは下記のライブラリを使用します。

[Flake8](https://flake8.pycqa.org/en/latest/)
> Your Tool For Style Guide Enforcement

[Black](https://black.readthedocs.io/en/stable/)
> By using Black, you agree to cede control over minutiae of hand-formatting. 

[isort](https://pycqa.github.io/isort/)
> isort is a Python utility / library to sort imports alphabetically, and automatically separated into sections and by type.

[mypy](http://www.mypy-lang.org/)
> Mypy is an optional static type checker for Python that aims to combine the benefits of dynamic (or "duck") typing and static typing.


## Formatter/Linterの設定
それぞれのFormatterやLinterは特別な設定はなくても使い始められますが、それぞれのツールで競合が発生することもあるため、少し設定が必要です。

### Flake8
Flake8では一行あたりの最大文字数のデフォルト値は79文字です。
Blackでは88文字です。
それぞれの設定が異なると競合が発生してしまうため統一します。
今回は119文字としました。

```toml:.flake8
[flake8]
max-line-length = 119
```

### Black
Flake8と設定を揃えるために一行あたりの最大文字数を設定します。

```toml:pyproject.toml
[tool.black]
line-length = 119
```

### isort
Blackの設定と競合する部分があるためprofileを設定します。

```toml:pyproject.toml
[tool.isort]
profile = "black"
```

### mypy
特に設定はせず初期状態のまま使用します。


**参考**
[Compatibility with black](https://pycqa.github.io/isort/docs/configuration/black_compatibility.html)


## VSCodeの設定
VSCodeの設定をします。
ファイルを保存したときにBlackやisortが自動で実行されるようにし、ファイルが整形されるようにします。

```json:.vscode/settings.json
{
    "[python]": {
        "editor.formatOnPaste": false,
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "ms-python.black-formatter",
        "editor.codeActionsOnSave": {
            "source.fixAll": "explicit",
            "source.organizeImports": "explicit"
        },
    }
}
```
