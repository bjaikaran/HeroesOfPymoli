

```python
#Imports and other functions required to run this file
import pandas as pd
import numpy as np
import math as math


file_import = "purchase_data.json"
nerds = pd.read_json(path_or_buf=file_import, orient=None, typ='frame', dtype=True, convert_axes=True, convert_dates=True)
#nerds.to_excel("nerds.xlsx", index=False)
#nerds.head()
```


```python
#Count of unique Screen Names
players_df= len(nerds["SN"].unique())

count_players= pd.DataFrame([{"Total Players":players_df}])
count_players.set_index('Total Players', inplace = True)
count_players
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
    </tr>
    <tr>
      <th>Total Players</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>573</th>
    </tr>
  </tbody>
</table>
</div>




```python
# Pricing Analysis
item_name= nerds.drop_duplicates(["Item ID"], keep = "last")
item_name = len(item_name)
avg_price= nerds["Price"].mean()
avg_price_d= "${:,.2f}".format(avg_price)
total_rev= nerds["Price"].sum()
total_rev_d= "${:,.2f}".format(total_rev)
item_id= nerds["Item ID"].count()
p_columns= ["Number of Unique Items", "Average Price", "Number of Purchases", "Total Revenue"]
pricing={"Number of Unique Items":[item_name],
         "Average Price":[avg_price_d],
         "Total Revenue": [total_rev_d],
        "Number of Purchases":[item_id]}
pricing=pd.DataFrame.from_dict(pricing)
pricing=pricing.reindex(columns=p_columns)

pricing
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Number of Unique Items</th>
      <th>Average Price</th>
      <th>Number of Purchases</th>
      <th>Total Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>183</td>
      <td>$2.93</td>
      <td>780</td>
      <td>$2,286.33</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Gender Demographics
g_data = nerds.drop_duplicates(["SN"], keep="last")
g_counts =g_data["Gender"].value_counts().reset_index()
g_counts["Percent of Players"]=(g_counts["Gender"]/g_counts["Gender"].sum())*100
g_counts.rename(columns={"index":"Gender", "Gender":"Total Players"},inplace=True)
g_counts.set_index(["Gender"], inplace = True)
g_c_format = g_counts
g_c_format.style.format({"Percent of Players": "{:.2f}%"})
```




<style  type="text/css" >
</style>  
<table id="T_c48f99bc_2b7c_11e8_8209_0a580a0c01cc" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Total Players</th> 
        <th class="col_heading level0 col1" >Percent of Players</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Gender</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_c48f99bc_2b7c_11e8_8209_0a580a0c01cclevel0_row0" class="row_heading level0 row0" >Male</th> 
        <td id="T_c48f99bc_2b7c_11e8_8209_0a580a0c01ccrow0_col0" class="data row0 col0" >465</td> 
        <td id="T_c48f99bc_2b7c_11e8_8209_0a580a0c01ccrow0_col1" class="data row0 col1" >81.15%</td> 
    </tr>    <tr> 
        <th id="T_c48f99bc_2b7c_11e8_8209_0a580a0c01cclevel0_row1" class="row_heading level0 row1" >Female</th> 
        <td id="T_c48f99bc_2b7c_11e8_8209_0a580a0c01ccrow1_col0" class="data row1 col0" >100</td> 
        <td id="T_c48f99bc_2b7c_11e8_8209_0a580a0c01ccrow1_col1" class="data row1 col1" >17.45%</td> 
    </tr>    <tr> 
        <th id="T_c48f99bc_2b7c_11e8_8209_0a580a0c01cclevel0_row2" class="row_heading level0 row2" >Other / Non-Disclosed</th> 
        <td id="T_c48f99bc_2b7c_11e8_8209_0a580a0c01ccrow2_col0" class="data row2 col0" >8</td> 
        <td id="T_c48f99bc_2b7c_11e8_8209_0a580a0c01ccrow2_col1" class="data row2 col1" >1.40%</td> 
    </tr></tbody> 
</table> 




```python
#Purchasing Analysis by Gender
p_c_gender= pd.DataFrame(nerds.groupby("Gender")["Gender"].count())
p_p_gender= pd.DataFrame(nerds.groupby("Gender")["Price"].sum())
g_nerds = pd.merge(p_c_gender,p_p_gender, left_index=True, right_index=True)
g_nerds = g_nerds.merge(g_counts, left_index=True, right_index=True)
g_nerds["Average Price"]= g_nerds["Price"]/g_nerds["Gender"]
g_nerds["Normalized Price"]= g_nerds["Price"]/g_nerds["Total Players"]
g_nerds.rename(columns={"Gender": "Purchase Count","Price":"Total Purchase Value"}, inplace= True)
del g_nerds["Total Players"]
del g_nerds["Percent of Players"]

