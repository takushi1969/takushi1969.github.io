---
layout: post
title: How to setup Jekyll for github
tags: github-pages
---

[Jekyll](https://jekyllrb.com/)のページには、GitHub Pagesで公開するための簡単な手順が以下のように記載されている。

    
    ~ $  gem install bundler jekyll
    ~ $  jekyll new my-awesome-site
    ~ $  cd my-awesome-site
    ~/my-awsome-site $  bundle exec jekyll serve
    # => Now browse to http://localhost:4000

しかし、GitPagesが推奨する手順は少し違うようで、[Blogging with Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll)に詳細な記載がある。github-pagesのGemを使うことで、ローカル環境とGitHub Pagesの環境を同調してくれるらしい。
