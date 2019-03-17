---
layout: single
title:  "Averaging Attacks on TableBuilder"
categories: blog
---

<p>Recently, we published <a href="https://arxiv.org/pdf/1902.06414.pdf">our paper on arXiv</a>, which shows two attacks on an algorithm used by the Australian Bureau of Statistics (ABS). The algorithm allows people query Australian census data through their tool called <a href="https://www.abs.gov.au/websitedbs/censushome.nsf/home/tablebuilder">TableBuilder</a>.</p>

<p>Due to the technical/mathematical nature of the paper, the attacks might be misinterpreted as highly complex or involved. This is not the case, as I shall try and explain in this blog post. Most of the mathematics in the paper is there to formally show why the attack is expected to succeed and to derive probability bounds. The attacks themselves are fairly straightforward. The first of the two attacks is just a demonstration of why it is important to apply <a href="https://en.wikipedia.org/wiki/Kerckhoffs%27s_principle">Kerckhoffs's principle</a> to data privacy, just as it is done in cryptography. So, I will skip this, and concentrate on the second attack; which is the main contribution of the paper.</p>

<p> In what follows, I will try to explain step-by-step why the algorithm behind TableBuilder (which I shall refer to as the TBE algorithm after its <a href="https://www.unece.org/fileadmin/DAM/stats/documents/ece/ces/ge.46/2013/Topic_1_ABS.pdf">authors</a>), was designed the way it is, and how it succumbs to our attack. </p>

<h2>Unit Record vs Tabular Data</h2>

<p>The <a href="#fakedata">table below</a> shows a fake dataset which we shall use as an example. This is pretty much the same example used in our paper, except that I have now put fictitious names behind each row indicating that the data in that row belongs to that person. This representation of the dataset is what is called <i>unit record data</i>. However, the term "unit record data" does not seem to be well defined. For instance, the definition <a href="https://www.cancer.nsw.gov.au/glossary/unit-record-data">here</a> agrees with how we have defined it, but the one <a href="https://toolkit.data.gov.au/index.php/Definitions#Unit_Record_and_Integrated_Data">here</a> seems to define it as the most granular level of data (e.g., the age could be the year of birth instead of generalised to 10 year brackets as in the table). Nevertheless, we will consider any dataset presented in the form of the Fake Dataset as unit record level data, i.e., each row containing data from a single individual, since there is no limit to how granular data could be.</p>

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

<p>Now focus on the suburb of Redfern. The <a href="#tabularfake">table below</a> shows another way of data representation, which we call the tabular format. Basically, it divides the dataset into parts, in our case gender and age distribution, and shows <i>counts</i> of people that are in each gender-age pair. Notice how you can exactly recreate the three Redfern rows in the unit record level <a href="#fakedatanonames">representation</a>, simply by copying the number of times each tuple (Redfern, Age, Gender) appears. The take away here is that even though the data is formatted as a table rather than in unit record format, it is still possible to convert the data to the latter format for sub-populations of interest (Redfern, in our case) or for the entire database (by constructing similar tables for other suburbs).</p> 

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

<p>Now imagine that we wish to build a tool that allows access to this dataset (so that something useful can be learned from the data, e.g., average age of a Redfern inhabitant). But, we want the identity of the people hidden for privacy reasons. We therefore can't just build a tool which downloads the entire Fake Dataset by simply clicking a button. We would be tempted to release the <a href="#fakedatanonames">Fake Dataset with No Names</a>. But notice that even though names have been removed, most of the rows in the dataset are unique (in fact, all except the first two). This means that an attacker knowing someone's age, gender and residential location can easily attach an identity to the corresponding row of the data. Is "most rows being unique" just an artefact of this (fake) dataset? or is it true in general? It turns out that it is indeed <a href="https://dataprivacylab.org/projects/identifiability/paper1.pdf">true</a> of real datasets with moderately large dimension (number of columns). This has been known for a long time, and I won't go into the details. Interested readers can click the link above, or <a href ="https://arxiv.org/pdf/cs/0610105.pdf">many</a> <a href="http://science.sciencemag.org/content/347/6221/536.full">other</a> <a href="https://arxiv.org/abs/1712.05627">studies</a> demonstrating such "re-identification" attacks.</p>

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

