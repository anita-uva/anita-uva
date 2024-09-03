# Data Science and Analytics

* TOC
{:toc}


## Data Visualization
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

The survey asked respondents to rate their feelings of Anxiety on a scale from 1 to 5.  This is the sum response of all survey participants, regardless of personal vaccination status, from January 2021 through July 2021. 

<img width="788" alt="MentalHealth-NewsLinePlot" src="https://github.com/user-attachments/assets/7d73685f-036e-48a8-b8bc-1c6d706be3c9">

Notice, as the vaccination is distributed, people begin to feel less anxious, overall.  But we also see people on edge, with notable reactions to potentially destabilizing news.


## Data Collection and Cleaning
I am particularly interested in Data Cleaning, Collection and Exploratory Data Analysis.

### Command Line Data Transform:  Shipments Data
The shipments data is part of a slightly larger project where I converted flat data files into a relational database.

<dl>
<dt>This is the original file, before cleaning</dt>
<dd>Right click and open in a new tab increase the size</dd>
<dd><img width="1250" alt="Shipments_Before" src="https://github.com/user-attachments/assets/1096923c-4018-48da-b9ec-35ef44b732f1"></dd>
</dl>

<dl>
<dt>I used <mark>sed and awk</mark> to clean the data at the command line.</dt>
<dd>This is a one-liner, but it is modified into multiple lines to better fit the boundaries of this presentation space.</dd>
</dl>
```
cat Shipments\ 2021.csv.orig |
sed '1d' | tr -d '\r' |
awk 'BEGIN { FS = ",";OFS="," } ; {split($2,d," ");split(d[1],shd,"/");
print $1, "20"shd[3]"-"shd[2]"-"shd[1], $3, $4, $5, $8, $9, $10, $15, $16, $17, $18, $19, $28 }' |
sed s/\,/\"\,\"/g |
awk -FS=, '{ print "INSERT INTO shipments VALUES(" "\""$0"\"" ")" }' > shipments.inserts.txt
```
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
<dt>I used <mark>python and sqlite3</mark> to create a <mark>repeatable and scalable process</mark> to load and reload the database.</dt> 
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

<dl>
<dt>Review the code where this file is used</dt> 
<dd>Please use the browser back button to return to this portfolio</dd>
</dl>
[Freight Marketplace Code](./Freight_Marketplace.html)

### Web Scraping:  Beautiful Soup simple example
Here is a short example of web scraping with Beautiful Soup.  I am also good at obtaining data using APIs and Database Clients.  I believe I can collect data in any format that is legally available.

<dl>
<dt>Web Scraping</dt> 
<dd>Beautiful Soup simple example</dd>
</dl>

```python
## Set the user-agent string
user_agent = {'User-agent' : 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.87 Safari/1.0 (agt@mailer.com)'}

## Set the base url
base_url = 'http://books.toscrape.com/'

## Check for robots.txt
robots = requests.get(base_url+"robots.txt", headers = user_agent)
if robots.status_code == 404:
    print('This site does not have a robots.txt\n\n{}'.format(robots))
else:
    print(robots.text)

def books_from_page(url):

    global user_agent
    page_of_books = pd.DataFrame()
    
    ## Get the starting page
    page = requests.get(url, headers = user_agent)

    ## Make it useful in python
    books_raw = BeautifulSoup(page.text, 'html')

    ## Parse out titles
    title_list = [x.h3.a['title'] for x in books_raw.findChildren('article')]
    
    ## Parse out Prices and remove currency symbol
    prices = [x.find_all('div')[1].p.text.replace('Â£', '') for x in books_raw.findChildren('article')]

    ## Star Rating for each book
    star_rating = [x.p['class'][1] for x in books_raw.findChildren('article')]
    
    ## Book Cover Image URL
    image_url = [x.div.a.img['src'] for x in books_raw.findChildren('article')]
    
    ## Put the data together
    data_for_cols = list(zip(title_list, prices, star_rating, image_url))

    ## Place it in a nicely formatted Datafame
    page_of_books = pd.DataFrame(data_for_cols, columns=['Title', 'Price', 'Rating', 'Image URL'])
    
    return page_of_books

## Run books from page function
books_from_page(base_url)
```
<dl>
<dt>Web Scraping Result</dt> 
<dd>The result is nicely formatted local data from http://books.toscrape.com/</dd>
</dl>

