+++
title = "Hugoの記事を書き始めるための emacs lisp を書いた"
description = ""
date = "2017-10-02T22:09:55+09:00"
categories = ["Programming"]
tags = ["Emacs", "lisp", "hugo"]
archives = ["2017-10"]
url = "post/2017-10/02/add-emacs-function-for-hugo-post"
thumbnail = "/img/2017-10/02/Emacs.png"
+++

Hugoで記事を書き始めるための emacs lisp を書いてみました。

<!--more-->

### Motivation

Hugo でブログを書こうと決めたのですが、記事の元となる markdown file を
置くパスや、記事の URL、それに合わせて記事の front matter を設定したり、
と、

```lisp
(defvar myblog-hugo-base-directory-format-string "~/blog/myblog-hugo/content/post/%Y-%m/%d/")
```

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
