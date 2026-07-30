---
title: Nim製ウェブフレームワーク一挙大解説！
source: https://qiita.com/dumblepy/items/9417626f33de19fe14ba
author:
  - "[[dumblepy]]"
published: 2020-12-14
created: 2026-07-09
description: 昨年やっとV1.0がリリースされたNim言語ですが、今年はv1.4.2まで開発が進みました。 Nim自体も徐々に人気を集めてきて、それに伴いウェブフレームワークも沢山作られるようになりました。 今回は群雄割拠のNim製のウェブフレームワークについて網羅的に解説していきます！...
tags:
  - clippings
  - nim
---
この記事は最終更新日から5年以上が経過しています。

昨年やっとV1.0がリリースされたNim言語ですが、今年はv1.4.2まで開発が進みました。  
Nim自体も徐々に人気を集めてきて、それに伴いウェブフレームワークも沢山作られるようになりました。  
今回は群雄割拠のNim製のウェブフレームワークについて網羅的に解説していきます！

比較するために、以下の全て同じシナリオを実装したいと思います。

| Httpメソッド                                          | URI   | リクエスト            | 期待するレスポンス                 |
| ------------------------------------------------- | ----- | ---------------- | ------------------------- |
| GET                                               | /     |                  | `<h1>Hello World</h1>`    |
| POST                                              | /     | {"name": "John"} | {"message": "Hello John"} |
| GET                                               | /{id} |                  | `{id}`                    |
| サーバーは8000番ポートで立ち上げます。                             |       |                  |                           |
| POSTでアクセスされた時、JSONリクエストから値を取り出すのに失敗した時には400を返します。 |       |                  |                           |
| HttpメソッドやURIが一致しない場合には404を返します。                   |       |                  |                           |
|                                                   |       |                  |                           |

## asynchttpserver

<iframe src="https://qiita.com/embed-contents/link-card#qiita-embed-content__07289a1ad1bec88e5143da10804e9ae9" frameborder="0" height="80"></iframe>

asynchttpserverは非同期通信するアプリケーションサーバーを作るためのライブラリで、Nim標準ライブラリに入っています。

```nim
import asynchttpserver, asyncdispatch, re, strutils, json

proc main {.async.} =
  var server = newAsyncHttpServer()
  proc cb(req: Request) {.async.} =
    if req.reqMethod == HttpGet and req.url.path == "/":
      let headers = {"Content-type": "text/html; charset=utf-8"}
      await req.respond(Http200, "<h1>Hello World</h1>", headers.newHttpHeaders())

    elif req.reqMethod == HttpPost and req.url.path == "/":
      let headers = {"Content-type": "application/json; charset=utf-8"}
      try:
        let requestBody = req.body.parseJson
        let name = requestBody["name"].getStr
        let response = %*{"message": "Hello " & name}
        await req.respond(Http200, $response, headers.newHttpHeaders())
      except Exception:
        let response = %*{"message": "エラーが発生しました"}
        await req.respond(Http400, $response, headers.newHttpHeaders())

    elif req.reqMethod == HttpGet and req.url.path.match(re"\/\d"):
      let headers = {"Content-type": "text/html; charset=utf-8"}
      let id = req.url.path.split("/")[1]
      await req.respond(Http200, id, headers.newHttpHeaders())

    else:
      await req.respond(Http404, "Not found")

  waitFor server.serve(Port(8000), cb)

discard main()
```

`Request` 型を引数に取るコールバック関数を作り、それを `server.serve()` の第二引数に入れてあげることでサーバーが起動します。  
`Request` 型に様々なメソッドが用意されており、そのメソッドを使うことでURLパスやHttpメソッドやリクエストボディを取り出せるようになっています。  
標準ライブラリなのでリッチなルーティングも用意されておらず、自前で実装しなければなりませんが、必要最小限の機能はあるかなと思います。

## httpbeast

<iframe src="https://qiita.com/embed-contents/link-card#qiita-embed-content__ed69717b496f0f7a49791ab372703734" frameborder="0" height="112"></iframe>

httpbeastはNimのコミッターとして活発に活動されているdom96氏が作ったライブラリです。マルチスレッドで高速に動作するアプリケーションサーバーを作ることができます。  
後述するjesterはこのhttpbeastをベースにして作られています。

