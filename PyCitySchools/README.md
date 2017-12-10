

```python
import pandas as pd
import numpy as np
```


```python
# load CSVs
School_data = "raw_data\schools_complete.csv"
Student_data = "raw_data\students_complete.csv"
```


```python
# Read with pandas--low_memory required to suppress errors about mixed data types
Student_data_df = pd.read_csv(Student_data, low_memory=False)
#Student_data_df.head()
```


```python
# Read with pandas--low_memory required to suppress errors about mixed data types
School_data_df = pd.read_csv(School_data, low_memory=False)
#School_data_df.head()
#School_data_df.columns
```


```python
total_students = School_data_df["size"].sum()
#total_students
```


```python
total_schools = School_data_df["name"].count()
#total_schools
```


```python
avg_read_score = Student_data_df["reading_score"].mean()
avg_math_score = Student_data_df["math_score"].mean()
#print(avg_read_score)
#print(avg_math_score)
```


```python
total_budget = School_data_df["budget"].sum()
#total_budget
```


```python
readpass = Student_data_df[Student_data_df["reading_score"] >70]["reading_score"].count()
readpasspercent = readpass/total_students * 100
#readpasspercent
```


```python
mathpass = Student_data_df[Student_data_df["math_score"] >70]["math_score"].count()
mathpasspercent = mathpass/total_students * 100
#mathpasspercent
```


```python
# Create a new table consolodating above calculations
district_summary = pd.DataFrame({"Total Schools": [total_schools],
                                   "Total Students": [total_students],
                                   "Total Budget": [total_budget],
                                   "Average Math Score": [avg_math_score],
                                   "Average Reading Score": [avg_read_score],
                                   "% Passing Math":[mathpasspercent],
                                   "% Passing Reading":[readpasspercent],
                                   "% Overall Passing Rate": [overallpasspercent],
})
district_summary  = district_summary [["Total Schools", 
                                         "Total Students", 
                                         "Total Budget", 
                                         "Average Math Score", 
                                         "Average Reading Score", 
                                         "% Passing Math", 
                                         "% Passing Reading",
                                          "% Overall Passing Rate"]]
district_summary = district_summary.round(2)

#district_summary
```


```python
overallpass_df = Student_data_df.loc[(Student_data_df["reading_score"] > 70) & (Student_data_df["math_score"] > 70), :]
overallpass = overallpass_df["Student ID"].count()
overallpasspercent = overallpass/total_students * 100
#overallpasspercent
```


```python
School_data_df["Per Student Budget"] = School_data_df["budget"]/School_data_df["size"]
#School_data_df
```


```python
district_summary["Total Students"] = district_summary["Total Students"].map("{0:,.0f}".format)
district_summary["Total Budget"] = district_summary["Total Budget"].map("${0:,.0f}".format)
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
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39,170</td>
      <td>$24,649,428</td>
      <td>78.99</td>
      <td>81.88</td>
      <td>72.39</td>
      <td>82.97</td>
      <td>60.8</td>
    </tr>
  </tbody>
</table>
</div>




```python
Student_score_byschool = Student_data_df.groupby("school")
Student_score_byschool = Student_score_byschool["reading_score","math_score"].mean()
#Student_score_byschool
```


```python
Student_readpass = Student_data_df.loc[Student_data_df["reading_score"] > 70,:]
Student_mathpass = Student_data_df.loc[Student_data_df["math_score"] > 70,:]
Student_overallpass = Student_data_df.loc[(Student_data_df["math_score"] > 70) & (Student_data_df["reading_score"] > 70) ,:]

```


```python
Student_score_byschool["readpassno"] = Student_readpass.groupby("school")["reading_score"].count()
Student_score_byschool["mathpassno"] = Student_mathpass.groupby("school")["math_score"].count()
Student_score_byschool["overallpassno"] = Student_overallpass.groupby("school")["math_score"].count()
Student_score_byschool=Student_score_byschool.rename(columns={"reading_score":"Average Reading Score","math_score": "Average Math Score"})
#Student_score_byschool
```


```python
School_data_df = School_data_df.rename(columns={"name":"school", "type":"School Type","size": "Total Students","budget": "Total School Budget"})
#School_data_df
```


```python
School_data_df = School_data_df.loc[:,["school", "School Type","Total Students","Total School Budget","Per Student Budget"]]
#School_data_df
```


```python
School_data_df=School_data_df.sort_values("school")
School_data_df= School_data_df.set_index("school")
#School_data_df 
```


```python
school_summary_df = pd.merge(School_data_df, Student_score_byschool, left_index = True ,right_index = True)
#school_summary_df
```


```python
school_summary_df["% Passing Math"] = school_summary_df["mathpassno"]/school_summary_df["Total Students"]*100
school_summary_df["% Passing Reading"] = school_summary_df["readpassno"]/school_summary_df["Total Students"]*100
school_summary_df["% Overall Passing Rate"] = school_summary_df["overallpassno"]/school_summary_df["Total Students"]*100
#school_summary_df.columns
#school_summary_df
```


```python
school_summary = school_summary_df[['School Type', 'Total Students', 'Total School Budget',
       'Per Student Budget', 'Average Reading Score', 'Average Math Score',
       '% Passing Math', '% Passing Reading', '% Overall Passing Rate']]
