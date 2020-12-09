---
name: What's on Your Mind, Boulder?
tools: [Python, Pandas]
image: /assets/boulder_6_0.png
description: A data analysis project on three years of emails sent to the Boulder city council
---
# Whats on Your Mind, Boulder?

## Project Description

While browsing through various open data repositories, I stumbled upon one dataset in particular that intrigued me.  It contains three years of emails sent to the city council of Boulder, Colorado.  Being a fan of the show *Parks & Rec*, I was curious to know if the citizens of Boulder were just as crazy as the citizens of Pawnee.

The code below is all abridged, but you can see the full source code on GitHub.

## Let's see what our data looks like first

```python
# Ingest the email data
emails2018 = pd.read_csv('emails-2018.csv', index_col=[6])
emails2019 = pd.read_csv('emails-2019.csv', index_col=[6])
emails2020 = pd.read_csv('emails-2020.csv', index_col=[6])

emails = pd.concat([emails2018, emails2019, emails2020])

# Let's see what we're working with
print(emails.info())
emails.head()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 27732 entries, AAMkADQ2ZmVlYWI4LWI1MmEtNDc1NC05ZjhkLTI5YTA3ZDZhYWFkZABGAAAAAABKVzMXYWWETKNC5OzLgZmiBwDAPnDKzs5_QbpGrs-tvK30AAAAAAEMAADAPnDKzs5_QbpGrs-tvK30AAHQ_5gXAAA= to AAMkADQ2ZmVlYWI4LWI1MmEtNDc1NC05ZjhkLTI5YTA3ZDZhYWFkZABGAAAAAABKVzMXYWWETKNC5OzLgZmiBwDAPnDKzs5_QbpGrs-tvK30AAAAAAEMAADAPnDKzs5_QbpGrs-tvK30AALCXMumAAA=
    Data columns (total 6 columns):
     #   Column         Non-Null Count  Dtype 
    ---  ------         --------------  ----- 
     0   SentFrom       27731 non-null  object
     1   SentTo         27405 non-null  object
     2   SentCC         5376 non-null   object
     3   ReceivedDate   27732 non-null  object
     4   EmailSubject   27562 non-null  object
     5   PlainTextBody  27652 non-null  object
    dtypes: object(6)
    memory usage: 1.5+ MB
    None
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>SentFrom</th>
      <th>SentTo</th>
      <th>SentCC</th>
      <th>ReceivedDate</th>
      <th>EmailSubject</th>
      <th>PlainTextBody</th>
    </tr>
    <tr>
      <th>MessageIdentifier</th>
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
      <th>..-abridged-tvK30AAHQ_5gXAAA=</th>
      <td>Eric Johnson</td>
      <td>Council</td>
      <td>NaN</td>
      <td>2018-12-31 20:01:59.0000000 +00:00</td>
      <td>redevelopment of the Balsam BCH campus</td>
      <td>Dear council members, After reading Friday's g...</td>
    </tr>
    <tr>
      <th>..-abridged-tvK30AAHQ_5gWAAA=</th>
      <td>Jane McClannan</td>
      <td>Council</td>
      <td>NaN</td>
      <td>2018-12-31 17:26:45.0000000 +00:00</td>
      <td>OAU Unintended Consequences</td>
      <td>Dear City Council Members, I have recently spe...</td>
    </tr>
    <tr>
      <th>..-abridged-tvK30AAHQ_5gVAAA=</th>
      <td>No Reply</td>
      <td>Council</td>
      <td>NaN</td>
      <td>2018-12-31 14:09:40.0000000 +00:00</td>
      <td>Messages on hold for [6bb18bf32d5a8f2798e95f9c...</td>
      <td>The following messages, addressed to Council, ...</td>
    </tr>
    <tr>
      <th>..-abridged-tvK30AAHQ_5gUAAA=</th>
      <td>Matt Young</td>
      <td>Council</td>
      <td>NaN</td>
      <td>2018-12-31 11:47:31.0000000 +00:00</td>
      <td>Pls keep shelter open all winter</td>
      <td>Dear Council -- I urge you to accept the propo...</td>
    </tr>
    <tr>
      <th>..-abridged-tvK30AAHQ_5gTAAA=</th>
      <td>Rabbi Deborah Bronstein</td>
      <td>Council</td>
      <td>NaN</td>
      <td>2018-12-31 11:24:29.0000000 +00:00</td>
      <td>Emergency shelter all winter</td>
      <td>Dear Members of the City Council, I appreciate...</td>
    </tr>
  </tbody>
</table>
</div>

## Sentiment analysis

Are these emails from enraged citizens, similar to Pawnee?  Or do the citizens of Boulder engage in civil discourse with their city officials?  Let's do some sentiment analysis to find out.


```python
# Clean the data
emails['ReceivedDate'] = pd.to_datetime(emails['ReceivedDate'])
emails['year'] = emails.apply(lambda x: x['ReceivedDate'].year, axis=1)
emails['month'] = emails.apply(lambda x: x['ReceivedDate'].month, axis=1)
emails['week'] = emails.apply(lambda x: x['ReceivedDate'].weekofyear, axis=1)


# Do the sentiment analysis
sentimentScore = []
vader = SentimentIntensityAnalyzer()

def GetSentiment(text):
    score = vader.polarity_scores(str(text))
    if (score['compound'] > 0):
        return 'positive'
    elif score['compound'] == 0:
        return 'neutral'
    else:
        return 'negative'

emails['Sentiment'] = emails.apply(lambda row: GetSentiment(row['PlainTextBody']), axis=1)


# Plot the count of sentiment over time
emails = emails.sort_values(by=['year', 'month'])

dates = emails.groupby([emails.year, emails.month, emails['Sentiment']]).size()
dates.unstack().plot()
```


![svg](/assets/boulder_4_0.svg)


## Who emails the most?

Next let's find out who's emailing city council the most.  I plan to better visualize most of this data in Tableau later, but for now we can use matplotlib to display the top 25 senders.

```python
squeaky_wheels = emails.groupby([emails.SentFrom]).size().sort_values(ascending=False).head(25)

squeaky_wheels.plot(kind='bar')
plt.show()
```


![svg](/assets/boulder_5_0.svg)

## Wordcloud

Next let's see what's on the mind of Boulder residents with a wordcloud.

```python
from wordcloud import WordCloud, STOPWORDS, ImageColorGenerator

# Get the text into a long string
text = emails.PlainTextBody.tolist()
text = ' '.join(map(str, text))

# use darker colors that are easier to read
cmap = mpl.cm.Blues(np.linspace(0,1,20))
cmap = mpl.colors.ListedColormap(cmap[-10:,:-1])

wordcloud = WordCloud(width=550, height=250, max_words=100, colormap=cmap, contour_width=4,
  background_color='#f0f7fa', stopwords=STOPWORDS, collocations=True, min_word_length=3).generate(text)

# increase the plot size
plt.figure(figsize=(5.5, 2.5), frameon=False)

plt.imshow(wordcloud, interpolation='bilinear')
plt.axis('off')
plt.tight_layout(pad=0)
plt.show()
plt.savefig('wordcloud.png')
```


![svg](/assets/boulder_6_0.svg)