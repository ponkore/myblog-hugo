+++
title = "[小ネタ] Hugoの記事を書き始めるための emacs lisp"
description = ""
date = "2017-10-02T22:09:55+09:00"
categories = ["Programming"]
tags = ["Emacs", "lisp", "hugo", "小ネタ"]
archives = ["2017-10"]
url = "post/2017-10/02/add-emacs-function-for-hugo-post"
thumbnail = "/img/2017-10/02/Emacs.png"
+++

Hugoで記事を書き始めるための emacs lisp を書いてみました。

<!--more-->

### Motivation

Hugo でブログを書こうと決めたのですが、記事の元となる markdown file を
置くパスや、記事の URL、それに合わせて記事の front matter を設定したり、
と細々したことを毎回思い出すのが面倒に思いました。

どうせエディタは Emacs を使っているので、適当な emacs lisp で省力化を
図ろうという魂胆です。いろんな人が使うことを想定すると、結構複雑に
なってしまうのですが、そこは自分専用と割り切って極力単純な
仕組みになるよう考えました。

### Code

**記事を置く markdown file のパス**

年月日は自動で今日の日付が入る仕様です。

```lisp
(defvar myblog-hugo-base-directory-format-string "~/blog/myblog-hugo/content/post/%Y-%m/%d/")
```

**記事の初期状態のテンプレート**

front matter と、 `<!--more-->` までを初期状態として作成します。

```lisp
(defvar myblog-hugo-front-matter-template "+++
title = \"\"
description = \"\"
date = \"%Y-%m-%dT%H:%M:%S+09:00\"
categories = [\"Programming\"]
tags = [\"\"]
archives = [\"%Y-%m\"]
url = \"post/%Y-%m/%d/{{post-title}}\"
thumbnail = \"/img/%Y-%m/%d/{{post-title}}.png\"
+++

<!--more-->
")
```

**関数本体( myblog-hugo-create-post が本体)**


```lisp
(defun myblog-hugo-create-frontmatter (post-title)
  (let* ((tmp (format-time-string myblog-hugo-front-matter-template (current-time))))
    (replace-regexp-in-string "{{post-title}}" post-title tmp)))

(defun myblog-hugo-create-post (post-title)
  "create a hugo post file with default template."
  (interactive "Mblog-title-string(en): ")
  (let* ((post-filename-prefix (format-time-string myblog-hugo-base-directory-format-string (current-time)))
         (tmp-post-title (downcase (replace-regexp-in-string "[ \t\./]+" "-" post-title)))
         (filename (concat post-filename-prefix tmp-post-title ".md"))
         (directory (file-name-directory filename)))
    (if (y-or-n-p (concat "create a post " filename ". ok? "))
        (progn
          (unless (file-exists-p directory)
            (make-directory directory t))
          (let* ((front-matter (myblog-hugo-create-frontmatter tmp-post-title))
                 (buf (set-buffer (find-file-noselect filename t))))
            (with-current-buffer buf
              (goto-char (point-min))
              (insert front-matter)
              (basic-save-buffer)
              (switch-to-buffer buf)
              (goto-char (point-max)))))
      (message "canceled."))))
```

### Usage

* `M-x myblog-hugo-create-post` で呼び出します。

![screenshot](/img/2017-10/02/sc-01.png)

* ミニバッファに `blog-title-string(en): ` と表示されるので、簡潔なタイトル（英文字のみ）を入力します。
  ファイル名として使用するため英文字のみ(といいつつ特に処理的に禁止はしていませんが...)。
  空白は `-` に置き換えますので使用可能です。

![screenshot](/img/2017-10/02/sc-02.png)


* `create a post ....  ok? (y or n) ` と表示されるので、`y` を入力すると、

![screenshot](/img/2017-10/02/sc-03.png)

このような内容のファイルが初期状態として作成されます。

結局のところ、

* 日付ベースのパス名を自動で作成
* 上記パスを自動で作成
* ファイル名を編集（スペースをハイフンに置き換え）

位しかやっていませんが、記事を作成する「心理的障壁」は少し下げられました
（個人的感想）。

### おわりに

今のところ、自分の init.el から呼び出す小物の elisp に混ぜ込んでいる状態
ですが、そのうち気が向いたら（自分専用）パッケージ化するかも。
