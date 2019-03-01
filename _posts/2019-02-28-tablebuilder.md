---
layout: single
title:  "Attack"
categories: blog
---

<p>Recently, we made <a href="https://arxiv.org/pdf/1902.06414.pdf">our privacy paper</a> public. The paper shows two attacks on an algorithm used by the Australian Bureau of Statistics to allow people query Australian census data. The algorithm in fact is used by a web-based tool called <a href="https://www.abs.gov.au/websitedbs/censushome.nsf/home/tablebuilder">TableBuilder</a>.</p>

<p>I have realised that due to the technical/mathematical nature of the paper, the attacks might be misinterpreted as highly complex or involved. This is certainly not the case, as I shall try and explain in this blog post. Most of the mathematics in the paper exists to formally show why the attack is expected to succeed and to derive probability bounds. The attacks themselves are fairly straightforward. The first of the two attacks is just a demonstration of why it is important to apply <a href="https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle">Kerckhoffs's principle</a> in data privacy, as it is done in cryptography. So, I will skip this, and concentrate on the second attack; which is the main contribution of the paper.</p>

<p> In what follows, I will try to explain step-by-step why the algorithm behind TableBuilder (which I shall refer to as the TBE algorithm for reasons mentioned in the paper), is the way it is, and how we were able to launch our attack. </p>

<h2>Unit Record vs Tabular Data</h2>

<p>Table </p>

<table id="#fakedata" summary="A Fake Dataset">
  <tr>
    <th>Name</th>
    <th>Suburb</th> 
    <th>Age</th>
    <th>Gender</th>
  </tr>
  <tr>
    <td>Jack</td>
    <td>Redfern</td> 
    <td>20-29</td>
    <td>Male</td>
  </tr>
  <tr>
    <td>John</td>
    <td>Redfern</td> 
    <td>20-29</td>
    <td>Male</td>
  </tr>
  <tr>
    <td>Noelia</td>
    <td>Newtown</td> 
    <td>30-39</td>
    <td>Female</td>
  </tr>
  <tr>
    <td>Liz</td>
    <td>Redfern</td> 
    <td>20-29</td>
    <td>Female</td>
  </tr>
  <tr>
    <td>Jimmy</td>
    <td>Surry Hills</td> 
    <td>40-49</td>
    <td>Male</td>
  </tr>
  <tr>
    <td>Susan</td>
    <td>Darlinghurst</td> 
    <td>70-79</td>
    <td>Female</td>
  </tr>
</table>
