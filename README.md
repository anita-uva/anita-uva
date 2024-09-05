# Data Science and Analytics

* TOC
{:toc}


## Data Visualization
Telling the data's story is best accomplished with powerful graphics.  Here are a couple of examples of useful graphics.

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

### TODO: Steaming Data with Kafka
Need to create a better example than I have existing...

## Exploratory Data Analysis
Some of my favorite work is improving the scalability and processes for EDA.  And I love to discover what the data has to say!

### Data Transformation
Data transformation takes on different meanings.  For example, Survey data might be better represented as a text-based, readable scale of values for better understanding, as opposed to a numeric 1-5 kind of scale.  But, we might also change the values of the data to exist within a known distribution, or to be changed to the same scale as other variables.

Here are a couple of snippets showing data transformation.

#### Reveal Meanings with Understandable Values
Data transformation to useful, quickly digestable information.  This is especially useful for presenting data to a broad audience or to help explain the raw data.

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

#### Data Normalization & Standardization
In some instances, placing data will need to be placed on the same scale or normalized to an expected distribution.

<dl>
<dt>Transform data Values</dt> 
<dd>Preparing for model consumption.</dd>
</dl>

```python
# Empirical review of distributions and outliers
_=sns.pairplot(income_compare, height=4)
```
<img width="594" alt="Screenshot 2024-09-05 at 1 17 04 PM" src="https://github.com/user-attachments/assets/02638cc5-f7b0-4458-8a09-50752912f645">

* The histogram for occuplational_prestige data shows an approximately normal distribution but needs standardized to be centered on the mean.
* The Income histogram shows a right skew and requires normalization.
* The scatter plots indicate outliers around the value 80 for occupational prestige, and possibly above 120,000 in income.

```python
from scipy.stats import boxcox, zscore

# Income is not normal, so normalize here
inc_data, inc_lambda = boxcox(income_compare.income)

# With Income and Prestige approximately Normal
# Make a new dataframe
normal = pd.DataFrame(zip(inc_data, income_compare.occupational_prestige, income_compare.gender))\
            .rename(columns={0:'income', 1:'occupational_prestige', 2: 'gender'})

# Use a zscore test to eliminate outliers; we're removing +-2.5 std deviations
inc_drop_list = [i for i,x in enumerate(zscore(normal.income)) if x > 2.5 or x < -2.5]
prs_drop_list = [i for i,x in enumerate(zscore(normal.occupational_prestige)) if x > 2.5 or x < -2.5]

# drop rows from dataframe, based on previously zscore +-2.5 std deviations
normal = normal.drop(index=inc_drop_list)
normal = normal.drop(index=prs_drop_list)

# Plot New Histograms to show normalized and standardized data

fig, (ax1, ax2) = plt.subplots(1, 2)
fig.suptitle('Approximately Normal Distribution for Income and Occupational Prestige')

plt.rc('figure', figsize = (10,5))
plt.rc('font', size=12)

ax1.hist(normal.income, bins=14, color='lavender',  edgecolor='black')
ax1.set_title('Income')
ax1.set(xlabel='Annual Income', ylabel='Frequency')

ax2.hist(normal.occupational_prestige, bins=10, color='lavender', edgecolor='black')
_=ax2.set_title('Occupational Prestige')
_=ax2.set(xlabel='Occupational Prestige Score')
```

<img width="413" alt="Screenshot 2024-09-05 at 1 24 17 PM" src="https://github.com/user-attachments/assets/4d9fe201-6bba-432b-b40b-07f8a27f9898">

If this data will be used in models for prediction, I would scale these two items together. But for EDA, it is easier to understand and explain the variables visually when they are on their original scales.

```python
plt.figure(figsize=(7,7))
sns.kdeplot(normal.income, shade=True, label='Income')
sns.kdeplot(normal.occupational_prestige, shade=True, label='Prestige')
plt.ylabel('Frequency')
plt.xlabel('Scale')
plt.title('Distribution of Income and Prestige')
plt.legend()
```

<img width="496" alt="Screenshot 2024-09-05 at 1 53 36 PM" src="https://github.com/user-attachments/assets/ba342011-6516-4bf9-93c2-49908c19972d">


### Dimensionality Reduction & Feature Importance
Discovering the most influential features in a data set, and exploring the topics in unstructured data contribute to presenting the initial understanding of an unknown data set.

#### Latent Dirichlet Allocation (LDA) for Topic Modeling
This example uses LDA for topic modeling, based on articles collected from the 2016 presidential election cycle.  The larger study compares real news to fake news as determined by the fact-checking website, Politifact.  This code snippet represents a dimensionality reduction effort for exploring topics.

