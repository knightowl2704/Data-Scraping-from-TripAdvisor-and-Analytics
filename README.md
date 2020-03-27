# Data-Scraping-from-TripAdvisor-and-Analytics

Data scraping using Selenium Webdriver for Chrome and Beautiful Soup.

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
