---
layout:	post
title:	google-translate.elのtkkエラー
author:	takushi1969
tags:	emacs google-translate
---
emacsからgoogle-translateを使用すると、ミニバッファに**Search failed: ",tkk:'"**となり、動作しない。

<img src="/assets/2021/01/24/google-translate-error.png" alt="google-translate error" style="width:480px;">

## 原因 ##

動作しない原因は、[google translateのAPI](https://github.com/Boudewijn26/gTTS-token/blob/master/docs/november-2020-translate-changes.md)の変更にあるらしい。

## 暫定対処 ##

google-tanslateの内部関数:`google-translate--get-b-d1`を上書いて、[公知の**TKK**を返す方法](https://qiita.com/minoruGH/items/75eb4fab53e93653f999)がある。


## 正式対応に関する調査 ##

未検証だが、[google-translate-server](https://github.com/guyrotem/google-translate-server)で公開されているソースを利用して、proxyサーバを起てる方法があるらしい。
