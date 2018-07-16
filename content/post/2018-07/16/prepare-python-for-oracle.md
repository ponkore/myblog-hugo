+++
shortname = "prepare-python-for-oracle"
title = "Python から Oracle を使う際の環境設定メモ"
description = "Python から Oracle を使う際の環境設定メモ"
date = "2018-07-16T14:48:53+09:00"
categories = ["Programming"]
tags = ["Python", "Oracle"]
archives = ["2018-07"]
url = "post/2018-07/16/prepare-python-for-oracle"
thumbnail = "/img/2018-07/16/prepare-python-for-oracle.png"
+++

Python から Oracle を使う際の環境設定メモです。

<!--more-->

## OS/Python 環境

|        | 環境１         | 環境２         |
|--------|:--------------|:--------------|
| os     | windows 7     | macOS Sierra  |
| python | python 3.6.1  | python 3.6.1  |

## 作業手順

### virtual env に環境を仕込む

Jupyter Notebook で遊ぶつもりだったので、`jnb`という名前で venv を仕込
みます。以下は Windows の例ですが、macでも同様です。

```bat
C:\Apps\> python -m venv jnb
C:\Apps\> cd jnb
C:\Apps\jnb> Scripts\activate
(jnb) C:\Apps\jnb> pip list
```

### cx_Oracle のインストール(Windows)

```bat
(jnb) C:\Apps\jnb> pip install cx_Oracle --pre
```

`--pre` オプションをつけることで、コンパイル済DLLも一緒にインストール
してくれるようです。

### cx_Oracle のインストール(mac)

mac の場合は、まず環境変数の設定が必要です。

```shell-session
(jnb) bash$ export ORACLE_HOME=<path-to-oracle-home>
(jnb) bash$ export LD_LIBRARY_PATH=$ORACLE_HOME
```

さらに、pip install する際のコンパイル時に、参照する共有ライブラリのバー
ジョン名をごまかすため、シンボリックリンクを作成します。

```shell-session
(jnb) bash$ pushd $ORACLE_HOME
(jnb) bash$ ln -s libclntsh.dylib.12.1 libclntsh.dylib
(jnb) bash$ ln -s libocci.dylib.12.1 libocci.dylib
(jnb) bash$ popd
```

これだけできれば、あとは普通に `pip install` でいけます。

```shell-session
(jnb) bash$ pip install cx_Oracle
```

## Oracleへの事前接続確認(SQL*Plus)

検証のため、SQL*Plus でちゃんと接続できるか、確認しておきましょう。

### tnsnames.ora を使わない場合

```
(jbb) bash$ sqlplus scott/tiger@//dbserver:port:sid
```

### tnsnames.ora を使う場合

```
(jbb) bash$ sqlplus scott/tiger@DB
```

## Oracleへの事前接続確認(python)

あとは、Python から接続できるか、を確認します。具体的には connect を呼
び出してみます。

```shell-session
(jbb) bash$ python
Python 3.6.1 (default, Apr  4 2017, 09:40:21) 
[GCC 4.2.1 Compatible Apple LLVM 8.1.0 (clang-802.0.38)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import cx_Oracle as cx
>>> conn = cx.connect('SCOTT','TIGER','XE')
>>> ^D
(jnb) bash$
```

## 終わりに

もう一年以上前に作成したメモで、ずっと放置していたのですが、記事に書き
起こしてみました。最近は仕事で Oracle を触らなくなってしまったので、も
うあまり役にはたたないのですが、せっかくなので、ということで。まあ今後
のバージョンで手順に変化があるかもしれませんが、その時はその時で。
