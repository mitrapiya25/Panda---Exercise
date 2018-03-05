

```
import pandas as pd
import os
import numpy as np
```


```
csv_school_path = "raw_data/schools_complete.csv"
csv_student_path ="raw_data/students_complete.csv"
```


```
school_raw_df = pd.read_csv(csv_school_path)
student_raw_df = pd.read_csv(csv_student_path)
```


```
total_school_count = len(school_raw_df)
total_student_count = len(student_raw_df)
total_budget = school_raw_df["budget"].sum()
Average_math_score = student_raw_df["math_score"].mean()
Average_reading_score = student_raw_df["reading_score"].mean()

#Passing score = 60
pass_score =60
math_pass_count = len(student_raw_df[student_raw_df["math_score"] >= pass_score])
reading_pass_count = len(student_raw_df[student_raw_df["reading_score"] >= pass_score])
math_pass_percent = math_pass_count/total_student_count *100
reading_pass_percent = reading_pass_count/total_student_count*100
average_pass_percent = ((math_pass_percent +reading_pass_percent)/2)

```


```
district_summary = pd.DataFrame({"Total Number of School":[total_school_count],
                               "Total Number of Student":[total_student_count],
                               "Total Budget":[total_budget],
                               "Average Math Score":[Average_math_score],
                               "Average Reading Score":[Average_reading_score],
                               "% Passing Math":[math_pass_percent],
                               "% Passing Reading":[reading_pass_percent],
                               "Overall Passing Rate":[average_pass_percent]})

district_summary
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
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Overall Passing Rate</th>
      <th>Total Budget</th>
      <th>Total Number of School</th>
      <th>Total Number of Student</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>92.445749</td>
      <td>100.0</td>
      <td>78.985371</td>
      <td>81.87784</td>
      <td>96.222875</td>
      <td>24649428</td>
      <td>15</td>
      <td>39170</td>
    </tr>
  </tbody>
</table>
</div>




```
## School Summary
student_raw_df = student_raw_df.rename(columns = {"school" :"school_name"})
school_raw_df = school_raw_df.rename(columns = {"name": "school_name"})
student_school_df = student_raw_df.merge(school_raw_df,on="school_name")
           
```


```
def math_pass_score(row):
    if row["math_score"] >= pass_score:
       status = 1
    else:
       status = 0
    return status    
student_school_df["% Passing Math"] = student_school_df.apply(math_pass_score,axis =1)
student_school_df["% Passing Reading"] = np.where(student_school_df["reading_score"]>=pass_score, 1, 0)

```


```
school_summary_obj = student_school_df.groupby(by='school_name')
school_student = school_summary_obj["Student ID"].count()
school_student_df = pd.DataFrame(school_student)
school_student_df = school_student_df.reset_index()                        

```


```
school_budget_df = school_raw_df[["school_name","type","budget"]]

```


```
school_summary_df = pd.merge(school_budget_df,school_student_df,on="school_name")
school_summary_df = school_summary_df.rename(columns ={"Student ID":"Total Students"})

```


```
school_summary_df["Per student Budget"] = school_summary_df["budget"]/school_summary_df["Total Students"]
school_summary_df = school_summary_df.sort_values("school_name")

```


```
avg_math_per_school = school_summary_obj["math_score"].mean()
math_percent_pass = school_summary_obj["% Passing Math"].sum()
avg_math_per_school_df = pd.DataFrame(avg_math_per_school)
math_percent_pass_df = pd.DataFrame(math_percent_pass)
avg_math_per_school_df = avg_math_per_school_df.reset_index()
math_percent_pass_df =math_percent_pass.reset_index()

avg_reading_per_school = school_summary_obj["reading_score"].mean()
reading_percent_pass = school_summary_obj["% Passing Reading"].sum()
avg_reading_per_school_df = pd.DataFrame(avg_reading_per_school)
reading_percent_pass_df=pd.DataFrame(reading_percent_pass)
avg_reading_per_school_df = avg_reading_per_school_df.reset_index()
reading_percent_pass_df = reading_percent_pass_df.reset_index()
```


```
school_summary_df = pd.merge(school_summary_df,avg_math_per_school_df,on="school_name")

school_summary_df = pd.merge(school_summary_df,avg_reading_per_school_df,on="school_name")