Because the dimensionality of the topic variable is assumed known and fixed in LDA (i.e. it will not tell us the ideal number of topics), I created a repeatable process for LDA topic modeling which allows rapid investigation of topics given any combination of:

* Vocabulary size (Number of features)
* Number of topics to plot
* Number of words per topic

I centralized the variables so that the process is to change the values in one place, then run the same code cell to see the result.

```python
def dynamic_grid(n_topics):
    
    ## Dynamic plot grid for topic models
    
    ## First Arrange the plot rows and columns to fit
    num_topics = n_topics
    
    multiples = [x for x in range(1, num_topics+1) if num_topics%x == 0]

    if multiples[-2] == 1:
        ax_rows=1
        ax_cols=num_topics
    else:
        ax_rows = int(num_topics/multiples[-2])
        ax_cols = multiples[-2]
   
    # We never want more than 5 cols wide
    while ax_cols > 5:
        ax_cols = ceil(ax_cols/2)
        ax_rows = ax_rows *2
        
        # eliminate empty graph rows
        overflow = (ax_cols*ax_rows)-num_topics
        if overflow >= ax_cols:
            ax_rows = ax_rows-floor(overflow/ax_cols)

    return(ax_rows, ax_cols)
    
def plot_word_topics(model, feature_names, n_top_words, title):
    
    num_topics = model.components_.shape[0]
    ax_rows, ax_cols = dynamic_grid(num_topics)
            
    fig, axes = plt.subplots(ax_rows, ax_cols, figsize=(30, num_topics+10), sharex=True)
    
    ## https://scikit-learn.org/stable/auto_examples/applications/plot_topics_extraction_with_nmf_lda.html
    
    axes = axes.flatten()
    dfs=[]
    
    for topic_idx, topic in enumerate(model.components_):
        top_features_ind = topic.argsort()[: -n_top_words - 1 : -1]
        top_features = [feature_names[i] for i in top_features_ind]
        weights = topic[top_features_ind]
        dfs.append(pd.DataFrame( { 'topic': topic_idx+1, 'features': ' '.join(top_features) }, index=[topic_idx+1] ))

        ax = axes[topic_idx]
        ax.barh(top_features, weights, height=0.7)
        ax.set_title(f"Topic {topic_idx +1}", fontdict={"fontsize": 30})
        ax.invert_yaxis()
        ax.tick_params(axis="both", which="major", labelsize=20)
        for i in "top right left".split():
            ax.spines[i].set_visible(False)
        fig.suptitle(title, fontsize=40)

    plt.subplots_adjust(top=0.90, bottom=0.05, wspace=0.90, hspace=0.3)
    plt.show()
    
    return(pd.concat(dfs))

# This includes defaults, but they are not specifically meaningful
# They are just a catch-all in the event something is missing. 
def get_lda_word_model(corpus, 
                       min_df = 0.20, 
                       max_df = 0.80, 
                       lda_features  = 1000, 
                       lda_components= 20, 
                       topic_word_len= 10):
    
    '''Takes values for sklearn's CountVectorizer and LatentDirichletAllocation;
    Displays a grid of topics plotted with their words and returns the LDA Model and Vectorizer Model'''
    
    tf_vectorizer = CountVectorizer(max_df=max_df, min_df=min_df, max_features=lda_features, stop_words="english")
    tf = tf_vectorizer.fit_transform(corpus)

    tf_feature_names = tf_vectorizer.get_feature_names()

    lda = LatentDirichletAllocation(n_components=lda_components,max_iter=10,random_state=42)
    lda.fit(tf)

    topic_df = plot_word_topics(lda, tf_feature_names, n_topic_size, "Latent Dirichlet Allocation (LDA)")
    
    return(lda, tf_vectorizer, topic_df)

## Set values for Count Vectorizer and LDA
n_vocab_size = 1000 # Feature size for LDA to target
n_topics     = 20   # How many topics to plot
n_topic_size = 10   # Number of words per topic

## Use floats here, not integers
max_df = 0.80 ## words repeated in 80%  of documents are too common, so eliminate
min_df = 0.10 ## words used in only 10% of documents are too special, eliminate

## Get the LDA Model, a list of feature names, and a dataframe of topics
## from get_lda_word_model
mylda_r, myvect_r, topics_r = get_lda_word_model(corpusr, min_df, max_df, n_vocab_size, n_topics, n_topic_size)
```
<dl>
<dt>Resulting Topic Grid</dt> 
<dd>The result is a simplified process for reviewing and retrying any combination of vocabulary size, topics, and number of words per topic.</dd>
  <dd>This shows topics 11-20 (cropped from 1-20 to save space) of a 20 topic investigation with 10 words per topic with a vocabulary of 1000 words.</dd>
