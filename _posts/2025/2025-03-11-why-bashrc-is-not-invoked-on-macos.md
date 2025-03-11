---
layout: post
title: Why .bashrc is not called on macOS?
tags: osx shell
---
## はじめに

macOSにcanda-forgeをインストールして、.bashrcにconda initiliazeのブロッ
クが追加されたことを確認した後、Terminalを起動してもcondaの初期化が走
らない。その顛末をまとめた。

### .bashrcに定義したconda initilizeのブロック

	inaho:~ taku$ sed -ne '/>>> conda/,/<<< conda/ p' .bashrc
	# >>> conda initialize >>>
	# !! Contents within this block are managed by 'conda init' !!
	__conda_setup="$('/Users/taku/miniforge3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
	if [ $? -eq 0 ]; then
	    eval "$__conda_setup"
	else
	    if [ -f "/Users/taku/miniforge3/etc/profile.d/conda.sh" ]; then
	        . "/Users/taku/miniforge3/etc/profile.d/conda.sh"
	    else
	        export PATH="/Users/taku/miniforge3/bin:$PATH"
	    fi
	fi
	unset __conda_setup

	if [ -f "/Users/taku/miniforge3/etc/profile.d/mamba.sh" ]; then
	    . "/Users/taku/miniforge3/etc/profile.d/mamba.sh"
	fi
	# <<< conda initialize <<<

### 再起動したTerminalでのcondaの実行結果

![can't find conda](/assets/images/2025/0311/fail_to_invoke_conda.png)

## 原因

.bashrcの実行タイミングがlinuxと違うと感じていたが、どうやら、macOSの
graphicalログインとTerminalの仕様のよう。仕様の一次ソースを軽く探して
みたが、見つけることができなかった。ただ、動作確認の結果、腑に落ちた。

### 今回の調査で参考となったリンク

特に2つ目のリンクは、いまさらながらに.bashrcの存在意義を知るきっかけとなっ
た。

1. [Mac OS X .bashrc not working](https://superuser.com/a/244990)
2. [Why are interactive shells on OSX login shells by default?](https://unix.stackexchange.com/q/119627)

## 対処

### .bash_profileの変更

`.bash_profile`に以下のコードを追加。

    [[ -e ~/.bashrc  ]] && source ~/.bashrc 

### .bash_profile改版後の再起動したTerminalでのcondaの実行結果

![find conda](/assets/images/2025/0311/success_to_invoke_conda.png)

## 課題

macOSでのconda initilizeのブロックの実行に時間が掛かることがわかってい
たが、やはり不便極まりない。この課題については、後日対応方法を調査しよ
う。
