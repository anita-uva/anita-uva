# Data Science Highlights

<dl>
<dt>Survey Data:  US Census Bureau, Household Pulse Survey</dt>
<dd>Attitudes toward the Covid Vaccination</dd>
<dd>January 2021 through July 2021</dd>
</dl>

The COVID vaccine became available during the first quarter of 2021.  As the vaccine was distributed, first to those with the highest risk, then to the oldest and successively younger individuals, the Household Pulse Survey captured the reasons for those not getting vaccinations.  The total number of unvaccinated people drops dramatically through the months of the study.  As the overall numbers drop, the reasons given for remaining unvaccinated start to shift.  

![vax-whynot-barchartrace](https://github.com/anita-uva/anita-uva.github.io/assets/77550558/87df789c-276b-4601-8260-24919b80da58)

The top 2 reasons never shift:  People were solidly concerned about side effects of this new vaccine, and generally felt better waiting to see how it affected others.  Notice also the reason "others need it first" falls, maybe expectedly, as the vaccine was released to higher risk and lower risk individuals over time.

<dl>
<dt>Survey Data:  US Census Bureau, Household Pulse Survey</dt>
<dd>Visual Summary for Measurement of Mental Health</dd>
</dl>

The survey asked respondents to rate their feelings of Anxiety on a scale from 1 to 5.  This is the average response from January 2021 through July 2021. 

<img width="788" alt="MentalHealth-NewsLinePlot" src="https://github.com/user-attachments/assets/7d73685f-036e-48a8-b8bc-1c6d706be3c9">

Notice, as the vaccination is distributed, people begin to feel less anxious, overall.  We also see people on edge, with notable reactions to destabilizing news.

## Data Cleaning and Preparation
I am particularly interested in Data Cleaning and preparation.  Here are a couple of examples.

### Shipments Data
The shipments data is part of a slightly larger project where I converted flat data files into a relational database.

<dl>
<dt>This is the original file, before cleaning</dt>
<dd>Right click and open in a new tab increase the size</dd>
<dd><img width="1250" alt="Shipments_Before" src="https://github.com/user-attachments/assets/1096923c-4018-48da-b9ec-35ef44b732f1"></dd>
</dl>

<dl>
<dt>I used <mark>sed and awk</mark> to clean the data at the command line.</dt>
<dd>This is a one-liner, but it is modified into multiple lines to better fit the boundaries of this presentation space.</dd>
```
cat Shipments\ 2021.csv.orig |
sed '1d' | tr -d '\r' |
awk 'BEGIN { FS = ",";OFS="," } ; {split($2,d," ");split(d[1],shd,"/");
print $1, "20"shd[3]"-"shd[2]"-"shd[1], $3, $4, $5, $8, $9, $10, $15, $16, $17, $18, $19, $28 }' |
sed s/\,/\"\,\"/g |
awk -FS=, '{ print "INSERT INTO shipments VALUES(" "\""$0"\"" ")" }' > shipments.inserts.txt
```
</dl>
<!--
In case I need the url again for sed and awk command line pic; this has been replaced by a code snippet
<img width="1103" alt="Shipments_SedAwk" src="https://github.com/user-attachments/assets/a1fc9654-830e-4795-b513-91419b55226e">
-->


<dl>
<dt>The resulting cleaned data file of Shipments</dt>
<dd>This has been created as a script, ready to be inserted into the database.  The file was uploaded to github, with other script files, to create a standardized repository for the data files.</dd>
<dd><img width="1028" alt="Shipments_After" src="https://github.com/user-attachments/assets/2d868df8-28b5-4595-a578-dd5e4556f6a9"></dd>
</dl>

<dl>
<dt>I used <mark>python and sqlite3</mark> to create a repeatable and scalable process to load and reload the database.</dt> 
<dd></dd>
</dl>

```python
## Shipments Data file is stored in github
if getEntireDatabase is True:
  shipments_dat = "https://raw.githubusercontent.com/anita-uva/Freight-Marketplace/main/prod/shipments.inserts.txt?token=ASPVHXTCQAXVDA5UOXNSXNLBB5CAM"
else:
  shipments_dat = "https://raw.githubusercontent.com/anita-uva/Freight-Marketplace/main/poc/shipments.poc.inserts.txt?token=ASPVHXTEANVMPLIHMVRW6WTBB24IU"

## Read Shipments Data
dat = gitread.request("GET", shipments_dat)

## The INSERT statements are contained in the file because we have alot of data 
cursor.executescript(dat.data.decode("utf-8"))

## Commit Changes
conn.commit()
```
Place holder for full project.

<!--
**anita-uva/anita-uva** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.
-->