```nim
import options, asyncdispatch, json, re, strutils
import httpbeast

proc onRequest(req: Request):Future[void] =
  if req.httpMethod == some(HttpGet) and req.path.get() == "/":
    let headers = "Content-type: text/html; charset=utf-8"
    req.send("<h1>Hello World</h1>")

  elif req.httpMethod == some(HttpPost) and req.path.get() == "/":
    let headers = "Content-type: application/json; charset=utf-8"
    try:
      let requestBody = req.body.get.parseJson
      let name = requestBody["name"].getStr
      let response = %*{"message": "Hello " & name}
      req.send(Http200, $response, headers)
    except Exception:
      let response = %*{"message": "エラーが発生しました"}
      req.send(Http400, $response, headers)

  elif req.httpMethod == some(HttpGet) and req.path.get.match(re"\/\d"):
    let headers = "Content-type: text/html; charset=utf-8"
    let id = req.path.get.split("/")[1]
    req.send(Http200, id, headers)

  else:
    req.send(Http404)

let settings = initSettings(Port(8000))
run(onRequest, settings)
```

何もオプションを付けずに動かすと1 threadで動きますが

```sh
nim c -r http_beast.nim
>Starting 1 threads
```

コンパイルオプションに `--threads:on` を付けることで実行するPCのコア数分マルチスレッドで動いてくれます。

```sh
nim c -r --threads:on http_beast.nim
>Starting 4 threads
>Listening on port 8000
```

## jester

<iframe src="https://qiita.com/embed-contents/link-card#qiita-embed-content__259217cceab1326e5d63449a1cf6f901" frameborder="0" height="112"></iframe>