g_nerds
g_n_format = g_nerds
g_n_format.style.format({"Average Price": "${:.2f}", "Total Purchase Value": "${:,.2f}", "Normalized Price":"${:.2f}"})
```




<style  type="text/css" >
</style>  
<table id="T_c61387d0_2b7c_11e8_8209_0a580a0c01cc" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Purchase Count</th> 
        <th class="col_heading level0 col1" >Total Purchase Value</th> 
        <th class="col_heading level0 col2" >Average Price</th> 
        <th class="col_heading level0 col3" >Normalized Price</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Gender</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_c61387d0_2b7c_11e8_8209_0a580a0c01cclevel0_row0" class="row_heading level0 row0" >Female</th> 
        <td id="T_c61387d0_2b7c_11e8_8209_0a580a0c01ccrow0_col0" class="data row0 col0" >136</td> 
        <td id="T_c61387d0_2b7c_11e8_8209_0a580a0c01ccrow0_col1" class="data row0 col1" >$382.91</td> 
        <td id="T_c61387d0_2b7c_11e8_8209_0a580a0c01ccrow0_col2" class="data row0 col2" >$2.82</td> 
        <td id="T_c61387d0_2b7c_11e8_8209_0a580a0c01ccrow0_col3" class="data row0 col3" >$3.83</td> 
    </tr>    <tr> 
        <th id="T_c61387d0_2b7c_11e8_8209_0a580a0c01cclevel0_row1" class="row_heading level0 row1" >Male</th> 
        <td id="T_c61387d0_2b7c_11e8_8209_0a580a0c01ccrow1_col0" class="data row1 col0" >633</td> 
        <td id="T_c61387d0_2b7c_11e8_8209_0a580a0c01ccrow1_col1" class="data row1 col1" >$1,867.68</td> 
        <td id="T_c61387d0_2b7c_11e8_8209_0a580a0c01ccrow1_col2" class="data row1 col2" >$2.95</td> 
        <td id="T_c61387d0_2b7c_11e8_8209_0a580a0c01ccrow1_col3" class="data row1 col3" >$4.02</td> 
    </tr>    <tr> 
        <th id="T_c61387d0_2b7c_11e8_8209_0a580a0c01cclevel0_row2" class="row_heading level0 row2" >Other / Non-Disclosed</th> 
        <td id="T_c61387d0_2b7c_11e8_8209_0a580a0c01ccrow2_col0" class="data row2 col0" >11</td> 
        <td id="T_c61387d0_2b7c_11e8_8209_0a580a0c01ccrow2_col1" class="data row2 col1" >$35.74</td> 
        <td id="T_c61387d0_2b7c_11e8_8209_0a580a0c01ccrow2_col2" class="data row2 col2" >$3.25</td> 
        <td id="T_c61387d0_2b7c_11e8_8209_0a580a0c01ccrow2_col3" class="data row2 col3" >$4.47</td> 
    </tr></tbody> 
</table> 




```python
#Age Demographics
nerds.loc[(nerds["Age"] < 10), "Age Ranges"] = "<10"
nerds.loc[(nerds["Age"] >= 10) & (nerds["Age"] <= 14), "Age Ranges"] = "10 - 14"
nerds.loc[(nerds["Age"] >= 15) & (nerds["Age"] <= 19), "Age Ranges"] = "15 - 19"
nerds.loc[(nerds["Age"] >= 20) & (nerds["Age"] <= 24), "Age Ranges"] = "20 - 24"
nerds.loc[(nerds["Age"] >= 25) & (nerds["Age"] <= 29), "Age Ranges"] = "25 - 29"
nerds.loc[(nerds["Age"] >= 30) & (nerds["Age"] <= 34), "Age Ranges"] = "30 - 34"
nerds.loc[(nerds["Age"] >= 35) & (nerds["Age"] <= 39), "Age Ranges"] = "35 - 39"
nerds.loc[(nerds["Age"] >= 40), "Age Ranges"] = "> 40"