school_summary_df =pd.merge(school_summary_df,math_percent_pass_df,on="school_name")
school_summary_df = pd.merge(school_summary_df,reading_percent_pass_df,on ="school_name")

school_summary_df["% Passing Math"] = (school_summary_df["% Passing Math"]/school_summary_df["Total Students"])*100
school_summary_df["% Passing Reading"] =(school_summary_df["% Passing Reading"]/school_summary_df["Total Students"])*100

school_summary_df["Overall Passing"]=(school_summary_df["% Passing Math"]+school_summary_df["% Passing Reading"])/2

```


```
school_summary_df = school_summary_df.rename(columns = {"school_name":"School Name","reading_score":"Average Reading Score",
                                                        "math_score":"Average Math Score","type":"School Type","budget":"Total School Budget"
                                                    })

```


```
school_summary_df = school_summary_df.sort_values("Overall Passing",ascending=False)

school_summary_df.head(10)
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
      <th>School Name</th>
      <th>School Type</th>
      <th>Total School Budget</th>
      <th>Total Students</th>
      <th>Per student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Cabrera High School</td>
      <td>Charter</td>
      <td>1081356</td>
      <td>1858</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Griffin High School</td>
      <td>Charter</td>
      <td>917500</td>
      <td>1468</td>
      <td>625.0</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Holden High School</td>
      <td>Charter</td>
      <td>248087</td>
      <td>427</td>
      <td>581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>Charter</td>
      <td>585858</td>
      <td>962</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Shelton High School</td>
      <td>Charter</td>
      <td>1056600</td>
      <td>1761</td>
      <td>600.0</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Thomas High School</td>
      <td>Charter</td>
      <td>1043130</td>
      <td>1635</td>
      <td>638.0</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Wilson High School</td>
      <td>Charter</td>
      <td>1319574</td>
      <td>2283</td>
      <td>578.0</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Wright High School</td>
      <td>Charter</td>
      <td>1049400</td>
      <td>1800</td>
      <td>583.0</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Bailey High School</td>
      <td>District</td>
      <td>3124928</td>
      <td>4976</td>
      <td>628.0</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>89.529743</td>
      <td>100.0</td>
      <td>94.764871</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ford High School</td>
      <td>District</td>
      <td>1763916</td>
      <td>2739</td>
      <td>644.0</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>89.302665</td>
      <td>100.0</td>
      <td>94.651333</td>
    </tr>
  </tbody>
</table>
</div>




```
school_summary_top_five_df = school_summary_df.head(5)
school_summary_bottom_five_df = school_summary_df.tail(5)
```


```
school_summary_top_five_df
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
      <th>School Name</th>
      <th>School Type</th>
      <th>Total School Budget</th>
      <th>Total Students</th>
      <th>Per student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Cabrera High School</td>
      <td>Charter</td>
      <td>1081356</td>
      <td>1858</td>
      <td>582.0</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Griffin High School</td>
      <td>Charter</td>
      <td>917500</td>
      <td>1468</td>
      <td>625.0</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Holden High School</td>
      <td>Charter</td>
      <td>248087</td>
      <td>427</td>
      <td>581.0</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>Charter</td>
      <td>585858</td>
      <td>962</td>
      <td>609.0</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Shelton High School</td>
      <td>Charter</td>
      <td>1056600</td>
      <td>1761</td>
      <td>600.0</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>100.0</td>
      <td>100.0</td>
      <td>100.0</td>
    </tr>
  </tbody>
</table>
</div>




```
school_summary_bottom_five_df
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
      <th>School Name</th>
      <th>School Type</th>
      <th>Total School Budget</th>
      <th>Total Students</th>
      <th>Per student Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>8</th>
      <td>Johnson High School</td>
      <td>District</td>
      <td>3094650</td>
      <td>4761</td>
      <td>650.0</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>89.182945</td>
      <td>100.0</td>
      <td>94.591472</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Hernandez High School</td>
      <td>District</td>
      <td>3022020</td>
      <td>4635</td>
      <td>652.0</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>89.083064</td>
      <td>100.0</td>
      <td>94.541532</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Huang High School</td>
      <td>District</td>
      <td>1910635</td>
      <td>2917</td>
      <td>655.0</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>88.858416</td>
      <td>100.0</td>
      <td>94.429208</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Rodriguez High School</td>
      <td>District</td>
      <td>2547363</td>
      <td>3999</td>
      <td>637.0</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>88.547137</td>
      <td>100.0</td>
      <td>94.273568</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Figueroa High School</td>
      <td>District</td>
      <td>1884411</td>
      <td>2949</td>
      <td>639.0</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>88.436758</td>
      <td>100.0</td>
      <td>94.218379</td>
    </tr>
  </tbody>
