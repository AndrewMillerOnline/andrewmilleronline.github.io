---
name: Marathon Viz
title: Marathon Viz
tools: [Python, Pandas, Jupyter Notebooks, R, ggplot]
image: /assets/marathon/kenyan-metals.png
description: Data analysis and visualization project for a master's degree course
order: 1
---

# Project Description

For the final project in my data visualization course as part of the MS Data Science program at DePaul, I had to create complex visualizations from the dataset of my choice.  As an avid runner, naturally I wanted to visualize something running related.  Unable to find a suitable dataset, I instead created my own using Python to scrape over 2 million marathon finisher results from four of the largest marathons in the world.  I then used R and ggplot to create visualizations from this data.

## Data Gathering

To build the dataset, I used Python to scrape individual finisher results from the websites of the Berlin, New York City, Chicago, and London marathons.

Each website was distinct enough that it required its own scraper, however the four websites generally fell into two different categories: plain HTML-based results determined by url parameters, and Javascript-based results powered by a front-end framework such as Angular.

### Plain HTML

For the plain HTML pages, once I identified the relevant query string parameters used to generate the list of results, gathering the data was a simple matter of requesting each page of results and then parsing the HTML with BeautifulSoup.

However, each of these races had ~ 50,000 finishers per year, and the results websites were limited to between 50 and 1,000 finisher results per page.  That translated to a huge number of requests, and a lot of time spent scraping.  To speed this up, I was able to use the concurrent.futures library to employ multi-threading and significantly decrease the amount of time required.

See the [Chicago scraper on GitHub](https://github.com/AndrewMillerOnline/marathon-scrapers/blob/main/Chicago/chicago.ipynb) for an example.

### Javascript-based

For the JS-based websites, I needed to use Selenium webdriver in order to simulate a user interacting with the browser.  This was fairly straightforward, although I did encounter some problems with timeouts.  To overcome this, I implemented some simple error-catching that would tell the scraper to go back a page when it reached an unexpected error.  Doing so would send a new request to the backend, and generally fix whatever problem had caused the temporary timeout.

Here's a short snippet of the error handling:

```python
#loop through each page and gather the results
num_error = 0
while num_error < 5:
    
    try:
        try_do_scrape(current_page, last_page)
        
    except TimeoutException:
        current_page = int(driver.find_element_by_xpath("//li[@class='paginate_button page-item active']").text)
        #TimeoutException is always thrown on the final page
        #If we're on the final page, quit the loop and save
        if current_page == last_page:
            break
            
    except (UnexpectedAlertPresentException, StaleElementReferenceException) as ex:
        print(f"**Encountered exception #{num_error+1}**")
        print(ex)
        time.sleep(1)
        
        current_page = int(driver.find_element_by_xpath("//li[@class='paginate_button page-item active']").text)
        
        
        if current_page == last_page:
            break
        #Go back a page and start over (we'll dedupe during cleanup)
        current_page -= 1
        prev_button = driver.find_element_by_xpath("//a[text()='Previous']")
        prev_button.send_keys(Keys.ENTER)
        driver.execute_script("arguments[0].scrollIntoView();", select_element)
        num_error += 1
    
cleanup_and_save(year)
```

Want to see the full code?  Check out the [Berlin scraper on GitHub](https://github.com/AndrewMillerOnline/marathon-scrapers/blob/main/Berlin/berlin.ipynb).

### Finishing Up
After I gathered all the results, I did a small amount of normalizing to match up the column names and saved them all into one big CSV.  I also gathered weather-related data (max temp, min temp, and precipitation) from a NOAA website and augmented all of the results with race-day weather for the race location.

## Data Viz

Now that I had a dataset, the real work began (this was for a data viz course, after all).

### EDA

One downsides to my decision to build my own dataset is that my dataset was DIRTY.  But this is the real world, right?  All data in the real world is dirty.  Here are a few EDA visualizations to show the differences in data provided by the race websites.

![eda1](/assets/marathon/eda1.png)
![eda2](/assets/marathon/eda2.png)

### Get To The Real Viz, Man

Ok, that small caveat about the data quality is out of the way, so let's move on to the viz.  I posed a number of questions to the data, and these are the results I got.

### Which Race Has the Fastest Finish Times?
![eda2](/assets/marathon/fastest.png)

It looks like Berlin has the fastest finish times!  I wonder why that is...Could the weather have anything to do with it?

### How Does Weather Impact Finish Time?
![weather](/assets/marathon/chicago-weather.png)
![weather](/assets/marathon/berlin-weather.png)

Ok, the weather most definitely impacts finish time!  Look at Chicago around 2007, ouch.  Both the daily high temp and the daily low look like they affect runners.  Berlin looks like it gets just as hot as Chicago, but the lows appear lower.  How consistent is that?

### Which Race Has the Most Predictable Weather?
![weather](/assets/marathon/weather-box.png)

We can see in this graph that New York City has the least variability in both daily highs and daily lows.  However, there are only six years of data for New York, so we must be careful in assuming that New York has the best weather.  Berlin and Chicago both have identical median high and low temperatures, but Berlin clearly has less variability than Chicago.

### Pacing Strategies
Next, let's look at pacing strategies.  I performed a calculation based on each runner's half-marathon split and finish times to classify each runner as either a negative split (faster second half than first half), positive split (slower second half), or even split.  For runners to be classified as an even split, I allowed a 10% margin, where each half could be up to 5% slower or 5% faster than the other.

I then posed some questions of the pacing strategy data.

### Which Pacing Strategy is most Common?
![splits](/assets/marathon/splits.png)
This is one of the most surprising graphs to me!  It's so common in running circles to hear that negative splits are "best".  But looking at the data, they're certainly not the most common!  I wonder how a runner's fitness impacts their pacing strategy?

### How Does Pacing Strategy Vary Based on a Runner's Overall Speed?

![weather](/assets/marathon/split-speeds.png)
Oh, now this is very interesting.  The proportion of runners doing negative splits appears to be fairly consistent, regardless of their finish time (which is what the quartiles are based on).  However, as runners get slower, the proportion of positive splitters increases.  This makes intuitive sense for a few reasons -- but let's see how the weather impacts pacing strategy.

### How Does Pacing Strategy Differ Based on the Weather?

![weather](/assets/marathon/split-weather.png)
This is somewhat of a unique graph choice, inspired by a mosaic plot.  For each year, the graph displays the percent of runners who employed a positive split versus those who employed an even split.  As we can see, on cooler years the percent or runners who run even splits increases.  However, the scorching Chicago sun takes its toll in hotter years, and significantly more runners end up running a slower second half of the race due to heat.

## Bonus Viz!

Here's a waffle chart depicting the dominance of Kenyan men at the Berlin marathon.

![weather](/assets/marathon/kenyan-metals.png)

Finally, here's an animated viz to show when Kipchoge breaks away from the competition during the 2018 Berlin marathon (when he set a new World Record of 2:01:39).  To accomplish this graph, I took the top 5 runners from Berlin that year and at each split calculated how far they were behind Kipchoge.  Then I pivoted that data so I only had three columns: Name, Split, and TimeBehindLeader.  I used those variables with ggplot and gganimate to create the below gif.

![weather](/assets/marathon/eliud.gif)