age_demo = nerds.drop_duplicates(["SN"], keep="last")
age_b = pd.DataFrame(age_demo.groupby("Age Ranges")["SN"].count())
age_b["Percentage of Players"] = (age_b["SN"]/age_b["SN"].sum())*100

age_b.rename(columns={"SN":"Total Count"},inplace=True)
age_b_format = age_b
age_b_format.style.format({"Percentage of Players": "{:.2f}%"})
```




<style  type="text/css" >
</style>  
<table id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01cc" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Total Count</th> 
        <th class="col_heading level0 col1" >Percentage of Players</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Age Ranges</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01cclevel0_row0" class="row_heading level0 row0" >10 - 14</th> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow0_col0" class="data row0 col0" >23</td> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow0_col1" class="data row0 col1" >4.01%</td> 
    </tr>    <tr> 
        <th id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01cclevel0_row1" class="row_heading level0 row1" >15 - 19</th> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow1_col0" class="data row1 col0" >100</td> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow1_col1" class="data row1 col1" >17.45%</td> 
    </tr>    <tr> 
        <th id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01cclevel0_row2" class="row_heading level0 row2" >20 - 24</th> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow2_col0" class="data row2 col0" >259</td> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow2_col1" class="data row2 col1" >45.20%</td> 
    </tr>    <tr> 
        <th id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01cclevel0_row3" class="row_heading level0 row3" >25 - 29</th> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow3_col0" class="data row3 col0" >87</td> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow3_col1" class="data row3 col1" >15.18%</td> 
    </tr>    <tr> 
        <th id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01cclevel0_row4" class="row_heading level0 row4" >30 - 34</th> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow4_col0" class="data row4 col0" >47</td> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow4_col1" class="data row4 col1" >8.20%</td> 
    </tr>    <tr> 
        <th id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01cclevel0_row5" class="row_heading level0 row5" >35 - 39</th> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow5_col0" class="data row5 col0" >27</td> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow5_col1" class="data row5 col1" >4.71%</td> 
    </tr>    <tr> 
        <th id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01cclevel0_row6" class="row_heading level0 row6" ><10</th> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow6_col0" class="data row6 col0" >19</td> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow6_col1" class="data row6 col1" >3.32%</td> 
    </tr>    <tr> 
        <th id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01cclevel0_row7" class="row_heading level0 row7" >> 40</th> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow7_col0" class="data row7 col0" >11</td> 
        <td id="T_d63ffef4_2b7c_11e8_8209_0a580a0c01ccrow7_col1" class="data row7 col1" >1.92%</td> 
    </tr></tbody> 
</table> 




```python
#Age Purchases
age_items = pd.DataFrame(nerds.groupby("Age Ranges")["Item ID"].count())
age_purchases = pd.DataFrame(nerds.groupby("Age Ranges")["Price"].sum())
age_p_a= pd.merge(age_purchases,age_items, left_index=True, right_index=True)

age_uniq= pd.merge(age_p_a,age_b,left_index=True,right_index=True)
purch_total=age_p_a["Item ID"].sum()
del age_uniq["Percentage of Players"]
age_uniq.rename(columns={"Price":"Total Purchase Value","Item ID":"Purchase Count"},inplace=True)
age_uniq
age_uniq["Average Price"]= age_uniq["Total Purchase Value"]/age_uniq["Purchase Count"]
age_uniq["Normalized Avg"]= age_uniq["Total Purchase Value"]/age_uniq["Total Count"]
del age_uniq["Total Count"]
age_uniq_format= age_uniq
age_uniq_format.style.format({"Average Price": "${:.2f}", "Total Purchase Value": "${:,.2f}", "Normalized Avg":"${:.2f}"})


