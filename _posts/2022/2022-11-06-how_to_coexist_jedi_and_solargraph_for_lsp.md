---
layout: post
title: How to make solargraph avaiable with jedi for emacs lsp
tags: emacs ruby lsp
---
# 課題 #

Emacsに[lsp-jedi](https://github.com/fredcamps/lsp-jedi)をインストールして、公式ページ通りに設定した後、他のLSPを使用すると以下のようなメッセージが出力されてLSPが使用できない。

	LSP :: Client ruby-ls is not in lsp-enabled-clients
	LSP :: No LSP server for ruby-mode(check *lsp-log*).

# 原因 #

[lsp-jedi](https://github.com/fredcamps/lsp-jedi)の[公式ページの設定](https://github.com/fredcamps/lsp-jedi#configure)の設定で指定している以下のS式によって、nilだったlsp-enabled-clients変数に'jediのみを設定してしまう。この結果、lsp-enabled-clients変数を有効化した副作用として、jedi以外のlspクライアントは動作しなくなる。

{% highlight common_lisp %}
(add-to-list 'lsp-enabled-clients 'jedi)
{% endhighlight %}	


# 回避方法 #

[公式ページの設定](https://github.com/fredcamps/lsp-jedi#configure)を以下のように変更する。
{% highlight common_lisp %}
(use-package lsp-jedi
  :ensure t
  :config
  (with-eval-after-load "lsp-mode"
    (add-to-list 'lsp-disabled-clients 'pyls)))
{% endhighlight %}	