#school_summary
```


```python
school_summary_df = school_summary_df.drop('readpassno',axis=1)
school_summary_df = school_summary_df.drop('mathpassno',axis=1)
school_summary_df = school_summary_df.drop('overallpassno',axis=1)
#school_summary_df.columns
```


```python
school_summary_df["Total School Budget"] = school_summary_df["Total School Budget"].map("${0:,.2f}".format)
school_summary_df["Per Student Budget"] = school_summary_df["Per Student Budget"].map("${0:,.2f}".format)
school_summary = school_summary_df
school_summary
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
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Reading Score</th>
      <th>Average Math Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>District</td>
      <td>4976</td>
      <td>$3,124,928.00</td>
      <td>$628.00</td>
      <td>81.033963</td>
      <td>77.048432</td>
      <td>64.630225</td>
      <td>79.300643</td>
      <td>51.145498</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$1,081,356.00</td>
      <td>$582.00</td>
      <td>83.975780</td>
      <td>83.061895</td>
      <td>89.558665</td>
      <td>93.864370</td>
      <td>84.015070</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
      <td>$639.00</td>
      <td>81.158020</td>
      <td>76.711767</td>
      <td>63.750424</td>
      <td>78.433367</td>
      <td>49.915226</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2739</td>
      <td>$1,763,916.00</td>
      <td>$644.00</td>
      <td>80.746258</td>
      <td>77.102592</td>
      <td>65.753925</td>
      <td>77.510040</td>
      <td>51.296093</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1468</td>
      <td>$917,500.00</td>
      <td>$625.00</td>
      <td>83.816757</td>
      <td>83.351499</td>
      <td>89.713896</td>
      <td>93.392371</td>
      <td>83.651226</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4635</td>
      <td>$3,022,020.00</td>
      <td>$652.00</td>
      <td>80.934412</td>
      <td>77.289752</td>
      <td>64.746494</td>
      <td>78.187702</td>
      <td>50.161812</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087.00</td>
      <td>$581.00</td>
      <td>83.814988</td>
      <td>83.803279</td>
      <td>90.632319</td>
      <td>92.740047</td>
      <td>84.074941</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
      <td>$655.00</td>
      <td>81.182722</td>
      <td>76.629414</td>
      <td>63.318478</td>
      <td>78.813850</td>
      <td>49.914296</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>$650.00</td>
      <td>80.966394</td>
      <td>77.072464</td>
      <td>63.852132</td>
      <td>78.281874</td>
      <td>49.800462</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>$609.00</td>
      <td>84.044699</td>
      <td>83.839917</td>
      <td>91.683992</td>
      <td>92.203742</td>
      <td>84.823285</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363.00</td>
      <td>$637.00</td>
      <td>80.744686</td>
      <td>76.842711</td>
      <td>64.066017</td>
      <td>77.744436</td>
      <td>49.437359</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1761</td>
      <td>$1,056,600.00</td>
      <td>$600.00</td>
      <td>83.725724</td>
      <td>83.359455</td>
      <td>89.892107</td>
      <td>92.617831</td>
      <td>83.191369</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130.00</td>
      <td>$638.00</td>
      <td>83.848930</td>
      <td>83.418349</td>
      <td>90.214067</td>
      <td>92.905199</td>
      <td>84.281346</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
      <td>$578.00</td>
      <td>83.989488</td>
      <td>83.274201</td>
      <td>90.932983</td>
      <td>93.254490</td>
      <td>84.888305</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>$1,049,400.00</td>
      <td>$583.00</td>
      <td>83.955000</td>
      <td>83.682222</td>
      <td>90.277778</td>
      <td>93.444444</td>
      <td>84.444444</td>
    </tr>
  </tbody>
