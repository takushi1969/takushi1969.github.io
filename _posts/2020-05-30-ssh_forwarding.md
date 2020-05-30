---
layout: post
title:  "Net::SSHを使ったポート・フォワーディングの仕方"
author: takushi1969
categories: ruby ssh
---
rubyの[Net::SSH]でのポート・フォワーディングを試す。
sshコマンドでは、-Jオプションで実現できる機能。

切っ掛けは、仕事の関係でプライベート・ネットワークにあるLinuxに対して、minitestを使ったテストセットを作成したかったから。

[こちらの記事](https://qiita.com/metheglin/items/1e8459c46395c57391d1)に[Net::SSH::Gateway]を実例がある。

ここでは、[Net::SSH::Gateway]を使わずに、純粋に[Net::SSH]のみでトライしてみた。
結果、パスワード認証でのポート・フォワーディングをすることができた。

[Net::SSH]:https://github.com/net-ssh/net-ssh
[Net::SSH::Gateway]:https://github.com/net-ssh/net-ssh-gateway

以下、ソースを示す。
``` ruby
#!/usr/bin/env ruby
#coding: UTF-8

require 'net/ssh'

module Env
  module Target
    HOST="192.168.33.30"
    USER="vagrant"
    PWD="vagrant"
    PORT=22
  end

  module Proxy
    HOST="192.168.33.20"
    USER="vagrant"
    PWD="vagrant"
    PORT=2222
  end
end

def start_proxy
  stop_proxy_flag = false
  Signal.trap('INT') {stop_proxy_flag = true}
  
  Net::SSH.start(Env::Proxy::HOST, Env::Proxy::USER, password: Env::Proxy::PWD) do |ssh|
    ssh.forward.local(Env::Proxy::PORT, Env::Target::HOST, Env::Target::PORT)
    ssh.loop(0.1) {not stop_proxy_flag}
  end
end

def exec
  Net::SSH.start("localhost", Env::Target::USER, password: Env::Target::PWD, port: Env::Proxy::PORT) do |ssh|
    puts ssh.exec!("hostname")
  end
end

child = Process.fork {start_proxy}
sleep(5)
if not child
  $stderr.puts "Can't start proxy"
  exit(false)
end

exec
Process.kill("INT", child)
Process.wait
```

次回は、鍵認証によるsshのポート・フォワードを試してみようと思案中。
