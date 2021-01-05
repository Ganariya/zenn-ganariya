---
title: "Python中級者になるためにModern Pythonを目指す"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['python', 'modern']
published: false
---

# はじめに

## 本記事の読者対象

- Pythonに慣れてきて、新しい文法・開発環境を覚えたい方
- よりモダンに近いPython環境が欲しい方

## 想定していない方

- Python自体が初めての方
- Python上級者


## 説明すること・しないこと

説明する

- ツールのおおまかな説明
- ツールを使用する理由・嬉しさ
- 参考になるドキュメント・URL

説明しない

- 具体的なコマンド


# Modern Python

大学院で研究をするようになってから、長い時間Pythonを書くようになりました。
なぜならば、研究で利用しやすいライブラリが豊富、かつ研究のようなイテレーションがはやいプロジェクトに対して非常に有効であるためです。

しかし、Pythonは短期的にコードを試して動作を変更できる分、安定した動作が難しくなってきます。
例えば、C++などはコンパイルを通す必要があるため、設計をうまく考えて実装する必要があります。
一方、Pythonはスクリプト言語なので最悪`闇の設計`をすると実装考えなくてもなんとかなります。
ただ、このような後先考えないコードは将来的に大きな負債になります。
実際、締め切りが近い論文に合わせて急ピッチで書いたコードが現在自分の前に立ちはだかり、また1から設計・実装し直す羽目になりました。

Pythonは良くも悪くも以下のような特徴があります。

- 型を設定しなくていい（その分、変数名に思いを馳せる）
- モジュールが多い（バージョンがずれる）
- パッケージマネージャも豊富（どれを使えばいいのかが分かりづらい）
- 設計が甘くて良い（読み辛くなる）


今回は、これらの問題を解決できる可能性があるツールについてまとめます。
もっと自分もはやくPythonに慣れていきたいですね。


# パッケージマネージャ

Pythonのパッケージマネージャには多くの種類の組み合わせがあります。
以前[pythonの環境構築戦争にイラストで終止符をどうやら打てない](https://qiita.com/ganariya/items/1bf870275bad7b5ab506)という記事でPythonの環境の構築について説明しました。
そこから自分の理解もある程度進み、今のPythonの環境構築これでいいかな〜と感じるものがあったため、それについてまとめようと思います。

## パターン1: pyenv + pipenv

`pyenv`は**複数のPythonのバージョンを管理**することができます。
例えば、Python3.7、Python3.8、Python3.9をそれぞれインストールして好きに切り替えることが出来ます。
バージョンを切り替えることによって、複数のプロジェクトに対応することができます。
昔から続いているプロジェクトのコードはPython2.7でしか動かないんだ！みたいなこともあります。
そのため、pyenvでバージョンを切り替えられるとうれしいわけです。

しかし、`pyenv`はバージョンを切り替えることはできますが、仮想環境を作成することはできません。
この仮想環境とはプロジェクトごとに利用する環境のことを本記事では指します。

もし仮想環境を切り分けられない場合を考えてみます。
機械学習のプロジェクト開発を行った後、別のプロジェクトに割り当てられDjangoの開発になったとします。
このとき、機械学習のpytorchのライブラリはDjangoに直接必要はありません。
大量にライブラリが増えてくると動作も重くなり、必要なバージョンの整合性が取れなくなってきます。
そのため、プロジェクトごとにライブラリをインストールできるような仮想環境が必要なわけです。

そして、`pipenv`は、`あるバージョンのPythonにおいて、複数の仮想環境を作成する`ことができます。
venvをラップするような便利な機能がpipenvにたくさん備わっているのでおすすめです。
また、ここでは詳しくは述べませんが、バージョンのロックを行うことが出来ます。

まとめると、`pyenv`でPython自体のバージョンを管理し、`pipenv`で特定のPythonのバージョンにおける仮想環境を管理します。
Macではbrewでpipenv, pyenv両方インストールできます。

## パターン2: pyenv + poetry

pyenvは先程出てきました。
今回、新しく出てきたのはpoetryになります。

poetryはrustっぽくライブラリ管理ができるマネージャのようです。
まだ触ったことがないため詳しくないのですが、tomlファイル一つですべてを管理するらしいです。

かなり新しいマネージャのようなので、今度触ってみたいです。


## 参考URL

- [poetry](https://python-poetry.org/docs/)
- [pipenv](https://pipenv-ja.readthedocs.io/ja/translate-ja/basics.html)
- [Pythonのパッケージ周りのベストプラクティスを理解する](https://www.m3tech.blog/entry/python-packaging) おすすめです

# 型の導入

Pythonは型付けをしなくても動くスクリプト言語です。
そのため、初期段階ではかなりのスピードで開発できますが、後半になってくると変数に思いを馳せる時間が長くなり、実行時エラーに悩まされることが多くなります。

そこで、Python3.5から導入された`typing`などを利用して型安全にコーディングすることができます。
しかし、**実行時に型が異なってもエラーを出さない**ことに注意が必要です。
実行時に止まるわけではなく、IDEなどでおかしい部分を指摘されやすくなります。

## typing

```python
def greeting(name: str) -> str:
    return 'Hello ' + name
```

Python3.5から、上記のように変数や関数などに型を指定できるようになりました。
型を指定することで、IDEの恩恵を受けやすくなります。
また、長期的に見るとコーディングの効率化に繋がります。

`Final`キーワードもあり、より定数などを安全に設定することが出来ます。


詳しくは参考URLを御覧ください。

## data classes

データを格納するクラスをより簡単に作成するデコレータなどを提供します。

```python
rom dataclasses import dataclass

@dataclass
class InventoryItem:
    """Class for keeping track of an item in inventory."""
    name: str
    unit_price: float
    quantity_on_hand: int = 0

    def total_cost(self) -> float:
        return self.unit_price * self.quantity_on_hand
```

## namedtuple

名前をつけたタプルを宣言できます。
書き換えられないデータを保証できるので、変更しないものを管理するには便利そうです。

## ジェネリクス

## pydantic

FastAPIで採用されているライブラリです。
実行時に型情報を提供してくれます。

```python
from datetime import datetime
from typing import List, Optional
from pydantic import BaseModel


class User(BaseModel):
    id: int
    name = 'John Doe'
    signup_ts: Optional[datetime] = None
    friends: List[int] = []
    
external_data = {
    'id': '123',
    'signup_ts': '2019-06-01 12:22',
    'friends': [1, 2, '3'],
}
user = User(**external_data)
```

型があっていないようなデータが入ると**実行時でも**例外を投げてくれます。
（一方typingなどは投げません。そのため、より型でおかしい箇所がわかりやすく、堅牢性が上がります。）

## 参考URL

- [typings](https://docs.python.org/ja/3/library/typing.html)
- [dataclasses](https://docs.python.org/ja/3.9/library/dataclasses.html)
- [namedtuple](https://docs.python.org/ja/3/library/collections.html#collections.namedtuple)