Now even though this can still be converted into unit record format, notice that if the attacker's target individual is (say) a Redfern male in the age range 40-49, then the said individual "hides" behind 39 other individuals in the table. Based on this data table, the attacker has no way to distinguish between the 40 individuals. Thus, high counts seem reasonably safe to publish. The areas of concern are low counts, e.g., rows labeled Age 20-29 and Age 70-79 in table T1. One straightforward way to protect privacy is to suppress low counts. You decide that any table entry which is (say) less than or equal to 3 will be suppressed to 0. Thus, there is no fear of identity breach if the anaylst (user of the tool; a possible attacker) can't tell if a count is an actual 0 or not. So, using this technique, the count of 3 appearing in the Age 70-79 row and Male column of table <a href="#tabularfakemoredata">T1</a> would be set to 0, making no distincution between the actual 0 (corresponding to the Female column) and this 0. Alas, this is problematic too. For instance, the analyst can use our tool to create another table, this time only asking about people in the Age range 60-79, as shown below:

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

Using this table and computing the <i>difference</i> between the counts reveals that the actual count behind males in the Age range 70-79 is 3, instead of zero. This kind of attack is generally called a <i>differencing attack</i>, for obvious reasons.</p>

<p>To circumvent this, another strategy is that, in addition to suppression, <i>perturb</i> the counts (all counts; not just low counts) by adding random noise. The analysts thus sees noisy counts and cannot carry out the differencing attack above. But notice that we can't add completely random noise, or else the output will be junk and useless for legitimate purposes. So, we can decide to add random noise within a fixed interval. Let's assume this is +/-5. Thus, instead of reporting the actual count, we would sample "fresh" noise at random, add to the count and return the <i>noisy count</i> as an entry in the table. The problem with this is that the analyst can create a table multiple times. If fresh noise is added to the count each time then one can launch an <i>averaging attack</i>. Why does it work? Notice that the noise is in the interval +/-5. Since the noise is sampled randomly each time, after a certain number of trials there will be an almost equal distribution of positives and negatives. We can add all the noisy counts, and divide it by the number of times the same table was requested to "average out" the error. An illustration of this is shown in the graph below. The analyst's target is the cell (Redfern, 50-59, Female) whose true count is 19 according to Table <a href="#tabularfakemoredata">T1</a>. The analyst asks the same query 100 times, and the graph shows how the average count converges to the true value. One can find the nearest integer to the fracitonal value obtained at the end (in this case 19.07) to obtain the true count.</p>

<img src="https://hasghar.github.io/assets/images/av-attacks-plot.png" alt="Averaging Attack">

<h2>The TBE Algorithm and Our Attack</h2>
<p>
The failure of the above techniques then led to the "same contributor, same noise" technique used in the TBE algorithm. Under this rule, if two queries (cells in a table) are satisfied by the exact same set of individuals (called contributors), then intead of adding fresh noise, the same noise will be added. This immediately renders the above-mentioned averaging attack useless. However, what we show in our paper is that there is another way to launch an averaging attack. To see this, let's assume that the perturbation parameter in TBE is +/-5. Consider Table <a href="#tabularfakemoredata">T1</a> again. If we ask TBE (via the TableBuilder tool) the count of people aged 30-39 and 40-49, we would get something like this:
</p>

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

<p>
The first thing to note here is that the counts in each cell are noisy. The second thing is the introduction of the row titled "Total." This basically corresponds to the age range 30-49. Importantly, since the contributors in the age range 30-49 are not the same as the contributors in the age ranges 30-39 and 40-49, the noise added to the <i>total cells</i> is different (and hence does not add up)! Note further that this observation holds even if one of the cells 30-39 or 40-49 (under say the Female column) was suppressed (due to having a low non-zero count), since the total would include the other cell count which is not suppressed.  
</p>

