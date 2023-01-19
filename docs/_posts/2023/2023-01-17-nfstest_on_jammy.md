---
title: if you encounter 'parser ModuleNotFoundError' error when running NFStest on Jammy
tags: linux nfs
---

## 問題 ##

[Ubuntu 22.04](https://releases.ubuntu.com/22.04/)で[NFStest-3.0](https://linux-nfs.org/wiki/index.php/NFStest)を動作させようとすると以下のエラーメッセージで実行できず。  

```
vagrant@ubuntu-jammy:~/NFStest-3.0/test$ ./nfstest_posix -h
Traceback (most recent call last): File
"/home/vagrant/NFStest-3.0/test/./nfstest_posix", line 27, in <module> from nfstest.test_util import TestUtil File
"/home/vagrant/NFStest-3.0/nfstest/test_util.py", line 60, in <module> from nfstest.nfs_util import NFSUtil File
"/home/vagrant/NFStest-3.0/nfstest/nfs_util.py", line 34, in <module> from nfstest.host import Host File
"/home/vagrant/NFStest-3.0/nfstest/host.py", line 33, in <module> from packet.pktt import Pktt File
"/home/vagrant/NFStest-3.0/packet/pktt.py", line 49, in <module> import parser ModuleNotFoundError: No module named 'parser'
vagrant@ubuntu-jammy:~/NFStest-3.0/test$
```
原因は、Python 3.10以降で、parseモジュールが削除されたため(参考：[Whatever happened to package 'parser' in Python 3.10?](https://stackoverflow.com/questions/73328659/whatever-happened-to-package-parser-in-python-3-10))。

## 対処 ##

NFStest-3.0の実行時のインタプリタとして、python 3.9を使用する。  
具体的には、pyenvを使用して対処する。pyenvは、実行環境のpythonのバージョ
ンをコントロールすることができる。詳細は、
[pyenv](https://github.com/pyenv/pyenv)を参照されたい。  
以下に、Ubunt 22.04でのログインshellがbashであるユーザへのpython 3.9を導入する手順を示す。

### Ubuntu 22.04へのpyenvのインストール ###

[pyenv](https://github.com/pyenv/pyenv)のREADME.mdの記載内容をUbuntu 22.04向け
にサマライズする。

1. pyenvのインストール

		git clone https://github.com/pyenv/pyenv.git ~/.pyenv

2. shell環境の準備

	1. .bashrcの編集
	
			echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
			echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
			echo 'eval "$(pyenv init -)"' >> ~/.bashrc

	1. .profileの編集

			echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.profile
			echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.profile
			echo 'eval "$(pyenv init -)"' >> ~/.profile
			
	1. 実行環境への反映
	
			exec "$SHELL"

3. pythonのビルド環境の準備

		sudo apt update -y; sudo apt install -y build-essential libssl-dev zlib1g-dev \
		libbz2-dev libreadline-dev libsqlite3-dev curl llvm \
		libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev

### Ubuntu 22.04へのpython 3.9のインストールと環境設定 ###

1. python 3.9のインストール

	以下のコマンドで、3.9の最新版をインストール。

		pytenv insatll $(pyenv install -l | egrep "^\s+3\.9" | tail -1)

2. python 3.9の有効化

		pyenv global $(pyenv versions | sed -nEe 's/.*(3\.9\.[[:digit:]]+).*/\1/p' | sort -V | tail -1)
		
3. pythonのバージョン確認

		python --version

### Ubuntu 22.04でのNFStest-3.0の実行 ###

1. NFStest-3.0のダウンロードと展開

		wget http://www.linux-nfs.org/~mora/nfstest/releases/NFStest-3.0.tar.gz
		tar -xzvf NFStest-3.0.tar.gz

2. NFStest-3.0の実行
   
        cd NFStest-3.0
		export PYTHONPATH=$PWD
		cd test
		sudo mkdir -p /mnt/t
		./nfstest_posix -s *server* -e *share* 