</dl>

<img width="998" alt="Screenshot 2024-09-04 at 12 07 13 PM" src="https://github.com/user-attachments/assets/56586463-47b5-4a00-b609-3c665be28f78">

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
selector    = ChiSqSelector(numTopFeatures=numFeatures, featuresCol='indexedFeatures', outputCol="selectedFeatures",labelCol='indexedLabel')
pipeline    = Pipeline(stages=[labelIndexer,assembler,featureIndexer,selector])
chisq_model = pipeline.fit(selector_df)

# use the selected predictors throughout the investigation
colList = [allCols[x] for x in chisq_model.stages[-1].selectedFeatures]

# Format the selected features so that they are neatly presented
pd.DataFrame (colList, columns = ['Selected Features'], index=None).style.set_caption('Chi-Squared feature selection')
```
<img width="160" alt="Screenshot 2024-09-03 at 4 50 47 PM" src="https://github.com/user-attachments/assets/f5498edf-b685-495a-9847-82c376c79344">

<dl>
<dt>Feature Importance</dt> 
<dd>Feature Importance is plotted using the Gradient Boosted Trees Classifier.</dd>
</dl>

Here we see the best predictor for getting a covid vaccination in 2021 (when the vaccine was initially released) is simply the week of the year.  During that year, the more widely available the vaccine became, the more people were getting it.  Maybe what is more notable is that other factors such as education, race, and geographic location were not as siginificant during that time.

```python
# Feature Importances
vals = gbt_pipelineModel.stages[-1].featureImportances.indices
arr  = gbt_pipelineModel.stages[-1].featureImportances.toArray()

pd.DataFrame([(colList[f],arr[f]) for f in vals], index=[colList[f] for f in vals])[[1]] \
         .sort_values(by=1, ascending=True) \
         .plot(kind='barh', title='Gradient Boosted Trees\nFeature Importances', legend=False)
plt.show()
```

<img width="645" alt="Screenshot 2024-09-03 at 4 48 52 PM" src="https://github.com/user-attachments/assets/83511ae2-765f-4a23-9cdd-c83350af3754">

#### Word Clouds
While not truly a dimensionality reduction technique, word clouds are a visually appealing way of setting expectations around possible topics that might be hidden in a corpus.

<dl>
<dt>Word Cloud</dt> 
<dd>A simple but impactful word cloud created from twitter data using the WordCloud library for Python.</dd>
</dl>

```python
## Set up the dataframe so that day is the index
dfwf.set_index(keys=dfwf.day, drop=True, inplace=True)
dfwf.drop(columns=['day'],inplace=True)

## Keep rows only when freq < 200000 (i.e. get rid of outliers on 7-10 and 7-11)
dfwf = dfwf.loc[dfwf.freq < 200000]

## Get the top N words from each day
topNwords = [dfwf.groupby(dfwf.index).get_group(x).nlargest(50, 'freq').word.values for x in dfwf.index.unique()] 

## The Word Cloud library requires a single string with all words space separated
text = ""
for w in topNwords:
    text += " ".join(map(str,w))
    text += " "
# Build the word cloud using matplot lib

def buildwc(maskfile=''):
    
    global tdata_path

    if len(maskfile) > 0:
        mask = np.array(Image.open(tdata_path+maskfile))
        word_cloud = WordCloud(collocations = False, mask=mask, max_font_size = 1000, background_color = 'white', contour_width=2, contour_color='black' ).generate(text)
        figname='covidwordcloud_'+maskfile[0:maskfile.find('.')]+'.png'
    else:
        word_cloud = WordCloud(collocations = False, background_color = 'white', width=700, height=350).generate(text)
        figname = 'covidwordcloud.png'

    
    plt.imshow(word_cloud, interpolation='none') # could also be bilinear
    plt.axis("off")
    plt.savefig(figname)
    plt.show()

