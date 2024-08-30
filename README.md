# Data Science Highlights

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

### Web Scraping / Data Collection:  Beautiful Soup simple example
Here is a short example of web scraping with Beautiful Soup.

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

### Data Transformation
Maybe some python with pandas

## Exploratory Data Analysis
Exploratory Data Analysis goes hand in hand with Data Collection and Cleaning.

### Feature Reduction
### Summary Statistics
### Correlation

## Machine Learning

<!--
src=https://html-preview.github.io/?url=https://github.com/anita-uva/anita-uva.github.io/blob/7383f755fb25c0e1cacd64ce24120cf5618cde84/Freight_Marketplace.html
-->
<!--
**anita-uva/anita-uva** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.
-->
