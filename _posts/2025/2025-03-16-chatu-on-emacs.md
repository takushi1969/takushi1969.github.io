---
title: Advanced Markdown Live Preview, which has PlanUML images, on Emacs
layout: post
tags: emacs
---
# はじめに

[Emacs](https://www.gnu.org/software/emacs/)で技術資料の
[Markdown](https://daringfireball.net/projects/markdown/)を編集してい
ると、[PlantUML](https://plantuml.com/ja/)で生成したダイアグラムを挿入
したくなるシーンが多々ある。

[Visual Studio Code](https://code.visualstudio.com)は、
[MPE](https://shd101wyy.github.io/markdown-preview-enhanced/#/)プラグ
インを導入することで埋め込みのPlantUML記法をサポートしている。ただし、
埋め込み記法は、Viewerを選ぶのでMPEの方式も一長一短のように思う。

一方で、EmacsもVisual Studio Codeの後塵を排するわけもなく、
[Markdown Mode](https://jblevins.org/projects/markdown-mode/)と
[chatu](https://github.com/kimim/chatu)のコンビネーションで、シンプル
かつ使い勝手の良いPlantUMLのダイアログの挿入手法を提供している。

# 必要パッケージ

- [Markdown Mode](https://jblevins.org/projects/markdown-mode/)
- [chatu](https://github.com/kimim/chatu)
    - [PlantUML mode](https://github.com/skuro/plantuml-mode)

        chatuが`plantuml-jar-path`を使用するので、Emacsの初期化ファイ
        ルなどで定義が必要。

        - 例
		
		    macOSに[macportsでplantuml](https://ports.macports.org/search/?q=plantuml&name=on)
            をインストールした場合の指定例を以下に示す。
		
		        (setq plantuml-jar-path "/opt/local/share/java/plantuml/plantuml.jar")

各パッケージの設定については、それぞれのオフィシャル・サイトを確認れた
い。

# 使用方法

以下の全ての操作は、Emacs上で行う。

1. chatu-new関数のコール

    Markdown Modeで編集中のファイルにchatu専用のコメントを挿入して保存。
	
    保存を忘れると、chatuで生成したダイアグラムが想定外のところに挿入
    されてしまう可能性がある。
2. chatu-open関数のコール

    関数コールで新たに開いた外部PlantUMLファイルを編集して保存。
3. 1.のMardownファイルのバッファに移動  
4. chatu-add関数のコール

    1.のコメント上で、chatu-add関数をコールすることで、chatuがコメント
    の直下にダイアグラムを挿入する。

# 注記

- chatuのplantumlの生成モードはsvgのみをサポート

    [pngのサポートのpullreq。](https://github.com/kimim/chatu/pull/5)

# 使用例

![Chatu on Emacs](/assets/images/2025/0316/chatu.gif)
	
