# Data-Scraping-from-TripAdvisor-and-Analytics

<h2> Data scraping using Selenium Webdriver for Chrome and Beautiful Soup. </h2>

**Problem Statement**: Scraping the Reviews from tripAdvisor webpage for a particular restaurant along with the review date, title and rating. 

Then use this prepared dataset for sentiment analysis using languare processing.

Since the TripAdvisor webpage is *dynamic* in nature, scraping only with BeautifulSoup would result in partial scraping of reviews. So we need to 
somehow expand the reviews and then use the page source with BeautifulSoup for scraping the complete review.

In order to expand these reviews, I am using Selenium Webdriver with Python to mimic the action of a user(clicking the *more* button).
In order to scrape all the reviews from around 20 different pages, the url of the webpage needs to be changed. On observation it is observed that
the url just changes by *or20* to *or30* after going from page 2 to page 3, so we need to figure out the total number of pages and we can then
loop through all the pages for scraping further data.

**Clicking the more button on the webpage to expand reviews**
```
driver.get(url)
    time.sleep(2)
    more_buttons = driver.find_element_by_xpath("//span[@class='taLnk ulBlueLinks' and text()='More']").click()
    print('clicked')
```

**Once the buttons are clicked, we can use the pagesource with BeautifulSoup to scrape required data.**
```
    page_source = driver.page_source
    soup = BeautifulSoup(page_source, 'lxml')
```
**Parsing the data**
```
    results = []
    for idx, review in enumerate(soup.find_all('div', class_='review-container')):
            item = {
                'bubble_rating': review.select_one('div.ui_column span.ui_bubble_rating')['class'][1][7:8],
                'review_date': review.find('span', class_='ratingDate')['title'],
                'review_paragraph': review.find('p', class_='partial_entry').text,
                'review_title': review.find('span', class_='noQuotes').text,
                'site': 'TripAdvisor'
            }
            results.append(item)
```
I have hardcoded the 'site' for this dataset, but it can be scraped from the url using regex. Getting the ratings part was tricky, but I
scraped the class name and sliced it to obtain the last digit.

Now we need to know the total number of pages that need to be scraped.

**Making a simple request and parsing the soup to get the last page number.**


```
page = requests.get('https://www.tripadvisor.in/Restaurant_Review-g304551-d13388460-Reviews-or00-Kitchen_With_A_Cause-New_Delhi_National_Capital_Territory_of_Delhi.html')
soup = BeautifulSoup(page.text, 'html.parser')
lastpageno = int(soup.find_all('a',class_ = 'pageNum')[-1].text)
```

**Looping through the urls and saving the data to pandas DataFrame.**
```
df = pd.DataFrame()
for i in range(0,lastpageno):
    url = 'https://www.tripadvisor.in/Restaurant_Review-g304551-d13388460-Reviews-or'+str(i)+'0-Kitchen_With_A_Cause-New_Delhi_National_Capital_Territory_of_Delhi.html'
    final_results = []
    soup = get_complete_page(url)
    result1 = parse_data(soup)
    print(result1)
    df = df.append(result1)
```

Notice how the urls just differ in the *or10*,*or20* part.

<h2> Data and Sentiment Analysis with Natural Language Processing. </h2>

**Problem Statement**: Cleaning the obtained dataset and using natural language processing techniques for data and sentiment analysis. 

**Cleaning the reviews**

```
def clean_review():
    for i in range(0,len(df['review_paragraph'])):
        review = re.sub('[^a-zA-Z]', ' ', df['review_paragraph'][i])  
        review = review.lower()  
        review = review.split()  
        review = [w for w in review if not w in stopwords]
        porter = PorterStemmer()
        review = [porter.stem(word) for word in review]
        review = ' '.join(review)   
        corpus.append(review) 
```
Created a corpus of reviews with stopwords removed, cases lowered, and the words stemmed. 

**Most Frequent words in the reviews**
```
from sklearn.feature_extraction.text import CountVectorizer
vec = CountVectorizer().fit(corpus)
bag_of_words = vec.transform(corpus)
sum_words = bag_of_words.sum(axis=0) 
words_freq = [(word, sum_words[0, idx]) for word, idx in vec.vocabulary_.items()]
words_freq =sorted(words_freq, key = lambda x: x[1], reverse=True)
```
Used *CountVectorizer* to create a simple bag-of-words model and then formed an array of tuples of word and its frequency. 
The top 10 words were as follows: 
```
[('food', 240),
 ('restaur', 136),
 ('great', 124),
 ('good', 109),
 ('place', 99),
 ('servic', 83),
 ('caus', 80),
 ('staff', 79),
 ('visit', 77),
 ('delhi', 71)]
 ```

**Sentiment Analysis using  vaderSentiment's SentimentIntensityAnalyser**

For each of the reviews, calculated the polarity score, which returns an object as follows : 
```
sentiment_dict = {
            "neg": value,
            "neu": value,
            "pos": value,
            "compound": value,
        }
```

Used the following thresholds for determining the Polarity of the review. Also added a new column with Positive Score for each review.

```
        if score['compound'] >= 0.05 : 
            new_df['Polarity'][i] = "Positive" 
        elif score['compound'] <= -0.05 : 
            new_df['Polarity'][i] = "Negative"
        else: 
            new_df['Polarity'][i] = "Neutral"
```

This newly created *Positive Score* column will help us in determining the most positive review and most negative review. The review with *max()* positive score is the most positive review and with *min()* positive score is the most negative review.

Check the notebook, `MWG_Analysis.ipnyb` for more analysis on reviews, ratings and to find out which is the most positive review and most negative review.