buildwc()
```
<img width="515" alt="Screenshot 2024-09-04 at 2 08 15 PM" src="https://github.com/user-attachments/assets/d2b5f0a6-b088-49ac-b885-61858cb875af">

I arranged the wordcloud code into a function to make it possible to test multiple mask files easily, to see which cloud made the best impression for a presentation.  Here is a second example using a different mask.

```python
buildwc(maskfile = 'virusshape.jpg')
```
<img width="407" alt="Screenshot 2024-09-04 at 2 11 26 PM" src="https://github.com/user-attachments/assets/387f8138-9793-4930-b6bc-c9c0de3ac74e">

### Statistical EDA
The best way to describe the underpinnings of a data set is with some straight-forward, easily understandable summary statistics, which can tell us anything from data imbalances, to population biases, to typical value ranges for variables, to consistency or inconsistency of text based fields, to relationships between variables, to the way nulls or otherwise missing values are recorded, and even more.

#### Bar Chart showing Data Imbalance
This bar chart shows how the selected outcome variable distribution changes over scope of time, by week.

```python
import seaborn as sns
import matplotlib.pyplot as plt

# Use seaborn and pandas for a quick plot
sns.set_theme(style="ticks", color_codes=True)

df_weekvac = df.select('VACC','WEEK').toPandas()

sns.catplot(y="WEEK", hue="VACC", kind="count", legend=False,
            palette="pastel", edgecolor=".6", data=df_weekvac)

plt.title("Response Variable by Week")
plt.legend(title='Vaccinated', labels=['No', 'Yes'], loc=2, bbox_to_anchor = (1,1))
plt.show()
```
<dl>
<dt>Data Distribution Bar Chart</dt> 
<dd>Notice how the distribution of this outcome variable changes as the week changes.  This indicates a time series data set, which will mean time series testing and models will need to be be pursued.</dd>
</dl>

<img width="628" alt="Screenshot 2024-09-04 at 2 38 20 PM" src="https://github.com/user-attachments/assets/cc76934b-31b2-4eb5-9fa3-b7e786617668">

#### RoBERTa Sentiment Analysis as EDA
In this snippet, I use a pre-trained RoBERTa model to classify article headlines and article text, separately, as Negative, Neutral, or Positive.  The dataset being processed by RoBERTa is comprised of articles that have been labeled with a bias of Left, Center, or Right.  

The goal of this EDA step is to gain an understanding of negativity and positivity by media bias.

```python
# load model and tokenizer

## This pretrained model determines Labels: 0 -> Negative; 1 -> Neutral; 2 -> Positive
## https://huggingface.co/cardiffnlp/twitter-roberta-base-sentiment

roberta   = "cardiffnlp/twitter-roberta-base-sentiment"
model     = AutoModelForSequenceClassification.from_pretrained(roberta)
tokenizer = AutoTokenizer.from_pretrained(roberta)

labels = ['Negative', 'Neutral', 'Positive']

## Sentiment Analysis on Article Snippets (Descriptions)
## Sentiment Analysis on Headlines
for index, row in ad.iterrows():
  snippet = getattr(row, "Description")

  snippet_words = []

  for word in snippet.split(' '):

      if word.startswith('http'):
          word = "http"
      snippet_words.append(word)

  snippet_proc = " ".join(snippet_words)
  #print(snippet_proc)

  encoded_snippet = tokenizer(snippet_proc, return_tensors='pt')
  #print(encoded_snippet)

  # sentiment analysis
  output = model(encoded_snippet['input_ids'], encoded_snippet['attention_mask'])
  #output = model(**encoded_snippet)

  scores = output[0][0].detach().numpy()
  ad.loc[index, ['negative']]  = scores[0]
  ad.loc[index, ['neutral']]   = scores[1]
  ad.loc[index, ['positive']]  = scores[2]
  #print(scores)

  scores = softmax(scores)
  ad.loc[index, ['negative_pct']] = scores[0]
  ad.loc[index, ['neutral_pct']]  = scores[1]
  ad.loc[index, ['positive_pct']] = scores[2]
  #print(scores)

import matplotlib.pyplot as plt
import numpy as np


myplot = hl[['Bias', 'negative_pct', 'positive_pct', 'neutral_pct']].copy()
myplot = myplot.groupby('Bias').agg('mean')
_ = myplot.plot(kind='bar', figsize=(20,5), title="Headlines", color=['burlywood', 'darkorange', 'bisque']).set_xticklabels(labels=myplot.index.unique(), rotation = 0)

