---
layout:	post
title:	Ruby Net::Telnet#cmdでタイムアウトする時は
author:	takushi1969
tags:	ruby net::telnet
---
サーバの管理の自動化の一環で、Rubyの[Net::Telnet](https://www.rubydoc.info/gems/net-telnet/Net/Telnet)パッケージを使用することがある。そして、サーバのコマンド実行には、ログイン後の[Net::Telnet#cmd](https://www.rubydoc.info/gems/net-telnet/Net/Telnet#cmd-instance_method)メソッドを使用する。

サーバのコマンド実行の経過時間が10秒を超過すると、Net::ReadTimeout例外が発生する。

予めに、サーバのコマンド実行の経過時間の最長時間が把握しているのであれば、[Net::Telnet.new](https://www.rubydoc.info/gems/net-telnet/Net/Telnet#initialize-instance_method)メソッドのTimeoutオプションに適切な秒数を指定しておくことで、Net::ReadTimeout例外を回避することができる。

## 例 ##

### サーバのコマンド例 ###

以下のコードを/tmp/hello.shに格納。
``` shell
#!/usr/bin/bash -e
sleep 20

echo 'Hello, World'
```
### Timeoutオプション未設定 ###

#### Net::Telnetクライアント ####

``` ruby
#!/usr/bin/env ruby
#coding UTF-8

require 'net/telnet'

server = Net::Telnet("Host" => "localhost")
server.login("vagrant", "vagrant") {|c| print c}
server.cmd("/tmp/hello.sh") {|c| print c}
server.cmd("exit") {|c| print c}
```

#### 結果 ####

    root@ubuntu-focal:~# ./telnet.rb
    ubuntu-focal login: vagrant
    Password:
    vagrant@ubuntu-focal:~$ /tmp/hello.sh
    Traceback (most recent call last):
            2: from ./telnet.rb:8:in `<main>'
            1: from /usr/lib/ruby/vendor_ruby/net/telnet.rb:694:in `cmd'
    /usr/lib/ruby/vendor_ruby/net/telnet.rb:557:in `waitfor': Net::ReadTimeout with
    "timed out while waiting for more data" (Net::ReadTimeout)
    root@ubuntu-focal:~#

### Timeoutオプション設定 ###

#### Net::Telnetクライアント ####

``` ruby
#!/usr/bin/env ruby
#coding UTF-8

require 'net/telnet'

server = Net::Telnet("Host" => "localhost", "Timeout" => 30)
server.login("vagrant", "vagrant") {|c| print c}
server.cmd("/tmp/hello.sh") {|c| print c}
server.cmd("exit") {|c| print c}
```
#### 結果 ####

    root@ubuntu-focal:~# ./telnet.rb
    ubuntu-focal login: vagrant
    Password:
    vagrant@ubuntu-focal:~$ /tmp/hello.sh
    Hello, World
    vagrant@ubuntu-focal:~$ exit
    logout
    root@ubuntu-focal:~#

