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

<p>When one wants to publish this data, a first step is always to remove the name column. Once that's done, the dataset looks like the one shown <a href="#fakedatanonames">below</a>. Again, each row still belongs to one individual (even though the name has been removed). So, we shall continue to call it unit record level data.</p>

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

<p>Now focus on the suburb of Redfern. The <a href="#tabularfake">table below</a> shows another way of data representation, which we call the tabular format. Basically, it divides the dataset into parts, in our case gender and age distribution, and shows <i>counts</i> of people that are in each gender-age pair. Notice how you can exactly recreate the three Redfern rows in the unit record level <a href="#fakedatanonames">representation</a>, simply by copying the number of times each tuple (Redfern, Age, Gender) appears. Take away here is that even though the data is merely a "table" rather than in unit record format, it is still possible to interconvert them for sub-populations of interest (Redfern in our case) or for the entire database (by constructing similar tables for other suburbs).</p> 

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
    <td>10-19</td> 
    <td>0</td>
    <td>0</td>
  </tr>
  <tr>
    <td>20-29</td> 
    <td>2</td>
    <td>1</td>
  </tr>
  <tr>
    <td>30-39</td> 
    <td>0</td>
    <td>0</td>
  </tr>
  <tr>
    <td>40-49</td> 
    <td>0</td>
    <td>0</td>
  </tr>
  <tr>
    <td>50-59</td> 
    <td>0</td>
    <td>0</td>
  </tr>
  <tr>
    <td>60-69</td> 
    <td>0</td>
    <td>0</td>
  </tr>
  <tr>
    <td>70-79</td> 
    <td>0</td>
    <td>0</td>
  </tr>
</table>

<h2>Privacy</h2>

<p>Now imagine that you are given the task of building a tool that allows access to this dataset (so that something useful can be learned from the data, e.g., average age of a Redferen inhabitant). But, you would want the identity of the people hidden for privacy reasons. You therefore can't just build a tool with a simple button clicking which downloads the entire Fake Dataset. You would be tempted to release the Fake Dataset with No Names, however. But notice that even though names have been removed, some of the rows in the dataset are unique. This means that someone knowing someone's age, gender and residential location can easily attach an identity to the corresponding row of the data. Is this just an artefact of this (fake) dataset or is it a general concern? It turns out that it is indeed <a href="https://dataprivacylab.org/projects/identifiability/paper1.pdf">true</a> of real datasets. This has been known for a long time, and I won't go into the details (interested readers can click the link).</p>

<p>Your remaining option then is to release some sort of tabular version. You can build a tool that allows the user to construct different tables from the Fake Dataset (with No Names). But we have seen that it is easy to interchange between the tabular and unit record data format. Thus, something more advanced needs to be done. One straightforward way is to suppress low counts. You decide that any table entry that is exactly 1 will be suppressed to 0. Thus, there is no fear of identity breach if the anaylst (user of your tool) can't tell if a count is an actual 0 or not. Thus, using this technique, the count of 1 appearing in the 20-29 row and Female column (Liz's data) of the table <a href="#tabularfake">above</a> would be set to 0. Alas, this is problematic too. For instance, the analyst can use your tool to create two tables. One table for the count of people in Redfern; and another for the number of males in Redfern. The <i>difference</i> between the two is exactly Liz's data. This kind of attack is generally called a differencing attack, for obvious reasons.</p>

<p>Slightly more complicated strategy is to <i>perturb</i> those counts by adding random noise. We can't add completely random noise, or else the output will be junk and useless for legitimate purposes. So, you decide to add random noise within a fixed interval. Let's assume this is +/-5. Thus, instead of reporting 2 or 1, you would sample "fresh" noise at random, add to the count and return the <i>noisy count</i> as an entry in the table (for this demonstration, let's assume that we are happy with negative numbers being returned). The problem with this is that the analyst can create a table multiple times. If fresh noise is added to the count each time then one can launch an <i>averaging attack</i>. Why does it work? Notice that the noise is in the interval +/-5. Since the noise is sampled randomly each time, after a certain number of trials there will be an almost equal distribution of positives and negatives. We can add all the noisy counts, and divide it by the number of times the same table was requested to "average out" the error.</p>
