+++
title = "hugo でブログ書くことにしました"
description = ""
date = "2017-09-09T18:02:36+09:00"
categories = ["Programming"]
tags = ["hugo", "雑記"]
archives = ["2017-09"]
url = "2017-09/02/hugo-first-post"
thumbnail = "https://raw.githubusercontent.com/gohugoio/hugoDocs/master/static/img/hugo-logo.png"
+++

今まで放置していたブログですが、リニューアルして久しぶりに復活させることにしました。

<!--more-->

以前は [misaki](https://github.com/liquidz/misaki) を使っていたのですが、いろいろ
調べてみたところ [hugo](https://gohugo.io/)が流行っていたので試してみたところ自分に
とって使いやすそうだったので、こちらに移行することにしました。

### 環境メモ

現時点(2017/09/09)の自分の環境は以下の通りです(`hugo env`で表示)。

    bash$ hugo env
    Hugo Static Site Generator v0.26 darwin/amd64 BuildDate: 2017-08-07T20:34:15+09:00
    GOOS="darwin"
    GOARCH="amd64"
    GOVERSION="go1.8.3"
    bash$ 

### 古いコンテンツの保存

以前に `misaki` で作っていた数少ないコンテンツは、どれも中途半端な気がして移行する気には
なれず、さりとて完全に消去するにはもったいない、ということで、static なファイルに generate
して URL のパスに `/misaki` をつけてアクセスできるようにして保存しておくことにしました。

このページの右下の **Older Contents** にあるリンクからたどれます。

#### テーマの選択

`hugo` にはたくさんの[テーマ](https://themes.gohugo.io/)があり、選ぶのに迷いましたが、
ブログに適したテーマのうち [Mainroad](https://themes.gohugo.io/mainroad/) が気に入ったので
これに決定。

ただし、こまかなところはいろいろカスタマイズしているので、それについては別途記事を書こうかと
思います。

まずはリニューアル第１弾ということでこの辺で。