</table>
</div>




```
school_grade_obj = student_school_df.groupby(by = ["school_name","grade"])

school_grade_math_df = school_grade_obj["math_score"].mean()
school_grade_math_df
```




    school_name            grade
    Bailey High School     10th     76.996772
                           11th     77.515588
                           12th     76.492218
                           9th      77.083676
    Cabrera High School    10th     83.154506
                           11th     82.765560
                           12th     83.277487
                           9th      83.094697
    Figueroa High School   10th     76.539974
                           11th     76.884344
                           12th     77.151369
                           9th      76.403037
    Ford High School       10th     77.672316
                           11th     76.918058
                           12th     76.179963
                           9th      77.361345
    Griffin High School    10th     84.229064
                           11th     83.842105
                           12th     83.356164
                           9th      82.044010
    Hernandez High School  10th     77.337408
                           11th     77.136029
                           12th     77.186567
                           9th      77.438495
    Holden High School     10th     83.429825
                           11th     85.000000
                           12th     82.855422
                           9th      83.787402
    Huang High School      10th     75.908735
                           11th     76.446602
                           12th     77.225641
                           9th      77.027251
    Johnson High School    10th     76.691117
                           11th     77.491653
                           12th     76.863248
                           9th      77.187857
    Pena High School       10th     83.372000
                           11th     84.328125
                           12th     84.121547
                           9th      83.625455
    Rodriguez High School  10th     76.612500
                           11th     76.395626
                           12th     77.690748
                           9th      76.859966
    Shelton High School    10th     82.917411
                           11th     83.383495
                           12th     83.778976
                           9th      83.420755
    Thomas High School     10th     83.087886
                           11th     83.498795
                           12th     83.497041
                           9th      83.590022
    Wilson High School     10th     83.724422
                           11th     83.195326
                           12th     83.035794
                           9th      83.085578
    Wright High School     10th     84.010288
                           11th     83.836782
                           12th     83.644986
                           9th      83.264706
    Name: math_score, dtype: float64




```
school_grade_reading_df = school_grade_obj["reading_score"].mean()
school_grade_reading_df
```




    school_name            grade
    Bailey High School     10th     80.907183
                           11th     80.945643
                           12th     80.912451
                           9th      81.303155
    Cabrera High School    10th     84.253219
                           11th     83.788382
                           12th     84.287958
                           9th      83.676136
    Figueroa High School   10th     81.408912
                           11th     80.640339
                           12th     81.384863
                           9th      81.198598
    Ford High School       10th     81.262712
                           11th     80.403642
                           12th     80.662338
                           9th      80.632653
    Griffin High School    10th     83.706897
                           11th     84.288089
                           12th     84.013699
                           9th      83.369193
    Hernandez High School  10th     80.660147
                           11th     81.396140
                           12th     80.857143
                           9th      80.866860
    Holden High School     10th     83.324561
                           11th     83.815534
                           12th     84.698795
                           9th      83.677165
    Huang High School      10th     81.512386
                           11th     81.417476
                           12th     80.305983
                           9th      81.290284
    Johnson High School    10th     80.773431
                           11th     80.616027
                           12th     81.227564
                           9th      81.260714
    Pena High School       10th     83.612000
                           11th     84.335938
                           12th     84.591160
                           9th      83.807273
    Rodriguez High School  10th     80.629808
                           11th     80.864811
                           12th     80.376426
                           9th      80.993127
    Shelton High School    10th     83.441964
                           11th     84.373786
                           12th     82.781671
                           9th      84.122642
    Thomas High School     10th     84.254157
                           11th     83.585542
                           12th     83.831361
                           9th      83.728850
    Wilson High School     10th     84.021452
                           11th     83.764608
                           12th     84.317673
                           9th      83.939778
    Wright High School     10th     83.812757
                           11th     84.156322
                           12th     84.073171
                           9th      83.833333
    Name: reading_score, dtype: float64




