---
layout: post
title: 'Lovely Soup - DEF CON Delhi Group | DC9111 CTF 2022'
tags: [DC9111, DEF CON, CTF]
color: rgb(0,90,200)
permalink: lovely-soup-writeup/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

**Challenge Category: OSINT**

**Challenge:**
The author is running his own tech blog and is hiding a weird thing on its source. (Hey player, do you like copy-pasting things and throwing your waste in the bin?).
<br>Author: cybersapien

<!--more-->

<hr>

<h1 id="contents-">Contents <a name="top"></a></h1>
* TOC
{:toc}

<hr>

## Overview

* I created this challenge as part of DC9111 2022 CTF.
* This challenge was intended to be solved using OSINT and Python Beautiful Soup, that’s why its name is **Lovely Soup**.
* Open Source INTelligence (OSINT) is the practice of collecting information from publicly available sources.
* Beautiful Soup is a library that makes it easy to scrape information from web pages. It allows you to interact with HTML in a similar way to how you interact with a web page using developer tools. It can be used to fetch URLs from a website, which was intended for this challenge. Read more about Beautiful Soup here - **[https://www.crummy.com/software/BeautifulSoup/bs4/doc/](https://www.crummy.com/software/BeautifulSoup/bs4/doc/)**.
* This challenge was intended to be solved using Beautiful Soup but it can also be solved by just guessing Pastebin from challenge description (which many people did).

<hr>

## How to Solve 

1). Simply, googling the text *"cybersapien defcon"* gives some websites links and also the description says *"the author is running his own tech blog"*. This gives us the hint to go for **[cybersapien.tech](cybersapien.tech)** blog link.
<center><img src="/assets/img/ctf/dc9111-2022/1.png" alt="Featured Section"></center>

2). Next part of the description says *"hiding a weird thing on its source"* which is hinting for checking the source code of the blog. But there is lot of text.

3). Next part of description says *"Hey player, do you like copy-**pasting** things and throwing your waste in the **bin**?"*. This hints for some **pastebin** link.

4). Either we can directly search for pastebin in the source code or we can also use following Python Beautiful Soup code as the challenge name (Lovely Soup) hints.

```
import requests
from bs4 import BeautifulSoup

URL = "https://cybersapien.tech/"
page = requests.get(URL)

soup = BeautifulSoup(page.content, "html.parser")

for link in soup.find_all('a'):
    if link.get('href').startswith("https"):
        print(link.get('href'))
```
5). Running this code gives us many links. Among these links, obviously, we need to go to Pastebin link.
<center><img src="/assets/img/ctf/dc9111-2022/2.png" alt="Featured Section"></center>

6). Going to this link, gives us a base64 encoded text.

7). Decode it using **[https://www.base64decode.org/](https://www.base64decode.org/)**.
<center><img src="/assets/img/ctf/dc9111-2022/3.png" alt="Featured Section"></center>

<hr>

This is the flag.<br>
This confirms that we have successfully completed the challenge.

<hr>

**I hope, this post helped you to solve this CTF challenge easily and you must have learned something new.**

**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

You can also **[Buy Me A Coffee](https://www.buymeacoffee.com/cybersapien)** if you love the content and want to support this blog page!

<script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="cybersapien" data-color="#FFDD00" data-emoji=""  data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#000000" data-font-color="#000000" data-coffee-color="#ffffff" ></script>

<a href="#top" style="float: right"><strong>Back to Top⮭</strong> </a>