```




<style  type="text/css" >
</style>  
<table id="T_bf08640a_2b78_11e8_8209_0a580a0c01cc" > 
<thead>    <tr> 
        <th class="blank level0" ></th> 
        <th class="col_heading level0 col0" >Total Purchase Value</th> 
        <th class="col_heading level0 col1" >Purchase Count</th> 
        <th class="col_heading level0 col2" >Average Price</th> 
        <th class="col_heading level0 col3" >Normalized Avg</th> 
    </tr>    <tr> 
        <th class="index_name level0" >Age Ranges</th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
        <th class="blank" ></th> 
    </tr></thead> 
<tbody>    <tr> 
        <th id="T_bf08640a_2b78_11e8_8209_0a580a0c01cclevel0_row0" class="row_heading level0 row0" >10 - 14</th> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow0_col0" class="data row0 col0" >$96.95</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow0_col1" class="data row0 col1" >35</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow0_col2" class="data row0 col2" >$2.77</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow0_col3" class="data row0 col3" >$4.22</td> 
    </tr>    <tr> 
        <th id="T_bf08640a_2b78_11e8_8209_0a580a0c01cclevel0_row1" class="row_heading level0 row1" >15 - 19</th> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow1_col0" class="data row1 col0" >$386.42</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow1_col1" class="data row1 col1" >133</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow1_col2" class="data row1 col2" >$2.91</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow1_col3" class="data row1 col3" >$3.86</td> 
    </tr>    <tr> 
        <th id="T_bf08640a_2b78_11e8_8209_0a580a0c01cclevel0_row2" class="row_heading level0 row2" >20 - 24</th> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow2_col0" class="data row2 col0" >$978.77</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow2_col1" class="data row2 col1" >336</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow2_col2" class="data row2 col2" >$2.91</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow2_col3" class="data row2 col3" >$3.78</td> 
    </tr>    <tr> 
        <th id="T_bf08640a_2b78_11e8_8209_0a580a0c01cclevel0_row3" class="row_heading level0 row3" >25 - 29</th> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow3_col0" class="data row3 col0" >$370.33</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow3_col1" class="data row3 col1" >125</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow3_col2" class="data row3 col2" >$2.96</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow3_col3" class="data row3 col3" >$4.26</td> 
    </tr>    <tr> 
        <th id="T_bf08640a_2b78_11e8_8209_0a580a0c01cclevel0_row4" class="row_heading level0 row4" >30 - 34</th> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow4_col0" class="data row4 col0" >$197.25</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow4_col1" class="data row4 col1" >64</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow4_col2" class="data row4 col2" >$3.08</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow4_col3" class="data row4 col3" >$4.20</td> 
    </tr>    <tr> 
        <th id="T_bf08640a_2b78_11e8_8209_0a580a0c01cclevel0_row5" class="row_heading level0 row5" >35 - 39</th> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow5_col0" class="data row5 col0" >$119.40</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow5_col1" class="data row5 col1" >42</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow5_col2" class="data row5 col2" >$2.84</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow5_col3" class="data row5 col3" >$4.42</td> 
    </tr>    <tr> 
        <th id="T_bf08640a_2b78_11e8_8209_0a580a0c01cclevel0_row6" class="row_heading level0 row6" >< 10</th> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow6_col0" class="data row6 col0" >$83.46</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow6_col1" class="data row6 col1" >28</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow6_col2" class="data row6 col2" >$2.98</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow6_col3" class="data row6 col3" >$4.39</td> 
    </tr>    <tr> 
        <th id="T_bf08640a_2b78_11e8_8209_0a580a0c01cclevel0_row7" class="row_heading level0 row7" >> 40</th> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow7_col0" class="data row7 col0" >$53.75</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow7_col1" class="data row7 col1" >17</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow7_col2" class="data row7 col2" >$3.16</td> 
        <td id="T_bf08640a_2b78_11e8_8209_0a580a0c01ccrow7_col3" class="data row7 col3" >$4.89</td> 
    </tr></tbody> 
</table> 




```python
#Top Spenders
top_spend = pd.DataFrame(nerds.groupby("SN")["Item ID"].count())
top_spd_price = pd.DataFrame(nerds.groupby("SN")["Price"].sum())
spd_df=pd.merge(top_spend,top_spd_price, left_index=True, right_index=True)