</table>
</div>




```python
top_performing_schools_bypassrate = school_summaryf.sort_values("% Overall Passing Rate", ascending=False)
top_performing_schools_bypassrate = top_performing_schools_bypassrate.iloc[0:4,0:9]
top_performing_schools_bypassrate
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
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Reading Score</th>
      <th>Average Math Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
      <td>$578.00</td>
      <td>83.989488</td>
      <td>83.274201</td>
      <td>90.932983</td>
      <td>93.254490</td>
      <td>84.888305</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>$609.00</td>
      <td>84.044699</td>
      <td>83.839917</td>
      <td>91.683992</td>
      <td>92.203742</td>
      <td>84.823285</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1800</td>
      <td>$1,049,400.00</td>
      <td>$583.00</td>
      <td>83.955000</td>
      <td>83.682222</td>
      <td>90.277778</td>
      <td>93.444444</td>
      <td>84.444444</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1635</td>
      <td>$1,043,130.00</td>
      <td>$638.00</td>
      <td>83.848930</td>
      <td>83.418349</td>
      <td>90.214067</td>
      <td>92.905199</td>
      <td>84.281346</td>
    </tr>
  </tbody>
</table>
</div>




```python
bottom_performing_schools_bypassrate = school_summaryf.sort_values("% Overall Passing Rate", ascending=True)
bottom_performing_schools_bypassrate = bottom_performing_schools_bypassrate.iloc[0:4,0:9]
bottom_performing_schools_bypassrate
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
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Reading Score</th>
      <th>Average Math Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3999</td>
      <td>$2,547,363.00</td>
      <td>$637.00</td>
      <td>80.744686</td>
      <td>76.842711</td>
      <td>64.066017</td>
      <td>77.744436</td>
      <td>49.437359</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>$650.00</td>
      <td>80.966394</td>
      <td>77.072464</td>
      <td>63.852132</td>
      <td>78.281874</td>
      <td>49.800462</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
      <td>$655.00</td>
      <td>81.182722</td>
      <td>76.629414</td>
      <td>63.318478</td>
      <td>78.813850</td>
      <td>49.914296</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
      <td>$639.00</td>
      <td>81.158020</td>
      <td>76.711767</td>
      <td>63.750424</td>
      <td>78.433367</td>
      <td>49.915226</td>
    </tr>
  </tbody>
</table>
</div>




```python
Student_data_df["grade"] = Student_data_df["grade"].astype('category', categories = ["9th", "10th","11th","12th"])
pd.pivot_table(Student_data_df,index=["school"],values=["math_score"],
               columns=["grade"],aggfunc=[np.mean])
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
    <tr>
      <th></th>
      <th colspan="4" halign="left">mean</th>
    </tr>
    <tr>
      <th></th>
      <th colspan="4" halign="left">math_score</th>
    </tr>
    <tr>
      <th>grade</th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.083676</td>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.094697</td>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.403037</td>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.361345</td>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.044010</td>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.438495</td>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.787402</td>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>77.027251</td>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.187857</td>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.625455</td>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.859966</td>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.420755</td>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.590022</td>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.085578</td>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.264706</td>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>




```python
pd.pivot_table(Student_data_df,index=["school"],values=["reading_score"],
               columns=["grade"],aggfunc=[np.mean])
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
    <tr>
      <th></th>
      <th colspan="4" halign="left">mean</th>
    </tr>
    <tr>
      <th></th>
      <th colspan="4" halign="left">reading_score</th>
    </tr>
    <tr>
      <th>grade</th>
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.303155</td>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.676136</td>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.198598</td>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.632653</td>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.369193</td>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.866860</td>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.677165</td>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.290284</td>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>81.260714</td>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.807273</td>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.993127</td>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>84.122642</td>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.728850</td>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.939778</td>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.833333</td>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>




```python
school_summary_df["Per Student Budget"]=school_summary_df["Per Student Budget"].str.replace('$','')
school_summary_df["Per Student Budget"]=school_summary_df["Per Student Budget"].astype(float)
#school_summary_df
```


```python
# Create the bins in which Data will be held
bins = [0, 585, 615, 645, 675]

# Create the names for the four bins
spend_ranges_group = ["<$585", '$585-615', '$615-645', '$645-675']
```


```python
budget_df = school_summary_df
```