adplot = ad[['Bias', 'negative_pct', 'positive_pct', 'neutral_pct']].copy()
adplot = adplot.groupby('Bias').agg('mean')
_=adplot.plot(kind='bar', figsize=(20,5), title="Article Text", color=['burlywood', 'darkorange', 'bisque']).set_xticklabels(labels=myplot.index.unique(), rotation = 0)
```
<dl>
<dt>Sentiment Analysis with Media Bias</dt> 
<dd>Consider the following points with the RoBERTa Sentiment Analysis results.</dd>
</dl>

* We see positivity is consistently the lowest sentiment, regardless of bias. And while it ranks much higher than positivity, negativity is always the second highest sentiment. Surprisingly, neutral is always the highest ranking sentiment for every bias.
  
* When comparing headlines and article texts, we see that positive sentiment for headlines is always ranked below 10%, whereas article texts exhibit more positivity (over 10% in every case) within the article itself. It seems that headlines may be less positive than the article content.
  
* Articles classified as "Center" bias show less negativity in their headines and their article content. Maybe not surprisingly, the Left and Right extremes are more likely to exhibit negativity in their headlines and articles than the more central classifications.
  
* The "Left" bias edges the "Right" bias with the most negativity of any classification, in both headlines and article text.
  
* Center bias is narrowly the most likely to show positivity when comapred to all other biases.

<img width="990" alt="Screenshot 2024-09-04 at 5 01 30 PM" src="https://github.com/user-attachments/assets/e315c498-ef8b-4e82-a476-792afc084051">

[Full Sentiment Analysis Code](./RoBERTa_Sentiment_Analysis.html)

## Regression EDA
Regression demands addressing a unique set of concerns, such as confirmation of linearity in the data, correlation between predictors, and influential outliers, among others.

### Outliers
Data points that are beyond the majority of the samples may be cause for concern in Regression scenarios.  The first step is to identify features who posess such outliers.

```R
par(mfrow=c(1,6), oma = c(1,1,0,0) + 0.1,  mar = c(3,3,1,1) + 0.1)

boxplot(residual.sugar, col="peachpuff2", pch=19)
mtext("Residual Sugar", cex=0.8, side=1, line=2)

boxplot(chlorides, col="peachpuff2", pch=19)
mtext("Chlorides", cex=0.8, side=1, line=2)

boxplot(sulphates, col="peachpuff2", pch=19)
mtext("Sulphates", cex=0.8, side=1, line=2)

boxplot(density, col="peachpuff2", pch=19)
mtext("Density", cex=0.8, side=1, line=2)

boxplot(fixed.acidity, col="peachpuff2", pch=19)
mtext("Fixed Acidity", cex=0.8, side=1, line=2)

boxplot(total.sulfur.dioxide, col="peachpuff2", pch=19)
mtext("Total SO2", cex=0.8, side=1, line=2)
```
The boxplots reveal outliers, as well as the scale of the data.  An interesting point here is additionally how little variance the Density feature is providing.

<img width="734" alt="Screenshot 2024-09-05 at 5 38 01 PM" src="https://github.com/user-attachments/assets/d9ddf701-165d-4420-9353-ed64cc40c722">

[Full Exploratory Analysis on Wine Data](./Exploratory_Wine_Dataset.html)

### Multicollinearity
Regression is sensitive to the independent variables being correlated with each other. Here we see an indicator of multicolliniarity through a Person's test.  A follow on activity would be to review the Variance Inflation Factor (VIF) for each highly correlated pair.

Review the Pearson's test for indications of highly correlated pairs.

```R
## Use Pearsons
pears_corr <- round(cor(redwine), digits = 2)
kable(pears_corr)
```
<img width="1147" alt="Screenshot 2024-09-05 at 4 51 03 PM" src="https://github.com/user-attachments/assets/581a0848-8b4b-4f86-85c0-4327d3752eff">

Each of the following pairs exhibit a correlation coefficient over .60, which begs more investigation.

```R
par(mfrow=c(2,2), oma = c(1,1,0,0) + 0.1, mar = c(3,3,1,1) + 0.1)
plot(fixed.acidity, citric.acid, main="Fixed Acidity and Citric Acid")
plot(fixed.acidity, density, main="Fixed Acidity and Density")
plot(free.sulfur.dioxide, total.sulfur.dioxide, main="Free and Total Sulfur Dioxide")
plot(pH, fixed.acidity, main="pH and Fixed Acidity")
```
Scatter plots for each of the four statistically significant correlations.

<img width="695" alt="Screenshot 2024-09-05 at 4 51 41 PM" src="https://github.com/user-attachments/assets/f49e8614-a153-4986-93c1-3265ed9c4607">



## TODO: Machine Learning
## TODO: Bayesian

<!--
src=https://html-preview.github.io/?url=https://github.com/anita-uva/anita-uva.github.io/blob/7383f755fb25c0e1cacd64ce24120cf5618cde84/Freight_Marketplace.html
-->
<!--
**anita-uva/anita-uva** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.
-->