jesterはhttpbeastをベースにして、さらにプロダクションユースでより使いやすくしたマイクロウェブフレームワークです。Nim開発者のためのコミュニティである [Nim Forum](https://forum.nim-lang.org/) はこのJesterで動いています。  
Nimのウェブフレームワークとしては最も選択肢に上がるかなと思いますが、非同期処理には対応していないところが個人的には玉に瑕かなという印象があります。  
しかしレスポンスの速さはGoもRustも凌駕しており、一目置くものがあります。

[![スクリーンショット 2020-12-13 16.52.07.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295826/d57888b4-6a61-b9a5-072c-57c485eaf217.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F295826%2Fd57888b4-6a61-b9a5-072c-57c485eaf217.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=8066cf13dd349a61d7a740e8fa44d3ed)

[![スクリーンショット 2020-12-13 16.52.30.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/295826/a5b9585f-4997-97f4-4174-e4137516a5a2.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F295826%2Fa5b9585f-4997-97f4-4174-e4137516a5a2.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=664c8446769b1213784585a6a8d5a392)

```nim
import jester, json

router route:
  get "/":
    resp "<h1>Hello World</h1>"

  post "/":
    try:
      let params = request.body.parseJson
      let name = params["name"].getStr
      let response = %*{"message": "Hello " & name}
      resp response
    except Exception:
      let headers = [("Content-type", "application/json; charset=utf-8")]
      let response = %*{"message": "エラーが発生しました"}
      resp Http400, headers, $response

  get "/@id":
    resp @"id"

proc main() =
  let settings = newSettings(port=Port(8000))
  var jester = initJester(route, settings=settings)
  jester.serve()

main()
```

jesterでは `request` という変数がルーティングのスコープの中で使えるようになります。ここからリクエストが持つ様々な値を取り出すことができます。

## prologue

[https://github.com/planety/prologue](https://github.com/planety/prologue)  
[https://planety.github.io/prologue/](https://planety.github.io/prologue/)

prologueは非同期処理に対応したフルスタックウェブフレームワークです。jesterの次に人気のあるウェブフレームワークのようです。公式ドキュメントが綺麗ですね！  
ドキュメントでは言及されていませんが、私にはDjangoへの設計の近さを感じました。  
テンプレートエンジンには [Karax](https://github.com/pragmagic/karax) を使うことが推奨されているようです  
[NimのSPAフレームワーク "Karax" のご紹介](https://qiita.com/momeemt/items/599c9959a66e7440690e)

```nim
import prologue
import ./urls

let settings = newSettings(port=Port(8000))
var app = newApp(settings = settings)
app.addRoute(urls.urlPatterns, "")
app.run()
```

```nim
import prologue

import ./views

const urlPatterns* = @[
  pattern("/", index, HttpGet),
  pattern("/", store, HttpPost),
  pattern("/{id}", show)
]
```

```nim
import json
import prologue
import ./templates.index_view

proc index*(ctx: Context) {.async.} =
  resp index_view.render()

proc store*(ctx: Context) {.async.} =
  try:
    let params = ctx.request.body.parseJson
    let name = params["name"].getStr
    let response = %*{"message": "Hello " & name}
    resp jsonResponse(response)
  except Exception:
    let response = %*{"message": "エラーが発生しました"}
    resp jsonResponse(response, Http400)

proc show*(ctx: Context) {.async.} =
  let id = ctx.getPathParams("id")
  resp id
```

```nim
import karax / [karaxdsl, vdom]

proc render*(): string =
  let vnode = buildHtml(tdiv(class = "mt-3")):
    h1: text("Hello World")
  result = $vnode
```

## basolato

<iframe src="https://qiita.com/embed-contents/link-card#qiita-embed-content__f65397b2c3eb3676b4ae309576cae03f" frameborder="0" height="29"></iframe>

basolatoはasynchttpserverをベースにしたMVCのフルスタックウェブフレームワークです。PHPのLaravel4と同じような使い勝手を目指しつつ、DDDを取り入れた開発にレールを敷いて、フレームワークがビジネスロジックに介在しないように設計されています。  
ActiveRecordはありませんが、代わりにallographerというクエリビルダがデフォルトで使えるようになっています。  
[LaravelそっくりなNim製クエリビルダallographerを作った話](https://qiita.com/dumblepy/items/17dd2ba99741a17128d8)

`ducere` というCLIツールも機能が豊富で、最適化されたビルドやコントローラーやドメインモデル、ビューの作成がコマンド一発で行えます。

ビューはいわるゆるMPAのテンプレートエンジンですが、NuxtJSの設計を参考にCssInJsに影響を受けた機能もあり、コンポーネント= **htmlとCSSの塊を返す関数** で作ることができます。  
このため初期の開発ではMPAで作り、プロダクトがグロースした後はビューのコンポーネントをそのままReact/Vueに置き換え、更にコントローラーとビジネスロジックが分離しているため、コントローラーのわずかな変更でビジネスロジックに手を加えることなくAPIサーバーに置き換えることができます。

```sh
# プロジェクトを作成
ducere new basolato_app
cd basolato_app

# 　コントローラーを作成
ducere make controller hoge
>Created controller hoge_controller.nim

# ビューを作成
ducere make page hello_world
>created page view /basolato_app/resources/pages/hello_world_view.nim

# アプリケーションを起動
ducere serve -p 8000
>Starting 1 thread
>Listening on port 8000
```

```nim
# framework
import basolato
# controller
import app/controllers/hoge_controller

var routes = newRoutes()
routes.get("/", hoge_controller.index)
routes.post("/", hoge_controller.store)
routes.get("/{id:int}", hoge_controller.show)

serve(routes)
```

```nim
import json
# framework
import basolato/controller
# view
import ../../resources/pages/hello_world_view

proc index*(request:Request, params:Params):Future[Response] {.async.} =
  return render(helloWorldView())

proc store*(request:Request, params:Params):Future[Response] {.async.} =
  try:
    let name = params.getJson["name"].getStr
    let response = %*{"message": "Hello " & name}
    return render(response)
  except Exception:
    let response = %*{"message": "エラーが発生しました"}
    return render(Http400, response)

proc show*(request:Request, params:Params):Future[Response] {.async.} =
  let id = params.getInt("id")
  return render($id)
```

```nim
import basolato/view
import ../layouts/application_view

proc impl():string = tmpli html"""
<h1>Hello World</h1>
"""

proc helloWorldView*():string =
  let title = ""
  return applicationView(title, impl())
```

## phoon

<iframe src="https://qiita.com/embed-contents/link-card#qiita-embed-content__0d2102901b547ccc3754e23e02ecefbd" frameborder="0" height="29"></iframe>

phoonはExpressJSに影響を受けたマイクロフレームワークです。  
私の手元ではフレームワーク内部のバグにより、動かすことができませんでした。開発も停滞しているので今後に期待です。

```sh
/root/.nimble/pkgs/phoon-0.1.0/phoon.nim(13, 32) Error: ambiguous identifier: 'Callback' -- use one of the following:
  asyncdispatch.Callback: Callback
  route.Callback: Callback
```

## akane

<iframe src="https://qiita.com/embed-contents/link-card#qiita-embed-content__f9ba6bfa09779708281bc227ac0b34e2" frameborder="0" height="29"></iframe>

akaneは内部的に非同期処理を実装したマイクロフレームワークのようです。  
まだまだドキュメントの不足や、開発の滞りが見られるので今後に期待です。

```nim
import akane, json, re

proc main =
  var server = newServer("0.0.0.0", 8000)

  server.pages:
    equals("/", HttpGet):
      await request.answer("<h1>Hello World</h1>")

    equals("/", HttpPost):
      try:
        let params = request.body.parseJson
        let name = params["name"].getStr
        let response = %*{"message": "Hello " & name}
        await request.sendJson(response)
      except Exception:
        let response = %*{"message": "エラーが発生しました"}
        await request.sendJson(response, Http400)

    regex(re"\/\d", HttpGet):
      let id = decoded_url.split("/")[1]
      await request.answer(id)

  server.start()
main()
```

## rosencrantz

<iframe src="https://qiita.com/embed-contents/link-card#qiita-embed-content__ba9c200bbebcb8ca4303de26562ef517" frameborder="0" height="29"></iframe>

rosencrantzはasynchttpserverをベースにしてScalaのAkkaというフレームワークを参考にして作られたウェブ開発用のDSLです。

DSLが難解すぎて私には実装できませんでした…

## whip

<iframe src="https://qiita.com/embed-contents/link-card#qiita-embed-content__b8503c672249c059d3dc12086a006c78" frameborder="0" height="29"></iframe>

whipはhttpbeastをベースにしたマイクロフレームワークです。めちゃくちゃ速いようです。  
`JsonNode` 型が標準ライブラリにあるものではなく、 `packedjson` というライブラリに入ってるものを使わなければいけないようで、そこは注意が必要です。

```nim
import whip, sugar, packedjson

proc store(params:JsonNode):JsonNode =
  try:
    let name = params["name"].getStr
    let response = %*{"message": "Hello " & name}
    return response
  except Exception:
    let response = %*{"message": "エラーが発生しました"}
    return response

let w = initWhip()
w.onGet "/", (r:Wreq) => r.html("<h1>Hello World</h1>")
w.onPost "/", (r:Wreq) => r.json(store(r.body))
w.onGet "/{id}", (r:Wreq) => r.html(r.path("id"))

w.start(8000)
```

## servy

[https://github.com/xmonader/nim-servy](https://github.com/xmonader/nim-servy)  
[https://xmonader.github.io/nim-servy/](https://xmonader.github.io/nim-servy/)

servyは軽量なマイクロフレームワークとのことです。  
こちらはnimbleでインストールしてもservy.nimのファイルがなく、ビルド済みバイナリがライブラリの中にできてしまい、フレームワークを使うことができませんでした。

## geminim

<iframe src="https://qiita.com/embed-contents/link-card#qiita-embed-content__5ef2688caf26146387ac829a74e9c65f" frameborder="0" height="29"></iframe>

geminimはNimによる `gemini protocol` の実装です。  
`gemini protocol` というのは、これまでよりもより低電力消費で、低速ネットワークでも稼働し、プライバシーを非常に重視した新しいインターネットプロトコルとのことです。  
非常に低レイヤーでの実装であり、私の力量ではサンプルを実装することができませんでした。

## まとめ

いかがでしたでしょうか。  
ウェブフレームワークは玉石混交ではありますが、2020年になってNimでもようやくプロダクションユースで使えるウェブフレームワークが出てきたと思います。  
私が開発しているbasolatoも今後も頑張って開発していくので、ぜひ2021年からのウェブ開発ではNimを使ってみることを検討してもらいたいと思います！

[0](#comments)

コメント一覧へ移動

X（Twitter）でシェアする

Facebookでシェアする

はてなブックマークに追加する