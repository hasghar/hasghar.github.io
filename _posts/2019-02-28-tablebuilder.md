---
layout: single
title:  "Attack"
categories: blog
---

<p>Recently, we published <a href="https://arxiv.org/pdf/1902.06414.pdf">our paper on arXiv</a>. The paper shows two attacks on an algorithm used by the Australian Bureau of Statistics to allow people query Australian census data through their tool called <a href="https://www.abs.gov.au/websitedbs/censushome.nsf/home/tablebuilder">TableBuilder</a>.</p>

<p>Due to the technical/mathematical nature of the paper, the attacks might be misinterpreted as highly complex or involved. This is not the case, as I shall try and explain in this blog post. Most of the mathematics in the paper is there to formally show why the attack is expected to succeed and to derive probability bounds. The attacks themselves are fairly straightforward. The first of the two attacks is just a demonstration of why it is important to apply <a href="https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle">Kerckhoffs's principle</a> to data privacy, just like it is done in cryptography. So, I will skip this, and concentrate on the second attack; which is the main contribution of the paper.</p>

<p> In what follows, I will try to explain step-by-step why the algorithm behind TableBuilder (which I shall refer to as the TBE algorithm named after its <a href="https://www.unece.org/fileadmin/DAM/stats/documents/ece/ces/ge.46/2013/Topic_1_ABS.pdf">authors</a>), was designed the way it is, and how it is succumbs to our attack. </p>

<h2>Unit Record vs Tabular Data</h2>

<p>The <a href="#fakedata">table below</a> shows a fake dataset which we shall use as an example. This is pretty much the same example used in our paper, except that I have now put fictitious names for each row indicating that the data in that row belongs to that person. This representation of the dataset is what is called <i>unit record data</i>. However, the term "unit record data" does not seem to be well defined. For instance, the definition <a href="https://www.cancer.nsw.gov.au/glossary/unit-record-data">here</a> agrees with how we have defined it, but the one <a href="https://toolkit.data.gov.au/index.php/Definitions#Unit_Record_and_Integrated_Data">here</a> seems to define it as the most granular level of data (e.g., the age could be exact instead of generalised to 10 year bracket as in the table). Nevertheless, we will consider any dataset presented in the form of the Fake Dataset as unit record level data, i.e., each row containing data from a single individual, since there is no limit to how granular data could be.</p>

<table id="fakedata" align="center" style="width:400px">
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

<p>When one wants to publish this data, a first step is always to remove the name column. Once that's done, the dataset looks like the one shown <a href="#fakedatanonames">below</a>. Again, each row still belongs to one individual; even though the name has been removed. So, we shall continue to call it unit record level data.</p>

<table id="fakedatanonames" align="center" style="width:400px">
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

<p>Now focus on the suburb of Redfern. The <a href="#tabularfake">table below</a> shows another way of data representation, which we call the tabular format. Basically, it divides the dataset into parts, in our case gender and age distribution, and shows <i>counts</i> of people that are in each gender-age pair. Notice how you can exactly recreate the three Redfern rows in the unit record level <a href="#fakedatanonames">representation</a>, simply by copying the number of times each tuple (Redfern, Age, Gender) appears. The take away here is that even though the data is formatted as a table rather than in unit record format, it is still possible to convert the data in the latter format for sub-populations of interest (Redfern, in our case) or for the entire database (by constructing similar tables for other suburbs).</p> 

<table id="tabularfake" align="center" style="width:400px">
  <caption><b>A Table for Redfern from the Fake Dataset</b></caption>
  <tr>
    <th rowspan="2">Age</th> 
    <th colspan="2" align="center">Gender</th>
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

<p>Now imagine that we wish to build a tool that allows access to this dataset (so that something useful can be learned from the data, e.g., average age of a Redfern inhabitant). But, we want the identity of the people hidden for privacy reasons. We therefore can't just build a tool which downloads the entire Fake Dataset by simply clicking a button. We would be tempted to release the <a href="#fakedatanonames">Fake Dataset with No Names</a>. But notice that even though names have been removed, most of the rows in the dataset are unique (in fact, all except the first two). This means that an attacker knowing someone's age, gender and residential location can easily attach an identity to the corresponding row of the data. Is "most rows being unique" just an artefact of this (fake) dataset? or is it true in general? It turns out that it is indeed <a href="https://dataprivacylab.org/projects/identifiability/paper1.pdf">true</a> of real datasets with moderately large dimension (number of columns). This has been known for a long time, and I won't go into the details. Interested readers can click the link or <a href ="https://arxiv.org/pdf/cs/0610105.pdf">many</a> <a href="http://science.sciencemag.org/content/347/6221/536.full">other</a> <a href="https://arxiv.org/abs/1712.05627">studies</a> demonstrating such "re-identification" attacks.</p>

<p>Our remaining option then is to release some sort of tabular version of the data. We can build a tool that allows the user to construct different tables from the Fake Dataset (with No Names). But we have seen that it is easy to interchange between the tabular and unit record data format. Thus, something more advanced needs to be done. For this, we consider a dataset with more rows than the Fake Dataset, whose Redfern sub-population statistics is given by table T1 below:

<table id="tabularfakemoredata" align="center" style="width:400px">
  <caption><b>T1: A Fake Table for Redfern</b></caption>
  <tr>
    <th rowspan="2">Age</th> 
    <th colspan="2" align="right">Gender</th>
  </tr>
  <tr>
    <th>Male</th> 
    <th>Female</th>
  </tr>
  <tr>
    <td>10-19</td> 
    <td>90</td>
    <td>20</td>
  </tr>
  <tr>
    <td>20-29</td> 
    <td>2</td>
    <td>1</td>
  </tr>
  <tr>
    <td>30-39</td> 
    <td>35</td>
    <td>73</td>
  </tr>
  <tr>
    <td>40-49</td> 
    <td>40</td>
    <td>51</td>
  </tr>
  <tr>
    <td>50-59</td> 
    <td>100</td>
    <td>19</td>
  </tr>
  <tr>
    <td>60-69</td> 
    <td>31</td>
    <td>73</td>
  </tr>
  <tr>
    <td>70-79</td> 
    <td>3</td>
    <td>0</td>
  </tr>
</table>

Now even though this can still be converted into unit record format, notice that if the target individual of the attack is say a Redfern male in the age range 40-49, then the said individual "hides" behind 39 other individuals in the table. Based on this data table, the attacker has no way to distinguish between the 40 individuals. Thus, high counts seem reasonably safe to publish. The areas of concern are low counts, e.g., rows Age 20-29 and Age 70-79 in table T1. One straightforward way to protect privacy is to suppress low counts. You decide that any table entry that is (say) less than or equal to 3 will be suppressed to 0. Thus, there is no fear of identity breach if the anaylst (user of the tool) can't tell if a count is an actual 0 or not. So, using this technique, the count of 3 appearing in the Age 70-79 row and Male column of table <a href="#tabularfakemoredata">T1</a> would be set to 0, making no distincution between the actual 0 (corresponding to the Female column) and this 0. Alas, this is problematic too. For instance, the analyst can use our tool to create another table, this time only asking about people in the Age range 60-79, as shown below:

<table id="tabular60to79" align="center" style="width:400px">
  <caption><b>Another Fake Table for Redfern</b></caption>
  <tr>
    <th rowspan="2">Age</th> 
    <th colspan="2" align="center">Gender</th>
  </tr>
  <tr>
    <th>Male</th> 
    <th>Female</th>
  </tr>
 <tr>
    <td>60-79</td> 
    <td>34</td>
    <td>73</td>
  </tr>
</table>

Using this table and computing the <i>difference</i> between the counts reveals that the actual count behind Males in the Age range 70-79 is 3, instead of zero. This kind of attack is generally called a differencing attack, for obvious reasons.</p>

<p>To circumvent this, another strategy is that, in addition to suppression, <i>perturb</i> the counts (all counts; not just low counts) by adding random noise. The analysts thus sees noisy counts and cannot carry out the differencing attack above. But notice that we can't add completely random noise, or else the output will be junk and useless for legitimate purposes. So, we can decide to add random noise within a fixed interval. Let's assume this is +/-5. Thus, instead of reporting the actual count, we would sample "fresh" noise at random, add to the count and return the <i>noisy count</i> as an entry in the table. The problem with this is that the analyst can create a table multiple times. If fresh noise is added to the count each time then one can launch an <i>averaging attack</i>. Why does it work? Notice that the noise is in the interval +/-5. Since the noise is sampled randomly each time, after a certain number of trials there will be an almost equal distribution of positives and negatives. We can add all the noisy counts, and divide it by the number of times the same table was requested to "average out" the error. An illustration of this is shown in the graph below. The analyst's target is the cell (Redfern, 50-59, Female) whose true count is 19 according to Table <a href="#tabularfakemoredata">T1</a>. The analyst asks the same query 100 times, and the graph shows how the average count converges to the true value. One can find the nearest integer to the fracitonal value obtained in the end (in this case 19.07) to obtain the true count.</p>

<img src="https://hasghar.github.io/assets/images/av-attacks-plot.png" alt="Averaging Attack">

<h2>The TBE Algorithm</h2>
<p>
The failure of the above techniques then led to the "same contributor, same noise" technique used in the TBE algorithm. Under this rule, if two queries (cells in a table) are satisfied by the exact same set of contributors, then intead of adding fresh noise, the same noise will be added. This immediately renders the above-mentioned averaging attack useless. However, what we show in our paper is that there is another way to launch an averaging attack. To see this, let's assume that the perturbation parameter in TBE is +/-5. Consider Table <a href="#tabularfakemoredata">T1</a> again. If we ask TBE (via the TableBuilder tool) the count of people aged 30-39 and 40-49, we would get something like this:

<table id="tablebuilderoutput" align="center" style="width:400px">
  <caption><b>T2: Example TableBuilder Output</b></caption>
  <tr>
    <th rowspan="2">Age</th> 
    <th colspan="2" align="right">Gender</th>
  </tr>
  <tr>
    <th>Male</th> 
    <th>Female</th>
  </tr>
  <tr>
    <td>30-39</td> 
    <td>37</td>
    <td>71</td>
  </tr>
  <tr>
    <td>40-49</td> 
    <td>44</td>
    <td>53</td>
  </tr>
  <tr>
    <td><b>Total</b></td> 
    <td>74</td>
    <td>121</td>
  </tr>
</table>
</p>


