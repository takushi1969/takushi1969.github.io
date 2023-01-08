---
title: The default environment of selenium on Ubuntu
---
[Ubuntu Jammy](https://releases.ubuntu.com/22.04/)で、Python3によるwwwのscrapingを実行したい。  

Ubuntuには、pyrhon3でscrapingするパッケージとして[python3-selenium](https://packages.ubuntu.com/ja/jammy/python3-selenium)がある。  
python3-seleniumは、browserを制御するdriverへのAPIのbindingであるため、実態であるdiverと、driverに対応したbrowserが必要となる。このソフトウェア・スタックについては、Seleniumの[The Parts and Pieces](https://www.selenium.dev/documentation/overview/components/#the-parts-and-pieces)に詳細な記載がある。  

そして、Ubuntuのpython3-seleniumは痒いところに手が届いており、(chromiumだけではあるが)driverとbrowserを同時にインストールしてくれる。以下のpython3-seleniumのインストールログが示す。
> Start-Date: 2023-01-07  15:14:41  
> Commandline: apt install -y python3-selenium  
> Requested-By: taku (1000)  
> Install: python3-selenium:amd64 (4.0.0~a1+dfsg1-1.1), chromium-browser:amd64 (1:85.0.4183.83-0ubuntu2, automatic), chromium-chromedriver:amd64 (1:85.0.4183.83-0ubuntu2, automatic)  
> End-Date: 2023-01-07  15:15:24  

さて、構築した環境を使用して、実際にPythonでchromiumを制御してみる。  
以下のPyhtonコードは、[Selenium with Python](https://selenium-python.readthedocs.io/index.html)から参考にした。  

{% highlight python %}
#!/usr/bin/env python
#coding: UTF-8

from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
import time

with webdriver.Chrome() as driver:
    driver.get("http://www.python.org")
    assert "Python" in driver.title
    elem = driver.find_element(By.NAME, "q")
    elem.clear()
    elem.send_keys("pycon")
    elem.send_keys(Keys.RETURN)
    time.sleep(3)
    assert "No results found." not in driver.page_source
{% endhighlight %}

動作の様子はこちら。
<video autoplay loop width=720px height=360px controls muted>
	<source src="/assets/images/2023/2023-01-07-ubuntu_selenium_01.webm">
</video>

