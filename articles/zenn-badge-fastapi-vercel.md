---
title: "ZennのバッジをFastAPI+Vercelでお手軽に作る"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "fastapi", "vercel"]
published: true
---

# はじめに

`FastAPI`と `Vercel` を使って、Zenn のバッジを提供するサービス [zenn-badge](https://github.com/Ganariya/zenn-badge) を作りました。

https://github.com/Ganariya/zenn-badge

たとえば、以下のようにバッジを作ることができます。

![ganariya-liked](https://zenn-badge.ganariya.vercel.app/ganariya/liked) ![ganariya-followers](https://zenn-badge.ganariya.vercel.app/ganariya/followers) ![ganariya-articles](https://zenn-badge.ganariya.vercel.app/ganariya/articles)

```markdown
![your-text](https://zenn-badge.ganariya.vercel.app/{username}/liked)
![your-text](https://zenn-badge.ganariya.vercel.app/{username}/followers)
![your-text](https://zenn-badge.ganariya.vercel.app/{username}/articles)
```

記事を書き出してから「vercel のドメイン名なんかおかしいなぁ」になり色々調べていると
[Zenn.badge](https://zenn-badge.nikaera.vercel.app/)という、より使いやすいサービスがありました。

`Zenn.badge`も参考にさせていただいて、もっと使いやすいものにしていきたいですね。

# FastAPI

`zenn-badge`では、バックエンドに[FastAPI](https://fastapi.tiangolo.com/ja/)というフレームワークを使っています。
開発スタートはかなり最近ですが、ものすごい勢いでスター数が増加しています。

以下のような特徴があります。

- Python3.5 以降に追加された `Type Hints` を積極的に利用
  - 変数に型を設定するとそのまま自動バリデーションに 
  - IDE の恩恵も受けることができて一石二鳥
- Swagger-UI がとくに設定することなく自動で作成される
  - https://zenn-badge.ganariya.vercel.app/docs
  - 変数に型を付けると自動でスキーマにも型情報が設定される
- 非同期処理が `async/await` で意識せずに書ける
  - FastAPI は ASGI
  - バックグラウンドで走らせるタスクもとても短く書ける
- ドキュメントが分かりやすい
  - 使い方・チュートリアルが丁寧に書かれている
  - FastAPI 以外の Python の特性についても取り扱っている
    - Pydantic へのモデルに辞書を渡すときの `**dict` によるキーワード変数展開
    - ファイル・モジュールの分割方法
- GraphQL / WebSocket などへの公式サポート
  - 内部で[Starlette](https://github.com/encode/starlette)を利用している
  - [FastAPI / Starlette / Pydantic の関係性について](https://qiita.com/bee2/items/d629d8acc102cf92b7b2)

[公式のアプリケーション例](https://fastapi.tiangolo.com/ja/#_4)は以下のコードです。
Flask のような文法で書くことができます。
ここで、`def read_item(item_id: int, q: str = None)`と引数に型を設定しています。
実はこれだけで自動で引数に関するバリデーションを行い、かつ SwaggerUI で型を表示します。

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def read_root():
    return {"Hello": "World"}


@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

これから新しくバックエンド API をミニマムに始めたい場合は、Flask だけでなく FastAPI を視野に入れると良いと思います。
かなり少ない勉強量に対して大きなメリットがあります。

# vercel

[vercel](https://vercel.com/dashboard)は、サーバレスで Python/Go/Nodejs などを動かすことができるサービスです。
サーバの構築をほぼ意識することなく、自動でビルド・デプロイ・運用が行われます。
しかも、類似サービスである Heroku と比較して、かなり無料枠が大きいです。
（ほぼ無料枠ということを意識することなく、自由に使える）

また、GitHub にプッシュするだけで自動でビルド・デプロイをしてくれるのも嬉しい点です。
ドメインも都度作成・更新してくれます。

ただ、そもぞも vercel の前身は `ZEIT` 社の `Now` というサービスでした。
そのため、インターネットの記事が混在していたり、CLI も `now-cli` ->`vercel-cli`になっています。
このあたりが分かりづらく、かつ　Python に関するケースが少ないです。
Next.js, Nuxt.js は公式のテンプレートもあってかなり分かりやすいのかな、という印象です。

https://github.com/Ganariya/zenn-badge

`zenn-badge`を参考に、vercel のデプロイについてまとめようと思います。

## vercelにデプロイする前

vercel にデプロイする前は、開発を GitHub 上で行います。
というのも、以下の画像のようにリポジトリを vercel 上で指定するだけで自動デプロイが設定されます。

![](https://storage.googleapis.com/zenn-user-upload/7horxa0lln8r85u4j2anxl6qjvr4)

そのため、それまでは普通に GitHub で開発できます。
ただし、`vercel`で FastAPI が動くように意識して開発する必要があります。

## vercelでPython/FastAPIを動作させる

vercel で運用したいサービスを設定するために、`now.json`もしくは `vercel.json` を作成します。
おそらく Now 時代の名残で `now.json` も使用できますが、公式ドキュメントで扱っているのは[vercel.json](https://vercel.com/docs/configuration#introduction)です。
いくつか書き方も異なるようです。

今回は参考にしたリポジトリをもとに、`now.json`を設定します。

```json
{
  "version": 2,
  "public": false,
  "builds": [
    {
      "src": "zenn_badge/app.py",
      "use": "@now/python"
    }
  ],
  "routes": [
    {
      "headers": {
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
        "Access-Control-Allow-Headers": "X-Requested-With, Content-Type, Accept"
      },
      "src": "/.*",
      "dest": "zenn_badge/app.py"
    }
  ]
}
```

`public`ではログなどを公開状態にするかを指定できます。
今回はオフにしています。

`builds`では、ビルドするソースコードの指定と、`use`でビルド時に利用するモジュールを指定します。
今回は PythonOnly なので `@now/python`（正式には、現在は `@vercel/python`）を利用しています。
[どうやらbuildsに関するドキュメント](https://vercel.com/docs/configuration#project/builds)を参考にすると、現在は推奨されておらず、かわりに[functions](https://vercel.com/docs/configuration#project/builds)を指定すべきらしいですね。（しかし functions では `/api` ディレクトリにコードを置く必要があり、今回のリポジトリでは違う設計になっているため、かわりに builds を用います。）

`routes`はやってきたクエリのルーティングを行います。
今回はすべての通信をすべて `zenn_badge/app.py` に投げています。
これは、FastAPI のルーティングを利用しているため、エントリポイントの app.py にすべてを任せているためです。
この[routes](https://vercel.com/docs/configuration?query=routes#project/routes)も現在は推奨されていないようです。

このように `now.json` を設定することで `zenn_badge/app.py` をビルドし、すべての通信を `zenn_badge/app.py` に投げています。

## ライブラリのインストール

`builds`で `@now/python` を指定しているため、vercel で GitHub のプロジェクトをインポートすると自動でライブラリもインストールされます。
このとき、リポジトリのトップディレクトリに `requirements.txt` もしくは `Pipfile.lock` を置いてください。
これらのどちらかを置いておけば自動でインストールが実行されます。

## Pythonのバージョン

注意点として、Python のバージョンは `3.6` のようです。
そのため、3.8 から導入された `Final` などは使えないことに注意してください。

## FastAPI

これまでの now.json の設定で、`zenn_badge/app.py`に Flask や FastAPI を書けば動作するようになりました。
あとはコードを書いていくだけです。

今回のエントリポイントである `zenn_badge/app.py` のソースコードの出だしは以下のようになっています。

```python
import pybadges

from fastapi import FastAPI
from fastapi.responses import HTMLResponse

from .zenn import scrape_user
from .user import User
from .logo import LOGO

app = FastAPI()
BASE_COLOR: str = '#3FA8FF'


def make_badge(username: str, left_text: str, right_text: str) -> str:
    url = f'https://zenn.dev/{username}'
    return pybadges.badge(left_text=left_text, right_text=right_text, right_color=BASE_COLOR, embed_logo=True, logo=LOGO, left_link=url, right_link=url)


@app.get("/")
def main() -> str:
    return "Zenn Badges"
```

もともとは絶対ディレクトリ指定で `from zenn_badge.user import User` のようにインポートしていたのですがデプロイしたところエラーを吐きました。
どうやらマウントされたトップモジュール名が `__vercel__dir__` のような感じになっていました。
これによって、絶対指定をしようとすると、そもそもパスが開発環境と異なっていました。

そのため、今回は相対ディレクトリを利用することにしました。
このあたりの仕様について、もっとしらべていきたいですね。

# 課題

以下のような課題に引き続き取り組みたいと思っています。

- 画像やデータのキャッシュ
  - DB を外部に用意するなど
- `/`パスではバッジのテキストの自動作成を行える React を提供する
- `now.json`から `vercel.json` に以降し、推奨設定に変更
- liked, followers など全体を踏まえた　きれいな SVG の作成（GitHub-trophy みたいな）
- チーム開発（プルリク・イシューなどもらえると嬉しいです）

まだまだサーバレスやバックエンドが苦手なので、もっと色々作る中で勉強して、そして改善していきたいです。
vercel と netlify, AWS に早いところ慣れたいです。
