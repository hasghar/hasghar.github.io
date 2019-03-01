---
layout: single
title:  "Attack"
categories: blog
---

<p>Recently, we made <a href="https://arxiv.org/pdf/1902.06414.pdf">our privacy paper</a> public. The paper shows two attacks on an algorithm used by the Australian Bureau of Statistics to allow people query Australian census data. The algorithm in fact is used by a web-based tool called <a href="https://www.abs.gov.au/websitedbs/censushome.nsf/home/tablebuilder">TableBuilder</a>.</p>

<p>I have realised that due to the highly technical/mathematical nature of the paper, the attacks might be misinterpreted as highly complex or involved. This is certainly not the case, as I shall try and explain in this blog post. Most of the mathematics in the paper exists to formally show why the attack is expected to succeed and to derive probability bounds. The attacks themselves are fairly straightforward. The first of the two attacks is just a demonstration of why it is important to apply <a href="https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle">Kerckhoffs's principle</a> to the realm of data privacy, as it is done in cryptography. So, I will skip this, and concentrate on the second attack; which is the main contribution of the paper.</p>

<h2>What is this all about?</h2>