<img width="793" alt="Screenshot 2024-09-03 at 11 34 57 AM" src="https://github.com/user-attachments/assets/659d8ea3-03b1-4f4f-98df-b55f568f385d">

### API Data Collection
This short example uses python and pandas for querying over endpoints and for using a library as a client interaction to collect data from genius.com.

<dl>
<dt>API</dt> 
<dd>Data Collection through endpoints or libraries</dd>
</dl>

```python
## Establish root and enpoint for the search API
root = 'https://api.genius.com'
endpoint = '/search'

## Set up the variables using access_token as a parameter
searchme = { 'q' : 'Bob Dylan', 'access_token' : ACCESS_TOKEN }

## Use requests to query endpoint
r = requests.get(root+endpoint, params=searchme )

## Create a json friendly variable
dylan_json = json.loads(r.text)

# Use the JSON Structure to find the dylan url; break these up to export to PDF
dylan_url = \
    dylan_json['response']['hits'][0]['result']['primary_artist']['api_path']

print('Bob Dylan\'s API endpoint \n{}' \
      .format(root+dylan_url))

## Use the Endpoint

## Prefer authorization header instead of access_token in the url
authtoken = { 'Authorization' : str('Bearer '+ACCESS_TOKEN) }

## finally ran across a sort parameter, hooray...
params = {'sort' : 'popularity'}
r = requests.get(root+dylan_url+'/songs', headers=authtoken, params=params )

## It gives us 20 without further wrangling, so just load it up
top20songs = json.loads(r.text)

## Use Normalize to break down nested dicts
pd.json_normalize(top20songs, record_path=['response', 'songs'])

## Use the Library

## Check for existing library installation
hm = !pip show lyricsgenius

## if not installed, perform install
if 'lyricsgenius' not in hm[0]:
    !pip install lyricsgenius

import lyricsgenius
genius = lyricsgenius.Genius(ACCESS_TOKEN)

# We are given the song name and artist name
target_song   = 'Tangled Up in Blue'
target_artist = 'Bob Dylan'

# Because of API read-timed-out errors I am looking for the song directly
songs = genius.search_songs(target_song + ' ' + target_artist)

# The API returns the song we want as the first item
tuib_id = [x['result']['id'] for x in songs['hits']][0]

# Using Song ID
tuib_lyrics = genius.lyrics(tuib_id)

# Make it pretty
print(tuib_lyrics)
```
<img width="382" alt="Screenshot 2024-09-03 at 4 03 58 PM" src="https://github.com/user-attachments/assets/63db7d7d-2eac-4ea8-a430-f8cee7181e15">


### Data Transformation
Data transformation to useful, human readable information. 

<dl>
<dt>Transform columns from numeric codes</dt> 
<dd>Data is only useful when we can understand the what the values represent.</dd>
</dl>

```python

## Define a function to map values for columns WD1 - WD7
def set_pct_values(np_series_vals):

    myarr = np_series_vals.unique()

    lth = [str(y) for y in range(0,46)]
    aeh = [str(y) for y in range(46,55)]
    mth = [str(y) for y in range(55,101)]

    valdict = {'997': "Don't know", '998': "Refusal", '999': "Blank\Invalid"}

    for x in myarr:
        if x in valdict.keys():
            continue
        `
        if str(x) in lth:
            valdict[x]= 'Less Than Half'
        elif str(x) in mth:
            valdict[x] = 'More Than Half'
        elif str(x) in aeh:
            valdict[x] = 'About Half'

    return valdict

## Build the multi-column replacement map
replace_map = {'org_profit_or_gov':
                   {'1': 'For profit, public','2': 'For profit, private','3': 'Non-profit',
                    '4': 'State or local government','5': 'Federal government',
                    '6': 'Other','97': np.nan,'98': np.nan, '99': np.nan},
               'health_coverage_offered':
                   {'1': 'Full insurance coverage offered','2': 'Partial insurance coverage offered',
                    '3': 'No insurance coverage offered','97': np.nan,'98': np.nan,'99': np.nan },
               'emp_yoy_cost_increase':
                   {'1':'Larger','2':'Smaller','3':'About the same','96': np.nan,
                    '97': np.nan,'98': np.nan,'99': np.nan},
               'emp_parttime_offer':
                   {'1':'Yes','2':'No','97': np.nan,'98': np.nan,'99': np.nan},
               'emp_health_ed_offer':
                   {'1':'Yes','2':'No','97': np.nan,'98': np.nan },
               'emp_wfh_offer':
                   {'1':'Yes','2':'No','97': np.nan,'98': np.nan,'99': np.nan},
               
               'emp_demographic_lt30':      set_pct_values(workingdf['emp_demographic_lt30']),
               'emp_demographic_gt59':      set_pct_values(workingdf['emp_demographic_gt59']),
               'emp_demographic_female':    set_pct_values(workingdf['emp_demographic_female']),
               'emp_demographic_hourly':    set_pct_values(workingdf['emp_demographic_hourly']),
               'emp_demographic_notday':    set_pct_values(workingdf['emp_demographic_notday']),
               'emp_demographic_notoffice': set_pct_values(workingdf['emp_demographic_notoffice']),
               'emp_demographic_union':     set_pct_values(workingdf['emp_demographic_union']),
               'emp_turnover_annual':       set_pct_values(workingdf['emp_turnover_annual'])
              }