```
bins = [550,580,610,640,670]
group_names = ['Low', 'Medium', 'High', 'Very High']
school_summary_df["Spending"] = pd.cut(school_summary_df["Per student Budget"], bins, labels=group_names)

average_spending_df=school_summary_df[["School Name","Spending","Average Math Score","Average Reading Score","% Passing Math",
                                      "% Passing Reading","Overall Passing"]]
average_spending_df
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
      <th>School Name</th>
      <th>Spending</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Cabrera High School</td>
      <td>Medium</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Griffin High School</td>
      <td>High</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Holden High School</td>
      <td>Medium</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Pena High School</td>
      <td>Medium</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Shelton High School</td>
      <td>Medium</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Thomas High School</td>
      <td>High</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Wilson High School</td>
      <td>Low</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Wright High School</td>
      <td>Medium</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Bailey High School</td>
      <td>High</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>89.529743</td>
      <td>100.0</td>
      <td>94.764871</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ford High School</td>
      <td>Very High</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>89.302665</td>
      <td>100.0</td>
      <td>94.651333</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Johnson High School</td>
      <td>Very High</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>89.182945</td>
      <td>100.0</td>
      <td>94.591472</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Hernandez High School</td>
      <td>Very High</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>89.083064</td>
      <td>100.0</td>
      <td>94.541532</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Huang High School</td>
      <td>Very High</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>88.858416</td>
      <td>100.0</td>
      <td>94.429208</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Rodriguez High School</td>
      <td>High</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>88.547137</td>
      <td>100.0</td>
      <td>94.273568</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Figueroa High School</td>
      <td>High</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>88.436758</td>
      <td>100.0</td>
      <td>94.218379</td>
    </tr>
  </tbody>
</table>
</div>




```
def total_math_score(row):
    school = row["School Name"]
    value = student_school_df.loc[student_school_df['school_name'] == school, 'math_score'].sum()
    return value 

school_summary_df["Total Math Score"] = school_summary_df.apply(total_math_score,axis =1)

def total_reading_score(row):
    school = row["School Name"]
    value = student_school_df.loc[student_school_df['school_name'] == school, 'reading_score'].sum()
    return value 

school_summary_df["Total Reading Score"] = school_summary_df.apply(total_reading_score,axis =1)

```


```
spending_grp_obj = school_summary_df.groupby(by ="Spending")
spending_grp_math = (spending_grp_obj["Total Math Score"].sum())/(spending_grp_obj["Total Students"].sum())
spending_grp_math_df = pd.DataFrame(spending_grp_math)
spending_grp_math_df = spending_grp_math_df.reset_index()

spending_grp_reading = (spending_grp_obj["Total Reading Score"].sum())/(spending_grp_obj["Total Students"].sum())
spending_grp_reading_df = pd.DataFrame(spending_grp_reading)
spending_grp_reading_df = spending_grp_reading_df.reset_index()

grouped_spending_df = pd.merge(spending_grp_math_df,spending_grp_reading_df, on="Spending")
grouped_spending_df = grouped_spending_df.rename(columns ={"0_x":"Average Math Score"})
grouped_spending_df = grouped_spending_df.rename(columns ={"0_y":"Average Reading Score"})

spending_percent_reading = spending_grp_obj["% Passing Reading"].mean()
spending_percent_math = spending_grp_obj["% Passing Math"].mean()
spending_percent_math_df = pd.DataFrame(spending_percent_math)
spending_percent_math_df = spending_percent_math_df.reset_index()
spending_percent_reading_df = pd.DataFrame(spending_percent_reading)
spending_percent_reading_df = spending_percent_reading_df.reset_index()

grouped_spending_df = grouped_spending_df.merge(spending_percent_math_df,on="Spending")
grouped_spending_df = grouped_spending_df.merge(spending_percent_reading_df,on="Spending")


grouped_spending_df["Overall Rating"] = (grouped_spending_df["% Passing Math"] + grouped_spending_df["% Passing Reading"])/2
grouped_spending_df
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
      <th>Spending</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Low</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Medium</td>
      <td>83.459313</td>
      <td>83.905259</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>High</td>
      <td>78.236441</td>
      <td>81.559460</td>
      <td>93.302728</td>
      <td>100.0</td>
      <td>96.651364</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Very High</td>
      <td>77.058995</td>
      <td>80.958411</td>
      <td>89.106772</td>
      <td>100.0</td>
      <td>94.553386</td>
    </tr>
  </tbody>
