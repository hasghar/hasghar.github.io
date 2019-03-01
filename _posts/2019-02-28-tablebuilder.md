---
layout: single
title:  "Attack"
categories: blog
---

<p>Recently, we made <a href="https://arxiv.org/pdf/1902.06414.pdf">our privacy paper</a> public. The paper shows two attacks on an algorithm used by the Australian Bureau of Statistics to allow people query Australian census data. The algorithm in fact is used by a web-based tool called <a href="https://www.abs.gov.au/websitedbs/censushome.nsf/home/tablebuilder">TableBuilder</a>.</p>

<p>I have realised that due to the technical/mathematical nature of the paper, the attacks might be misinterpreted as highly complex or involved. This is certainly not the case, as I shall try and explain in this blog post. Most of the mathematics in the paper exists to formally show why the attack is expected to succeed and to derive probability bounds. The attacks themselves are fairly straightforward. The first of the two attacks is just a demonstration of why it is important to apply <a href="https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle">Kerckhoffs's principle</a> in data privacy, as it is done in cryptography. So, I will skip this, and concentrate on the second attack; which is the main contribution of the paper.</p>

<p> In what follows, I will try to explain step-by-step why the algorithm behind TableBuilder (which I shall refer to as the TBE algorithm for reasons mentioned in the paper), is the way it is, and how we were able to launch our attack. </p>

<h2>Unit Record vs Tabular Data</h2>

<p>The <a href="#fakedata">table below</a> shows a fake dataset which we shall use as an example. This is pretty much the same example used in the paper, except that I have now put fictitious names for each row indicating that the data in that row belongs to that person. This representation of the dataset is what can be called <i>unit record data</i>. However, there does not seem to be well defined. For instance, the definition <a href="https://www.cancer.nsw.gov.au/glossary/unit-record-data">here</a> seems to agree with how we have defined it, but the one <a href="https://toolkit.data.gov.au/index.php/Definitions#Unit_Record_and_Integrated_Data">here</a> seems to define it as the most granular level of data (e.g., the age could be exact instead of generalised to 10 year bracket as in the table). Nevertheless, we will consider any dataset presented in the form of the Fake Dataset as unit record level data, i.e., each row containing data from a single individual.</p>

<table id="fakedata" align="center">
  <caption><b>A Fake Dataset</b></caption>
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

<p>When one wants to publish this data, a first step is always to remove the name column. Once that's done, the dataset looks like the one shown <a href="#fakedatanonames">below</a>. Again, each row still belongs to one individual (even though the name has been removed). So, we shall continue to call it unit record level data. Now focus on the suburb of Redfern. </p>

<table id="fakedatanonames" align="center">
  <caption><b>A Fake Dataset with No Names</b></caption>
  <tr>
    <th>Suburb</th> 
    <th>Age</th>
    <th>Gender</th>
  </tr>
  <tr>
    <td>Redfern</td> 
    <td>20-29</td>
    <td>Male</td>
  </tr>
  <tr>
    <td>Redfern</td> 
    <td>20-29</td>
    <td>Male</td>
  </tr>
  <tr>
    <td>Newtown</td> 
    <td>30-39</td>
    <td>Female</td>
  </tr>
  <tr>
    <td>Redfern</td> 
    <td>20-29</td>
    <td>Female</td>
  </tr>
  <tr>
    <td>Surry Hills</td> 
    <td>40-49</td>
    <td>Male</td>
  </tr>
  <tr>
    <td>Darlinghurst</td> 
    <td>70-79</td>
    <td>Female</td>
  </tr>
</table>

<table id="tabularfake" align="center">
  <caption><b>A Table for Redfern from the Fake Dataset</b></caption>
  <tr>
    <th rowspan="2">Age</th> 
    <th colspan="2">Gender</th>
  </tr>
  <tr>
    <th>Male</th> 
    <th>Female</th>
  </tr>
  <tr>
    <tr>10-19</tr> 
    <tr>0</tr>
    <tr>0</tr>
  </tr>
  <tr>
    <tr>20-29</tr> 
    <tr>2</tr>
    <tr>1</tr>
  </tr>
  <tr>
    <tr>30-39</tr> 
    <tr>0</tr>
    <tr>0</tr>
  </tr>
  <tr>
    <tr>40-49</tr> 
    <tr>0</tr>
    <tr>0</tr>
  </tr>
  <tr>
    <tr>50-59</tr> 
    <tr>0</tr>
    <tr>0</tr>
  </tr>
  <tr>
    <tr>60-69</tr> 
    <tr>0</tr>
    <tr>0</tr>
  </tr>
  <tr>
    <tr>70-79</tr> 
    <tr>0</tr>
    <tr>0</tr>
  </tr>
</table>
