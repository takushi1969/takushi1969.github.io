---
layout: post
title:  recovery from cygwin's failure after make cygserver run at boot.
author: takushi1969
tags:   cygwin cygserver sshd postgresql apache
---

## はじめに ##

[Cygwin](https://cygwin.com/)はPosix機能以外に
[PostgreSQL](https://www.postgresql.org/)や
[Apache](https://httpd.apache.org/)などのLAMPを構成するサーバも提供し
ている。

これらのサーバをバックグラウンドで実行するには、サーバに代ってセ
キュリティ・アービトレーションを実施する
[Cygserver](https://cygwin.com/cygwin-ug-net/using-cygserver.html)の起
動が不可欠である。

## 問題 ##

通常、
[Cygserver](https://cygwin.com/cygwin-ug-net/using-cygserver.html)を
Windowsのブート時に自動起動するよう設定する。

[Cygserver](https://cygwin.com/cygwin-ug-net/using-cygserver.html)は、
セキュリティ・アービトレーション以外にも、[Caching account
information](https://cygwin.com/cygwin-ug-net/ntsec.html#ntsec-mapping-caching)
の以下の記載にあるようにアカウントのキャッシュを行う。

> If cygserver is running it will provide passwd and group entry caching for all processes in every Cygwin process tree started after cygserver. So, if you start a Cygwin Terminal and cygserver is running at the time, mintty, the shell, and all child processes will use cygserver caching.

Windowsがスタンドアローンの運用であれば、このアカウント・キャッシュが悪さをすることはない。

しかし、組織のActive DirectoryなどのDomainに属しているWindowsで[Cygserver](https://cygwin.com/cygwin-ug-net/using-cygserver.html)をブート起動した場合、Windowsのログインに使用するドメイン・ユーザのアカウントがCygserverの起動時にキャシュされない。このため、ログイン後、Cygwin Terminalを起動に失敗する。

## 対処 ##

以下の手順を踏むことで、この問題に対処することができる。
  1. Cygserverの停止
  2. /etc/passwd、/etc/groupを作成
  3. /etc/nsswitch.confの編集
  4. Cygserverの再開

以降にそれぞれの詳細を記す。

### 1. Cygserverの停止 ###

  1. 管理者権限でpowershellを起動
  2. stop-serviceコマンドレットでCygserverを停止

		> Stop-Service cygserver
  3. set-serviceコマンドレットでCygserverの自動起動を停止
  
		> Set-Service -StartupType Disabled cygserver
  4. Windowsの再起動

![stop cygserver](/assets/2021/02/20/stop-cygserver.gif)

### 2. /etc/passwd、/etc/groupを作成 ###

上記手順の結果、Windowsの再起動後にCygwin Terminalが起動可能となる。

Cygwin Terminalを起動して、以下のコマンドを実行して、passwdファイルと
groupファイルを作成する。

	$ mkpasswd -c -l > /etc/passwd
	$ mkgroup -c -l > /etc/group

![mkpasswd](/assets/2021/02/20/mkpasswd.gif)

### 3. /etc/nsswitch.confの編集 ###

Nave Service Switchがpasswdとgroupのアカウント情報を検索する優先順位に、
上記で作成したファイルを第一優先となるように設定する。

  * 変更前

		# passwd:   files db
		# group:    files db

  * 変更後

		passwd:   files # db
		group:    files # db

なお、nsswitch.confの変更を有効にするには、サービスも含めた全Cygwinアプリの再起動が必要となる。この仕様については、[The /etc/nsswitch.conf file](https://cygwin.com/cygwin-ug-net/ntsec.html#ntsec-mapping-nsswitch)に以下のように記載がある。

> The /etc/nsswitch.conf file is read exactly once by the first process of a Cygwin process tree. If there was no /etc/nsswitch.conf file when this first process started, then no other process in the running Cygwin process tree will try to read the file.
>
> If you create or change /etc/nsswitch.conf, you have to restart all Cygwin processes that need to see the change. If the process you want to see the change is a child of another process, you need to restart all of that process's parents, too.

### 4. Cygserverの再開 ###

  1. 管理者権限でpowershellを起動
  2. set-serviceコマンドレットでCygserverの自動起動を停止

		> Set-Service -StartupType Automatic cygserver
  3. Windowsの再起動
	 nsswitch.confの変更を反映する意味も持つ

![start cygserver](/assets/2021/02/20/start-cygserver.gif)
