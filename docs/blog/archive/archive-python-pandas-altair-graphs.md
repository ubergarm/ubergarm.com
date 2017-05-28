Finally, an easy way to turn Pandas DataFrames into sorted bar charts using brewer color schemes!

It took a little playing around and it isn't quite perfect yet, but here is a quick example of how to turn a Series or DataFrame into a lovely sorted stacked bar chart.


#### References

* [altair](https://github.com/ellisonbg/altair)
* [palettable](https://jiffyclub.github.io/palettable/)
* [jupyter notebook](https://github.com/jupyter/docker-stacks/tree/master/datascience-notebook)

#### Run Jupyter Container
Running as root allows you to `!pip install` or `!apt-get install` dependencies.

```bash
#!/bin/bash
docker run --name jupyter -d -p 8888:8888 -v `pwd`/notebooks:/home/jovyan/work --user=root -e GRANT_SUDO=yes -e NB_UID=$UID jupyter/datascience-notebook
```

#### Example Notebook

```python
!pip install --upgrade pip
!pip install git+https://github.com/ellisonbg/altair
!pip install --upgrade notebook  # need jupyter_client >= 4.2 for sys-prefix below
!jupyter nbextension install --sys-prefix --py vega
!pip install palettable # for brewer color maps
```


```python
import pandas as pd
import numpy as np
```


```python
# pretty graphs https://github.com/ellisonbg/altair
from altair import *
```


```python
# excellent brewer color palettes https://jiffyclub.github.io/palettable/
from palettable import colorbrewer as cb
```


```python
data = pd.DataFrame()
data['counts'] = pd.Series(np.round(100 * np.abs(np.random.randn(8))))
data.index = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'i']
data['title'] = data.index
data
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>counts</th>
      <th>title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>a</th>
      <td>34</td>
      <td>a</td>
    </tr>
    <tr>
      <th>b</th>
      <td>49</td>
      <td>b</td>
    </tr>
    <tr>
      <th>c</th>
      <td>77</td>
      <td>c</td>
    </tr>
    <tr>
      <th>d</th>
      <td>111</td>
      <td>d</td>
    </tr>
    <tr>
      <th>e</th>
      <td>3</td>
      <td>e</td>
    </tr>
    <tr>
      <th>f</th>
      <td>36</td>
      <td>f</td>
    </tr>
    <tr>
      <th>g</th>
      <td>156</td>
      <td>g</td>
    </tr>
    <tr>
      <th>i</th>
      <td>15</td>
      <td>i</td>
    </tr>
  </tbody>
</table>
</div>




```python
Chart(data).mark_bar().encode(
    x='counts',
    y='title'
)
```


<div class="vega-embed" id="b5a28374-5210-4033-8eb0-d9d7151e269c"></div>

<style>
.vega-embed svg, .vega-embed canvas {
  border: 1px dotted gray;
}

.vega-embed .vega-actions a {
  margin-right: 6px;
}
</style>






![png](/content/images/2016/08/output_5_2-1.png)



```python
c = Chart(data).mark_bar().encode(
    x=X('counts', axis=Axis(title='Counts')),
    y=Y('title', sort=SortField(field='counts', order='descending', op='sum'), axis=Axis(title='Title')),
    color=Color('title:N', scale=Scale(range=cb.qualitative.Dark2_8.hex_colors))
)
#c.to_dict()
c
```


<div class="vega-embed" id="ac7e4fe7-17a5-4fb2-b728-5c649f4a2609"></div>

<style>
.vega-embed svg, .vega-embed canvas {
  border: 1px dotted gray;
}

.vega-embed .vega-actions a {
  margin-right: 6px;
}
</style>






![png](/content/images/2016/08/output_6_2-1.png)
