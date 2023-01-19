---
layout: post
title: If you miss crypt of openssl-passwd
tags: docker openssl
---
日頃、[Ubuntu](https://jp.ubuntu.com/)をよく使うが、
[Jammy](https://releases.ubuntu.com/22.04/)から
[OpenSSL](https://www.openssl.org/)がバージョンアップされて、
[passwd(1ssl)](https://www.openssl.org/docs/man3.1/man1/openssl-passwd.html)
のcryptが非サポートとなった。ちょっとしたパスワードの作成時に重宝して
いたので不便だ。

そこで、[Jammy](https://releases.ubuntu.com/22.04/)から
[OpenSSL-passwd(1ssl)](https://www.openssl.org/docs/man3.1/man1/openssl-passwd.html)
cryptを使用する代替を考慮してみた。どうやら、
[Ubuntu](https://jp.ubuntu.com/)公式の
[Focal](https://releases.ubuntu.com/focal/)の
[docker](https://www.docker.com/)を使用するのが軽くて良さそう。

以下に、[docker](https://www.docker.com/)で
[OpenSSL-passwd(1ssl)](https://www.openssl.org/docs/man3.1/man1/openssl-passwd.html)
cryptを使用するまでの手順を示す。

1. [docker](https://www.docker.com/)のインストール

        $ sudo apt update
        $ sudo apt install -y docker.io

2. [docker](https://www.docker.com/)権限の付与

        $ sudo adduser $(whoami) docker

    付与後、docker.socketに権限変更を認識させるため、システムの再起動を行う。  
		余談だが、[Ubuntu](https://jp.ubuntu.com/)にインストールしたパッケージをカスタマイズをする時は、google前に、まづは`/usr/share/doc/パッケージ名`配下のドキュメントの参照をお勧めする。
    
3. [docker](https://www.docker.com/)イメージの作成

    1. `Dockerfile`の作成
    
        以下の内容を持つ`Dockerfile`を作成。
        
            FROM ubuntu:focal
            RUN apt update
            RUN apt install -y openssl

    2. [docker](https://www.docker.com/)イメージの作成
    
        上記で作成したDockerfileがあるディレクトリをカレント・ディレクトリとして、以下のコマンドを実行する。
        
            $ docker build -t focal-openssl .

		実行の完了までには暫く時間がかる。  
		完了すると、以下のコマンドで[docker](https://www.docker.com/)イメージが作成できたことが確認できる。
		
            $ docker image ls focal-openssl 
            REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
            focal-openssl   latest    e6c3309acbc7   51 minutes ago   118MB
            $ 
			
4. cryptの実行

	以下にcryptのワンショットの実行例を示す。
	
		$ docker run  --rm focal-openssl openssl passwd -salt hoge hoge
		hojc.7BUvCifs
		$ 
