---
layout: post
title: Node-TimSort Performance with Node.js v4
description: Benchmark results for Node-TimSort module run on top of Node.js v4.1.1
keywords: benchmark, Node.js, sort, compare, TimSort, algorithm, performance
---
Node-TimSort is a Javascript implementation of the 
[TimSort](http://svn.python.org/projects/python/trunk/Objects/listsort.txt) algorithm developed 
by Tim Peters, that showed to be incredibly fast on top of Node.js v0.12.7 (article 
[here](/2015/08/10/node-timsort-fast-sorting-nodejs/)). Node-TimSort is avalaible on 
[Github](https://github.com/mziccard/node-timsort), [npm](https://www.npmjs.com/package/timsort) and bower.

Given the (not so) recent update of Node.js to version 4 I decided to benchmark the module against Node's 
latest release (v4.1.1). Results follow in table:

<table>
  <tr>
    <th></th><th></th>
    <th colspan="2">Execution Time (ns)</th>
    <th rowspan="2">Speedup</th>
  </tr>
  <tr>
    <th>Array Type</th>
    <th>Length</th>
    <th>TimSort.sort</th>
    <th>array.sort</th>
  </tr>
<tbody>
 <tr>
  <td>Array</td>
  <td>Length</td>
  <td>TimSort</td>
  <td>array.sort</td>
  <td>Speedup</td>
 </tr>
 <tr>
  <td rowspan="4">Random</td>
  <td>10</td><td>1529</td><td>4804</td><td>3.14</td>
 </tr>
 <tr>
  <td>100</td><td>16091</td><td>56875</td><td>3.53</td>
 </tr>
 <tr>
  <td>1000</td><td>199985</td><td>704214</td><td>3.52</td>
 </tr>
 <tr>
  <td>10000</td><td>2528060</td><td>9125651</td><td>3.61</td>
 </tr>
 <tr>
  <td rowspan="4">Descending</td>
  <td>10</td><td>1092</td><td>3680</td><td>3.37</td>
 </tr>
 <tr>
  <td>100</td><td>2503</td><td>31799</td><td>12.70</td>
 </tr>
 <tr>
  <td>1000</td><td>11821</td><td>543912</td><td>46.01</td>
 </tr>
 <tr>
  <td>10000</td><td>98039</td><td>7768847</td><td>79.24</td>
 </tr>
 <tr>
  <td rowspan="4">Ascending</td>
  <td>10</td><td>1030</td><td>2062</td><td>2.00</td>
 </tr>
 <tr>
  <td>100</td><td>2195</td><td>30635</td><td>13.95</td>
 </tr>
 <tr>
  <td>1000</td><td>8715</td><td>502126</td><td>57.61</td>
 </tr>
 <tr>
  <td>10000</td><td>72685</td><td>7581941</td><td>104.31</td>
 </tr>
 <tr>
  <td rowspan="4">Ascending + 3 Rand Exc</td>
  <td>10</td><td>1489</td><td>2503</td><td>1.68</td>
 </tr>
 <tr>
  <td>100</td><td>4064</td><td>31230</td><td>7.68</td>
 </tr>
 <tr>
  <td>1000</td><td>13647</td><td>515358</td><td>37.76</td>
 </tr>
 <tr>
  <td>10000</td><td>106676</td><td>7549566</td><td>70.77</td>
 </tr>
 <tr>
  <td rowspan="4">Ascending + 10 Rand End</td>
  <td>10</td><td>1543</td><td>3162</td><td>2.05</td>
 </tr>
 <tr>
  <td>100</td><td>6596</td><td>34657</td><td>5.25</td>
 </tr>
 <tr>
  <td>1000</td><td>22121</td><td>501595</td><td>22.67</td>
 </tr>
 <tr>
  <td>10000</td><td>127955</td><td>7240459</td><td>56.59</td>
 </tr>
 <tr>
  <td rowspan="4">Equal Elements</td>
  <td>10</td><td>1064</td><td>2172</td><td>2.04</td>
 </tr>
 <tr>
  <td>100</td><td>2089</td><td>6645</td><td>3.18</td>
 </tr>
 <tr>
  <td>1000</td><td>7640</td><td>42421</td><td>5.55</td>
 </tr>
 <tr>
  <td>10000</td><td>59737</td><td>392580</td><td>6.57</td>
 </tr>
 <tr>
  <td rowspan="4">Many Repetitions</td>
  <td>10</td><td>1501</td><td>3183</td><td>2.12</td>
 </tr>
 <tr>
  <td>100</td><td>18723</td><td>37164</td><td>1.98</td>
 </tr>
 <tr>
  <td>1000</td><td>252043</td><td>559894</td><td>2.22</td>
 </tr>
 <tr>
  <td>10000</td><td>3224430</td><td>7672401</td><td>2.38</td>
 </tr>
 <tr>
  <td rowspan="4">Some Repetitions</td>
  <td>10</td><td>1578</td><td>3260</td><td>2.07</td>
 </tr>
 <tr>
  <td>100</td><td>18486</td><td>36732</td><td>1.99</td>
 </tr>
 <tr>
  <td>1000</td><td>248599</td><td>544063</td><td>2.19</td>
 </tr>
 <tr>
  <td>10000</td><td>3272074</td><td>7590367</td><td>2.32</td>
 </tr>
</tbody>
</table>

Even with version 4.1.1 `TimSort.sort` **is faster** than `array.sort` on any of the tested array types. 
In general, the more ordered the array is the better `TimSort.sort` performs with respect to `array.sort` 
(up to 100 times faster on already sorted arrays).  

Once again, the data also depend on the machine on which the benchmark is run. 
I strongly encourage you to clone the [repository](https://github.com/mziccard/node-timsort) 
and run the benchmark on your own setup with:

```
npm run benchmark
```