<p>
Now suppose that the analyst is interested in knowing the true count under (Redfern, 70-79, Male) from Table <a href="#tabularfakemoredata">T1</a> with access to only the TBE algorithm. The TBE algorithm essentially suppresses this small count and returns 0. The analyst can do the following. Let <b>M</b> be the total number of males in Redfern (i.e., across all ages). The analyst can ask two queries: (10-19, 20-29,...,50-59) and (60-69, 70-79), receiving two tables from the TBE algorithm. Let <b>M1</b> the total number of males in the age range (10-59) and let <b>M2</b> denote the total number of males in the age range (60-79). Through the two tables, the analyst obtains noisy versions of these counts (through the "total" cells). Let's call them <b>M1'</b> and <b>M2'</b>. Note that <b>M</b> = <b>M1</b> + <b>M2</b>. Notice also that <b>M1</b> and <b>M2</b> have different contributors (in fact, mutually exclusive). The analyst records the noisy sum, i.e., <b>M1'</b> + <b>M2'</b>. Let us call this <b>N</b>.
</p>

<p>
The analyst then chooses another "two-partition," e.g., (10-19, 20-29, ..., 40-49) and (50-59, 60-69, 70-79). Once again the sum of the number of males is equal to <b>M</b>. And the analyst receives noisy counts <b>M1''</b> and <b>M2''</b>. Now, since the partitions are different, the contributors are different as well, and hence the noisy counts returned by the TBE algorithm contain fresh independent noise! The analysts adds <b>M1''</b> and <b>M2''</b> to the noisy sum <b>N</b>. Continuing on this way, the analyst obtains noisy counts for all "two-partitions". Each two-partition contains different contributors (unless the true count is 0; the analyst can discard queries whose answer under TBE is 0). For a total of m different attribute values, there are exactly <img src="http://latex.codecogs.com/gif.latex?2^(m-1) - 1" border="0"/>2^{(m-1)} - 1 such two-partitions (as shown in our paper). For our case, we have m = 7, and so 63 different two-partitions. Adding these noisy totals to <b>N</b> for each query, the analyst can finally divide <b>N</b> by the total number of two-partitions queried, and obtain the actual count <b>M</b>. This averaging attack works the same way as the previous simpler averaging attack except now we are carefully asking queries to ensure that we get around the "same contributors, same noise" property of the TBE algorithm.
</p>

<p>
Once the analyst has obtained <b>M</b>, he/she can repeat the same process to obtain the true count under <b>M</b> - (70-79) (by considering two-partitions without 70-79). Finally the true count of (Redfern, 70-79, Male) is obtained as the difference between the two answers. This is more or less the crux of the attack in our paper. I have excluded some technicalities (such as what to do when one of the two-partitions returns a zero count under TBE, and how many queries would ensure that the noise is removed with high confidence). These details are in our paper. For this post, notice that we can apply the same process to find the true count under each age range, and thus obtain the noiseless table for the age distribution of the entire Redfern+Male sub-population in the database. This table can then be converted into unit record format as explained before. Likewise, the attack can be used to "reconstruct" any target sub-population of the dataset.
</p>

<h2>Conclusion</h2>
<p>
So there it is. As you can see, the attack is fairly simple. How to mitigate the attack? We discuss some options in our paper. However, the most sound mitigation strategy is to calibrate noise according to the number of queries asked. Giving away overly accurate answers to too many queries is eventually going to violate privacy. Luckily, we do have a framework that allows us to release noisy answers to queries while simultaneously maintaining privacy. The framework is called <a href="https://en.wikipedia.org/wiki/Differential_privacy">differential privacy</a>, proposed first in this <a href="http://people.csail.mit.edu/asmith/PS/sensitivity-tcc-final.pdf">paper</a>. Differential privacy may not yield good utility for some uses cases, however, the amount of noise added to query answers is, in many cases, close to the lower bound needed to ensure even a <a href="http://www.cse.psu.edu/~ads22/privacy598/papers/dn03.pdf">nominal notion of privacy</a>. For averaging attacks, this means that the amount of noise added is such that the probability of obtaining the true count remains small (and controlled).
</p>