</table>
</div>




```
bins = [0,2000,3500,5000]
group_names = ['Small','Medium','Large']
school_summary_df["Size"] = pd.cut(school_summary_df["Total Students"], bins, labels=group_names)

```


```
size_group_obj = school_summary_df.groupby(by ="Size")

size_group_math = size_group_obj["Total Math Score"].sum()/size_group_obj["Total Students"].sum()
size_group_math_df = pd.DataFrame(size_group_math)
size_group_math_df = size_group_math_df.reset_index()


size_group_reading = size_group_obj["Total Reading Score"].sum()/size_group_obj["Total Students"].sum()
size_group_reading_df = pd.DataFrame(size_group_reading)
size_group_reading_df = size_group_reading_df.reset_index()

size_group_df = pd.merge(size_group_math_df,size_group_reading_df,on="Size")
size_group_df =size_group_df.rename(columns = {"0_x":"Average Math Score","0_y":"Average Reading Score"})

```


```
size_group_percent_math = size_group_obj["% Passing Math"].mean()
size_group_percent_math_df = pd.DataFrame(size_group_percent_math)
size_group_percent_math_df = size_group_percent_math_df.reset_index()

size_group_percent_reading = size_group_obj["% Passing Reading"].mean()
size_group_percent_reading_df = pd.DataFrame(size_group_percent_reading)
size_group_percent_reading_df = size_group_percent_reading_df.reset_index()

size_group_df = size_group_df.merge(size_group_percent_math_df,on= "Size")
size_group_df = size_group_df.merge(size_group_percent_reading_df,on= "Size")

size_group_df["Overall Rating"] =(size_group_df["% Passing Reading"] + size_group_df["% Passing Math"])/2
```


```
size_group_df
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
      <th>Size</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Small</td>
      <td>83.436586</td>
      <td>83.882857</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Medium</td>
      <td>78.164034</td>
      <td>81.654758</td>
      <td>91.649460</td>
      <td>100.0</td>
      <td>95.824730</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Large</td>
      <td>77.070764</td>
      <td>80.928365</td>
      <td>89.085722</td>
      <td>100.0</td>
      <td>94.542861</td>
    </tr>
  </tbody>
</table>
</div>




```
type_group_obj = school_summary_df.groupby(by ="School Type")

type_group_math = type_group_obj["Total Math Score"].sum()/type_group_obj["Total Students"].sum()
type_group_math_df = pd.DataFrame(type_group_math)
type_group_math_df = type_group_math_df.reset_index()


type_group_reading = type_group_obj["Total Reading Score"].sum()/type_group_obj["Total Students"].sum()
type_group_reading_df = pd.DataFrame(type_group_reading)
type_group_reading_df = type_group_reading_df.reset_index()

type_group_df = pd.merge(type_group_math_df,type_group_reading_df,on="School Type")
type_group_df =type_group_df.rename(columns = {"0_x":"Average Math Score","0_y":"Average Reading Score"})
```


```
type_group_percent_math = type_group_obj["% Passing Math"].mean()
type_group_percent_math_df = pd.DataFrame(type_group_percent_math)
type_group_percent_math_df = type_group_percent_math_df.reset_index()

type_group_percent_reading = type_group_obj["% Passing Reading"].mean()
type_group_percent_reading_df = pd.DataFrame(type_group_percent_reading)
type_group_percent_reading_df = type_group_percent_reading_df.reset_index()

type_group_df = type_group_df.merge(type_group_percent_math_df,on= "School Type")
type_group_df = type_group_df.merge(type_group_percent_reading_df,on= "School Type")

type_group_df["Overall Rating"] =(type_group_df["% Passing Reading"] + type_group_df["% Passing Math"])/2

type_group_df
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
      <th>School Type</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Charter</td>
      <td>83.406183</td>
      <td>83.902821</td>
      <td>100.000000</td>
      <td>100.0</td>
      <td>100.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>District</td>
      <td>76.987026</td>
      <td>80.962485</td>
      <td>88.991533</td>
      <td>100.0</td>
      <td>94.495766</td>
    </tr>
  </tbody>
</table>
</div>


