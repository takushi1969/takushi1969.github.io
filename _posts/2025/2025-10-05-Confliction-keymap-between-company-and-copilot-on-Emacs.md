---
layout: post
title:  Confliction keymap between company-mode and copilot-mode on Emacs
date:   2025-10-05
tags:   emacs copilot company
---

<a id="org84c7f24"></a>

## 結論

Emacsで、companyモードとcopilotモードのキーバインドの衝突を避けるには、
以下のような設定が望ましい。

    (add-hook 'after-init-hook 'global-company-mode)
    (use-package copilot
      :ensure t
      :hook (prog-mode . copilot-mode)
      :bind (:map copilot-completion-map
                  ("<tab>" . 'copilot-accept-completion)
                  ("TAB" . 'copilot-accept-completion)
                  ("C-TAB" . 'copilot-accept-completion-by-word)
                  ("C-<tab>" . 'copilot-accept-completion-by-word)))


<a id="orgff0580e"></a>

## 経緯

Ubuntuのパッケージで配布されているEmacsで[copilot](https://github.com/copilot-emacs/copilot.el)を使用しているが、
[copilot](https://github.com/copilot-emacs/copilot.el)向けの適切な設定が今一つわからない。その理由の多くは自身のelisp
スキルの拙さに起因しているが、[copilotのホームページ](https://github.com/copilot-emacs/copilot.el)にある[一般的Emacsの
設定方法](https://github.com/copilot-emacs/copilot.el?tab=readme-ov-file#general-configurations)の記載も以下のように階層が深く、一般向けでないように思う。

<p class="verse">
General Configrations<br />
&#xa0;&#xa0;1. Load copilot.el<br />
&#xa0;&#xa0;&#xa0;&#xa0;Option1: Load via use-package(recommended)<br />
&#xa0;&#xa0;&#xa0;&#xa0;Option3: Load manually<br />
&#xa0;&#xa0;2. Configration copletion<br />
&#xa0;&#xa0;&#xa0;&#xa0;Option1: Use copilot-mode to automatically provide completions<br />
&#xa0;&#xa0;&#xa0;&#xa0;Option2: Manualy provide completions<br />
&#xa0;&#xa0;3. Configure completion acceptation<br />
&#xa0;&#xa0;4. Configure LSP Settings<br />
</p>

このため、[copilot](https://github.com/copilot-emacs/copilot.el)の使用当初は[Doom Emacs](https://github.com/copilot-emacs/copilot.el?tab=readme-ov-file#example-for-doom-emacs)の設定を参考に以下のように設定
していた。

    (use-package copilot
      :hook (prog-mode . copilot-mode)
      :bind (:map copilot-completion-map
                  ("<tab>" . 'copilot-accept-completion)
                  ("TAB" . 'copilot-accept-completion)
                  ("C-TAB" . 'copilot-accept-completion-by-word)
                  ("C-<tab>" . 'copilot-accept-completion-by-word)
                  ("C-n" . 'copilot-next-completion)
                  ("C-p" . 'copilot-previous-completion)))

しかし、[copilot](https://github.com/copilot-emacs/copilot.el)の使用を進めるうち、[company](https://company-mode.github.io/)のcompletionのpopupの操作が
できないことに気づいた。理由は単純で、copilot-completion-mapのキーバイ
ンドで、 `C-n` 、 `C-p` を設定していたためである。

そこで、再度、[一般的Emacsの設定方法](https://github.com/copilot-emacs/copilot.el?tab=readme-ov-file#general-configurations)を読み直して、[結論](#org84c7f24)の設定に落ち着いた。