```python
budget_df["Spending Ranges (Per Student)"] = pd.cut(budget_df["Per Student Budget"], bins, labels=spend_ranges_group)
#budget_df
```


```python
school_summary_df_sr = budget_df.groupby("Spending Ranges (Per Student)")
school_summary_df_sr.max()
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
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Reading Score</th>
      <th>Average Math Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Ranges (Per Student)</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;$585</th>
      <td>Charter</td>
      <td>2283</td>
      <td>$248,087.00</td>
      <td>583.0</td>
      <td>83.989488</td>
      <td>83.803279</td>
      <td>90.932983</td>
      <td>93.864370</td>
      <td>84.888305</td>
    </tr>
    <tr>
      <th>$585-615</th>
      <td>Charter</td>
      <td>1761</td>
      <td>$585,858.00</td>
      <td>609.0</td>
      <td>84.044699</td>
      <td>83.839917</td>
      <td>91.683992</td>
      <td>92.617831</td>
      <td>84.823285</td>
    </tr>
    <tr>
      <th>$615-645</th>
      <td>District</td>
      <td>4976</td>
      <td>$917,500.00</td>
      <td>644.0</td>
      <td>83.848930</td>
      <td>83.418349</td>
      <td>90.214067</td>
      <td>93.392371</td>
      <td>84.281346</td>
    </tr>
    <tr>
      <th>$645-675</th>
      <td>District</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
      <td>655.0</td>
      <td>81.182722</td>
      <td>77.289752</td>
      <td>64.746494</td>
      <td>78.813850</td>
      <td>50.161812</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Create the bins in which Data will be held
schoolsizebins = [0, 1000, 2000, 5000]

# Create the names for the four bins
school_size_group = ["Small (<1000)","Medium (1000-2000)","Large (2000-5000)"]
school_summary_bysize = school_summary
#school_summary_bysize
```


```python
school_summary_bysize["Size Range"] = pd.cut(school_summary_bysize["Total Students"], schoolsizebins, labels=school_size_group)
#school_summary_bysize
```


```python
school_summary_dfbysize = school_summary_bysize.groupby("Size Range")
school_summary_dfbysize.max()
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
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Reading Score</th>
      <th>Average Math Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
      <th>Spending Ranges (Per Student)</th>
    </tr>
    <tr>
      <th>Size Range</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small (&lt;1000)</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858.00</td>
      <td>609.0</td>
      <td>84.044699</td>
      <td>83.839917</td>
      <td>91.683992</td>
      <td>92.740047</td>
      <td>84.823285</td>
      <td>$585-615</td>
    </tr>
    <tr>
      <th>Medium (1000-2000)</th>
      <td>Charter</td>
      <td>1858</td>
      <td>$917,500.00</td>
      <td>638.0</td>
      <td>83.975780</td>
      <td>83.682222</td>
      <td>90.277778</td>
      <td>93.864370</td>
      <td>84.444444</td>
      <td>$615-645</td>
    </tr>
    <tr>
      <th>Large (2000-5000)</th>
      <td>District</td>
      <td>4976</td>
      <td>$3,124,928.00</td>
      <td>655.0</td>
      <td>83.989488</td>
      <td>83.274201</td>
      <td>90.932983</td>
      <td>93.254490</td>
      <td>84.888305</td>
      <td>$645-675</td>
    </tr>
  </tbody>
</table>
</div>




```python
school_summary_bytype = school_summary_df.groupby("School Type")
school_summary_bytype.max()
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
      <th>Total Students</th>
      <th>Total School Budget</th>
      <th>Per Student Budget</th>
      <th>Average Reading Score</th>
      <th>Average Math Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>% Overall Passing Rate</th>
      <th>Spending Ranges (Per Student)</th>
      <th>Size Range</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>2283</td>
      <td>$917,500.00</td>
      <td>638.0</td>
      <td>84.044699</td>
      <td>83.839917</td>
      <td>91.683992</td>
      <td>93.864370</td>
      <td>84.888305</td>
      <td>$615-645</td>
      <td>Large (2000-5000)</td>
    </tr>
    <tr>
      <th>District</th>
      <td>4976</td>
      <td>$3,124,928.00</td>
      <td>655.0</td>
      <td>81.182722</td>
      <td>77.289752</td>
      <td>65.753925</td>
      <td>79.300643</td>
      <td>51.296093</td>
      <td>$645-675</td>
      <td>Large (2000-5000)</td>
    </tr>
  </tbody>
</table>
</div>