## Replace everythig in one call, set some values as missing
workingdf = workingdf.replace(replace_map)

## Show the values are used as expected 
workingdf.sample(15).T
```
<img width="742" alt="Screenshot 2024-09-03 at 11 50 28 AM" src="https://github.com/user-attachments/assets/13dc1755-5bef-4646-8abf-d8a4efba5422">

## Exploratory Data Analysis
Some of my favorite work is EDA.  I love to discover what the data has to say!



### Feature Reduction
#### LDA
#### Chi-Squared feature selection in pyspark
For survey data I chose to use the Chi Squared selector to see the best features to predict a binary response to whether a respondent had recieved the vaccine or not.  The data was reduced from a few hundred features to ten predictors.

```python
from pyspark.ml.feature import ChiSqSelector, VectorAssembler, StringIndexer, VectorIndexer
from pyspark.ml import Pipeline

## Set the number of selected features, this will be used throught the entire investigation
numFeatures = 10

# Set StringOrderType because the default of frequencyDesc labels VACC backwards 
labelIndexer = StringIndexer(inputCol="VACC", outputCol="indexedLabel", stringOrderType='frequencyAsc')

assembler = VectorAssembler(
                inputCols = allCols, 
                outputCol="features",
                handleInvalid='skip')

featureIndexer = VectorIndexer(inputCol="features", outputCol="indexedFeatures", handleInvalid='skip')

selector  = ChiSqSelector(numTopFeatures=numFeatures, featuresCol='indexedFeatures', outputCol="selectedFeatures",labelCol='indexedLabel')

pipeline = Pipeline(stages=[labelIndexer,assembler,featureIndexer,selector])

chisq_model = pipeline.fit(selector_df)

# use the selected predictors throughout the investigation
colList = [allCols[x] for x in chisq_model.stages[-1].selectedFeatures]

# Format the selected features so that they are neatly presented
pd.DataFrame (colList, columns = ['Selected Features'], index=None).style.set_caption('Chi-Squared feature selection')
```
<img width="160" alt="Screenshot 2024-09-03 at 4 50 47 PM" src="https://github.com/user-attachments/assets/f5498edf-b685-495a-9847-82c376c79344">


<dl>
<dt>Feature Importance</dt> 
<dd>Feature Importance is plotted from the Gradient Boosted Trees Classifier.</dd>
</dl>

Here I show the best predictor for getting a covid vaccine in 2021 (when the vaccine was initially released) is simply the week of the year.  Because in that year, the more widely available the vaccine became, the more people were actually getting it.  Maybe what is more notable is that other factors such as education, race, and geographic location were not as siginificant during that time.

```python
# Feature Importances
vals = gbt_pipelineModel.stages[-1].featureImportances.indices
arr = gbt_pipelineModel.stages[-1].featureImportances.toArray()

pd.DataFrame([(colList[f],arr[f]) for f in vals], index=[colList[f] for f in vals])[[1]] \
         .sort_values(by=1, ascending=True) \
         .plot(kind='barh', title='Gradient Boosted Trees\nFeature Importances', legend=False)
plt.show()
```

<img width="645" alt="Screenshot 2024-09-03 at 4 48 52 PM" src="https://github.com/user-attachments/assets/83511ae2-765f-4a23-9cdd-c83350af3754">


### Summary Statistics
### Correlation


## Machine Learning

<!--
src=https://html-preview.github.io/?url=https://github.com/anita-uva/anita-uva.github.io/blob/7383f755fb25c0e1cacd64ce24120cf5618cde84/Freight_Marketplace.html
-->
<!--
**anita-uva/anita-uva** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.
-->