spd_df.rename(columns={"Item ID":"Purchase Count","Price":"Total Purchase Value"}, inplace=True)

spd_df.sort_values("Total Purchase Value",ascending=False, inplace=True)
spd_df["Average Purchase Price"] = spd_df["Total Purchase Value"]/spd_df["Purchase Count"]
spd_df_format =spd_df
spd_df_format.style.format({"Purchase Count":"{:,0f}","Total Purchase Value": "${:,.2f}","Average Purchase Price": "${:.2f}"})
spd_df_format.head()


```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Purchase Count</th>
      <th>Total Purchase Value</th>
      <th>Average Purchase Price</th>
    </tr>
    <tr>
      <th>SN</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Undirrala66</th>
      <td>5</td>
      <td>17.06</td>
      <td>3.412000</td>
    </tr>
    <tr>
      <th>Saedue76</th>
      <td>4</td>
      <td>13.56</td>
      <td>3.390000</td>
    </tr>
    <tr>
      <th>Mindimnya67</th>
      <td>4</td>
      <td>12.74</td>
      <td>3.185000</td>
    </tr>
    <tr>
      <th>Haellysu29</th>
      <td>3</td>
      <td>12.73</td>
      <td>4.243333</td>
    </tr>
    <tr>
      <th>Eoda93</th>
      <td>3</td>
      <td>11.58</td>
      <td>3.860000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Top Items
top_item = pd.DataFrame(nerds.groupby("Item Name")["Item ID"].count())
top_item_price = pd.DataFrame(nerds.groupby("Item Name")["Price"].sum())
top_df=pd.merge(top_item,top_item_price, left_index=True, right_index=True)
top_df.sort_values("Item ID",ascending=False, inplace=True)
top_df["Average Price"]= top_df["Price"]/top_df["Item ID"]
top_df["Price"]
top_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Item ID</th>
      <th>Price</th>
      <th>Average Price</th>
    </tr>
    <tr>
      <th>Item Name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Final Critic</th>
      <td>14</td>
      <td>38.60</td>
      <td>2.757143</td>
    </tr>
    <tr>
      <th>Arcane Gem</th>
      <td>11</td>
      <td>24.53</td>
      <td>2.230000</td>
    </tr>
    <tr>
      <th>Betrayal, Whisper of Grieving Widows</th>
      <td>11</td>
      <td>25.85</td>
      <td>2.350000</td>
    </tr>
    <tr>
      <th>Stormcaller</th>
      <td>10</td>
      <td>34.65</td>
      <td>3.465000</td>
    </tr>
    <tr>
      <th>Woeful Adamantite Claymore</th>
      <td>9</td>
      <td>11.16</td>
      <td>1.240000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Most profitable items
top_item = pd.DataFrame(nerds.groupby("Item Name")["Item ID"].count())
top_item_price = pd.DataFrame(nerds.groupby("Item Name")["Price"].sum())
top_df=pd.merge(top_item,top_item_price, left_index=True, right_index=True)

top_df["Average Price"]= top_df["Price"]/top_df["Item ID"]
top_df["Price"]
top_df.sort_values("Average Price",ascending=False, inplace=True)
top_df.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Item ID</th>
      <th>Price</th>
      <th>Average Price</th>
    </tr>
    <tr>
      <th>Item Name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Orenmir</th>
      <td>6</td>
      <td>29.70</td>
      <td>4.95</td>
    </tr>
    <tr>
      <th>Winterthorn, Defender of Shifting Worlds</th>
      <td>4</td>
      <td>19.56</td>
      <td>4.89</td>
    </tr>
    <tr>
      <th>Singed Scalpel</th>
      <td>6</td>
      <td>29.22</td>
      <td>4.87</td>
    </tr>
    <tr>
      <th>Stormfury Longsword</th>
      <td>5</td>
      <td>24.15</td>
      <td>4.83</td>
    </tr>
    <tr>
      <th>The Decapitator</th>
      <td>3</td>
      <td>14.46</td>
      <td>4.82</td>
    </tr>
  </tbody>
</table>
</div>


