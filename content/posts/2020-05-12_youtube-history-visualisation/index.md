---
title: "Youtube History Visualisation in Plotly"
date: 2020-05-11T21:48:47+01:00
featuredImage: "/posts/2020-05-12_youtube-history-visualisation/header_image.png"
featuredImagePreview: "/posts/2020-05-12_youtube-history-visualisation/header_image_preview.png"
tags: ["python", "visualisation", "plotly"] 
categories: ["Software Engineering"]
draft: false
---

The following learning project is intended to help me learn concepts around the steps required to create a visualisation in python, but also explore the kind of information that is stored by companies such as YouTube (Google).

## Objectives

* **Data Preparation:** Data preparation makes up ~80% of the work in a Data Science pipeline. I'd like to explore the best techniques to go from raw CSV data, to a tabular format that is ready for data visualisation.
* **Plot:** There are various modules that can be used to plot the data. I'll consider which is best for this use case, and find the best way to represent the data in hand.
* **Web Display:** In order to display this project, what is the best way to allow users to upload their YouTube data and display the visualisation such that is can be deployed on a server with minimal effort to others.

## Preprocessing

### Getting access to YouTube Data

YouTube depricated their history API due to privacy reasons a few years ago, however it's possible to download a copy of your YouTube history in a JSON format by doing the following:

1. Go to [Google Takeout](https://takeout.google.com/)
2. Deselect All 
3. Select YouTube
4. Underneath YouTube, click on "All YouTube Data Included"
5. Deselect All
6. Select history

You can then download your YouTube data after it is generated and a link sent to you. You should get a file called `watch-history.json` that we'll be making use of in future steps.

### Data Preparation

We can take a look at the JSON and see what there is:


```python
import json
import pandas as pd

# Open History File as dataframe
file = '../watch-history.json'
with open(file, encoding='utf8') as wh_file:
    wh_dict = json.load(wh_file)
    
wh = pd.DataFrame.from_dict(wh_dict)

display(wh)
```

<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>description</th>
      <th>header</th>
      <th>products</th>
      <th>subtitles</th>
      <th>time</th>
      <th>title</th>
      <th>titleUrl</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCFK...</td>
      <td>2019-05-04T15:51:04.397Z</td>
      <td>Watched Stromae - Papaoutai (Clip Officiel)</td>
      <td>https://www.youtube.com/watch?v=oiKj0Z_Xnjc</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCbs...</td>
      <td>2019-05-04T15:16:07.495Z</td>
      <td>Watched Me My Body And I</td>
      <td>https://www.youtube.com/watch?v=tCbPpfIGk-s</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCVL...</td>
      <td>2019-05-04T14:24:33.373Z</td>
      <td>Watched I Did Peloton For Two Weeks Straight A...</td>
      <td>https://www.youtube.com/watch?v=erqLKwwZCVE</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCdd...</td>
      <td>2019-05-04T14:10:05.115Z</td>
      <td>Watched F8 2019 keynote in 12 minutes</td>
      <td>https://www.youtube.com/watch?v=UtxPdezclYw</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCw0...</td>
      <td>2019-05-04T13:56:17.096Z</td>
      <td>Watched Netflix culture deck via Reed Hastings</td>
      <td>https://www.youtube.com/watch?v=2fuOs6nJSjY</td>
    </tr>
    <tr>
      <th>5</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCWO...</td>
      <td>2019-05-04T13:36:24.194Z</td>
      <td>Watched Tuca &amp; Bertie | Official Trailer [HD] ...</td>
      <td>https://www.youtube.com/watch?v=ZybYIJtbcu0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCw0...</td>
      <td>2019-05-04T13:36:04.262Z</td>
      <td>Watched Netflix Interview (1 of 3): 2018 Cultu...</td>
      <td>https://www.youtube.com/watch?v=8-4is78UJZ8</td>
    </tr>
    <tr>
      <th>7</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UC7Y...</td>
      <td>2019-05-03T22:20:56.255Z</td>
      <td>Watched "You Make Me Sick So Much, Jimmy Carr"...</td>
      <td>https://www.youtube.com/watch?v=Q8BeTjCjmwQ</td>
    </tr>
    <tr>
      <th>8</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCes...</td>
      <td>2019-05-03T22:20:08.582Z</td>
      <td>Watched How to Install a Hidden Kill Switch in...</td>
      <td>https://www.youtube.com/watch?v=XUhXLsrZiE0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCp0...</td>
      <td>2019-05-03T22:19:19.939Z</td>
      <td>Watched John Bradley Doesn’t Know What He Know...</td>
      <td>https://www.youtube.com/watch?v=qPHlaLti3g0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>9015</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCYu...</td>
      <td>2013-05-30T13:02:43.993Z</td>
      <td>Watched Casio hint 5 Normal Probabilities</td>
      <td>https://www.youtube.com/watch?v=E5LViG-gmMQ</td>
    </tr>
    <tr>
      <th>9016</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCYu...</td>
      <td>2013-05-30T13:01:22.800Z</td>
      <td>Watched casio hint 4 Integration q7 p444</td>
      <td>https://www.youtube.com/watch?v=rVD4WN7k_9Q</td>
    </tr>
    <tr>
      <th>9017</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCYu...</td>
      <td>2013-05-30T12:56:12.477Z</td>
      <td>Watched casio hint 3 binomial distribution</td>
      <td>https://www.youtube.com/watch?v=CqcOXW16xJg</td>
    </tr>
    <tr>
      <th>9018</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCYu...</td>
      <td>2013-05-30T12:49:49.775Z</td>
      <td>Watched casio hint 2 unbiased estimates</td>
      <td>https://www.youtube.com/watch?v=N6kJzxNpUX4</td>
    </tr>
    <tr>
      <th>9019</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCYu...</td>
      <td>2013-05-30T12:45:31.681Z</td>
      <td>Watched casio hint 1 graphing</td>
      <td>https://www.youtube.com/watch?v=8nMi8wdzC00</td>
    </tr>
    <tr>
      <th>9020</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCYu...</td>
      <td>2013-05-30T12:36:29.574Z</td>
      <td>Watched Casio hint 9 Contingency Tables</td>
      <td>https://www.youtube.com/watch?v=Bc3dAcwCNw0</td>
    </tr>
    <tr>
      <th>9021</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>NaN</td>
      <td>2013-05-30T11:32:30.934Z</td>
      <td>Watched https://www.youtube.com/watch?v=Pl1UMf...</td>
      <td>https://www.youtube.com/watch?v=Pl1UMfxa_r4</td>
    </tr>
    <tr>
      <th>9022</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCe8...</td>
      <td>2013-05-23T19:51:15.893Z</td>
      <td>Watched Microsoft fails again (1998-2012)</td>
      <td>https://www.youtube.com/watch?v=2w8rypZQDbg</td>
    </tr>
    <tr>
      <th>9023</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCdd...</td>
      <td>2013-05-23T19:44:58.295Z</td>
      <td>Watched Microsoft Surface Keynote</td>
      <td>https://www.youtube.com/watch?v=jozTK-MqEXQ</td>
    </tr>
    <tr>
      <th>9024</th>
      <td>NaN</td>
      <td>YouTube</td>
      <td>[YouTube]</td>
      <td>[{'url': 'https://www.youtube.com/channel/UCi8...</td>
      <td>2013-05-23T16:56:22.071Z</td>
      <td>Watched Don Jon Official Trailer #1 (2013) - J...</td>
      <td>https://www.youtube.com/watch?v=6615kYTpOSU</td>
    </tr>
  </tbody>
</table>

There are some parts we'd ike to make use of such as `title` that show some useful information, and others like `titleUrl` which in this project, we won't have any use for. First, we can drop everything but `title` and `time` as we don't need anything else.


```python
# Drop columns except title and time
wh = wh[['title', 'time']]
```

We can see clearly that "Watched" is being prepended to the `title` field. Not only this, but where a video has become unavailable, we get a title like "Watched https://www.youtube.com/watch?v=rG5tV7zcl1s". Let's take a look at the percentage of videos that are no longer available in the dataset


```python
100 * len(wh[wh['title'].str.startswith('Watched https://www.youtube.com')]) / len(wh)
5.473684210526316
```

It's about 5%. Not too bad. Let's remove the prefix "Watched" from the titles and also drop all the unavailable videos. We'll also remove whitespace and make everything lower case for simplicity.

```python
import string

# Remove "Watched" from title "Watched Cat Video" -> "Cat Video"
wh['title'] = wh['title'].apply(lambda x: x[7:])

# Remove Unavailable videos
wh = wh.drop(wh[wh['title'].str.startswith('https://www.youtube.com')].index)

# Lower Case "Cat Video" -> "cat video"
wh['title'] = wh['title'].apply(lambda x: x.lower())

# Remove whitespace
wh['title'] = wh['title'].apply(lambda x: x.strip())
wh['title'] = wh['title'].apply(
    lambda x: x.translate(str.maketrans(' ', ' ', string.punctuation)))

wh.head()
```

<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>stromae  papaoutai clip officiel</td>
      <td>2019-05-04T15:51:04.397Z</td>
    </tr>
    <tr>
      <th>1</th>
      <td>me my body and i</td>
      <td>2019-05-04T15:16:07.495Z</td>
    </tr>
    <tr>
      <th>2</th>
      <td>i did peloton for two weeks straight and here’...</td>
      <td>2019-05-04T14:24:33.373Z</td>
    </tr>
    <tr>
      <th>3</th>
      <td>f8 2019 keynote in 12 minutes</td>
      <td>2019-05-04T14:10:05.115Z</td>
    </tr>
    <tr>
      <th>4</th>
      <td>netflix culture deck via reed hastings</td>
      <td>2019-05-04T13:56:17.096Z</td>
    </tr>
  </tbody>
</table>

We only want the date from the `time` field, as the time is too granular for this particular visualisation. Let's convert this attribute to just contain a `datetime` object with the date

```python
import datetime as dt

def get_date(datetime_str):
    return dt.datetime.strptime(datetime_str.split('T')[0], '%Y-%m-%d')

wh['time'] = wh['time'].apply(lambda x: get_date(x))

wh.head()
```

<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>stromae  papaoutai clip officiel</td>
      <td>2019-05-04</td>
    </tr>
    <tr>
      <th>1</th>
      <td>me my body and i</td>
      <td>2019-05-04</td>
    </tr>
    <tr>
      <th>2</th>
      <td>i did peloton for two weeks straight and here’...</td>
      <td>2019-05-04</td>
    </tr>
    <tr>
      <th>3</th>
      <td>f8 2019 keynote in 12 minutes</td>
      <td>2019-05-04</td>
    </tr>
    <tr>
      <th>4</th>
      <td>netflix culture deck via reed hastings</td>
      <td>2019-05-04</td>
    </tr>
  </tbody>
</table>

### Feature Engineering

Now we have the dataframe with the important information from the original history JSON we downloaded. We're now going to add the following features in order to get a better representation of the data. We have a large collection of titles, but the title "Body Workout Exercise" and "Full Body Workout" have very similar themes, but are treated as distinct occurances unless we find some way to group entries with similar themes. We need a way to:

* Extract key themes from a video title
* Find other videos that have this theme

#### Keyword Analysis

In order to get aggregated meaning of the titles, we can look at the keywords in the titles.

##### Tokenizing 

We could find the distribution of keywords such as "XBOX" or "Full-body Workout" over the range of the dataset. Notice that keywords could appear in the form of unigrams (one word) or bigrams (two words together) such as the latter example. We also want to remove stop words such as "and" and "with" in our tokens, as these won't provide any additional value to this example.


```python
import nltk
from nltk.corpus import stopwords

nltk.download('stopwords')
stop_words = stopwords.words('english')

def find_ngrams(input_list, n):
    if n > 1:
        return zip(*(input_list[i:] for i in range(n)))
    else:
        return input_list

# Split words and remove stopwords
wh['unigrams'] = wh['title'].apply(lambda x: [word for word in x.split(' ') if word not in stop_words and word != ''])
wh['bigrams'] = wh['unigrams'].apply(lambda x: list(find_ngrams(x, 2)))
wh.head()
```

<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>time</th>
      <th>unigrams</th>
      <th>bigrams</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>stromae  papaoutai clip officiel</td>
      <td>2019-05-04</td>
      <td>[stromae, papaoutai, clip, officiel]</td>
      <td>[(stromae, papaoutai), (papaoutai, clip), (cli...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>me my body and i</td>
      <td>2019-05-04</td>
      <td>[body]</td>
      <td>[]</td>
    </tr>
    <tr>
      <th>2</th>
      <td>i did peloton for two weeks straight and here’...</td>
      <td>2019-05-04</td>
      <td>[peloton, two, weeks, straight, here’s, happened]</td>
      <td>[(peloton, two), (two, weeks), (weeks, straigh...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>f8 2019 keynote in 12 minutes</td>
      <td>2019-05-04</td>
      <td>[f8, 2019, keynote, 12, minutes]</td>
      <td>[(f8, 2019), (2019, keynote), (keynote, 12), (...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>netflix culture deck via reed hastings</td>
      <td>2019-05-04</td>
      <td>[netflix, culture, deck, via, reed, hastings]</td>
      <td>[(netflix, culture), (culture, deck), (deck, v...</td>
    </tr>
  </tbody>
</table>

By creating a bag of words, we can explore the frequency of the unigrams and bigrams that are appearing in the dataset.


```python
from collections import Counter

# Create N-Gram Counters
def get_counter_from_column(df, column_name):
    ct = Counter()
    for row in df[column_name]:
        for element in row:
            ct[element] += 1
    return ct

bag_1 = get_counter_from_column(wh, 'unigrams')
bag_2 = get_counter_from_column(wh, 'bigrams')

print("Bag 1\n", bag_1.most_common(10))
print("Bag 2\n", bag_2.most_common(10))
```
    Bag 1
    [('video', 811), ('official', 606), ('trailer', 283), ('removed', 239), ('music', 218), ('1', 205), ('2', 196), ('hd', 190), ('life', 176), ('vs', 173)]
    Bag 2
    [(('video', 'removed'), 239), (('official', 'video'), 209), (('music', 'video'), 154), (('official', 'music'), 137), (('official', 'trailer'), 135), (('movie', 'hd'), 53), (('trailer', '1'), 53), (('trailer', 'hd'), 43), (('glass', 'animals'), 43), (('part', '1'), 42)]

##### Combine Pluralisms

We can see from the following code snippet that a singular word's plural can appear in the bag of words together. Semantically they mean the same thing, therefore we should find a way to combine these instances. In this example, we're taking every regular noun (You just have to add s to pluralise) that has both an occurence of itself and its plural in our unigrams. We then pick one of these with the most occurances and then replace this in both the bag of words and the dataset we're going to use to visualise the data.

```python
print("movies: {}".format(bag_1['movies']))
print("movie: {}".format(bag_1['movie']))
```

    movies: 8
    movie: 105
    



```python
# Pick singular or plural for most common 500 words
def replace_count(counter, removed, added):
    counter[added] += counter[removed]
    del counter[removed]
    
remove_words = []
for word in bag_1.most_common(1000):
    word = word[0]
    if word.endswith('s'):
        singular = word[:-1]
        plural = word
    else:
        singular = word
        plural = word + 's'
    if plural in bag_1 and singular in bag_1:
        if bag_1[plural] >= bag_1[singular]:
            remove_words.append((singular, plural))
        else:
            remove_words.append((plural, singular))

for (removed, added) in remove_words:
    replace_count(bag_1, removed, added)
    wh['unigrams'] = wh['unigrams'].apply(lambda x: [added if unigram == removed else unigram for unigram in x])
    
bag_1.most_common(10)
```

Now you can see that for the key unigrams in our dataset, we have picked either the singular form or pluralised form for our regular nouns.


    [('video', 833),
     ('official', 606),
     ('trailer', 294),
     ('removed', 239),
     ('new', 223),
     ('music', 218),
     ...
     ('movie', 113)]

##### Duplicates in bigrams


```python
print(bag_1['glass'])
print(bag_1['animals'])
print(bag_2[('glass', 'animals')])
```

    51
    53
    43



We now also have another problem to address. If we were to combine our unigrams and bigrams now and look at the distributions of each keyword, we'd get duplicates. For example the British band "Glass Animals" appears 43 times. "glass" and "animals" occurances are largely contributed by the band's name. If we assume that if a singular word's occurs less than three times much as the respective bigram does, then it's contribution was most likely due to the bigram. Therefore we can remove it. Likewise, if the bigram appears less than a third as many times as the respective unigram, then we can remove it from the dataset.

While this is more anecdotal, this was able to extract some more of the key bigrams that I knew where in the dataset. Obviously this requires more rigor to have reproducable outcomes for other datasets.


```python
THRESHOLD = 0.3
for bigram in bag_2.most_common(1000):
    # print("{}: {}".format(bigram[0][0], bag_1[bigram[0][0]] * 0.75))
    if (bag_1[bigram[0][0]] * THRESHOLD) <= bag_2[bigram[0]]:
        del bag_1[bigram[0][0]]
        if (bag_1[bigram[0][1]] * THRESHOLD) <= bag_2[bigram[0]]:
            del bag_1[bigram[0][1]]
    else:
        del bag_2[bigram[0]]
print(bag_1[('video',)])
wh['ngrams'] = wh['unigrams'] + wh['bigrams']
bag_1_2 = (bag_1 + bag_2)
```

Now that we've removed a lot of these duplicate themes, we can combine the bag of words as `bag_1_2`

#### Visualisation Format

We now have in our `bag_1_2` a collection of n-grams we'd like to investigate in our visulisation. Let's consider taking the most frequent `N` from this collection, and finding the mean date that they occur. By doing this, we can determine the general time of the year that videos with this feature appeared.

For this, we need to take a list of datetimes and return the average of those.


```python
import functools
import operator
from datetime import timedelta
def avg_datetime(series):
    dt_min = series.min()
    deltas = [(x - dt_min).days for x in series]
    if len(deltas) == 0:
        print(series)
    return dt_min + timedelta(functools.reduce(operator.add, deltas) // len(deltas))
```

For each of these most common n-grams, let's create a new dataframe that stores the following attributes: `{keyword, average datetime, frequency of keyword}`.


```python
N = 100
data = pd.DataFrame([], columns=['keyword', 'avg_datetime', 'freq'])
for ngram in list(bag_1_2.most_common(N)):
    keyword = ngram[0]
    dates = wh[wh['ngrams'].apply(lambda x: (True if keyword in x else False))]['time']
    data = data.append({
        'keyword': keyword if isinstance (keyword, str) else " ".join(keyword),
        'avg_datetime': avg_datetime(dates),
        'freq': len(dates)
    }, ignore_index=True)
```

<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>keyword</th>
      <th>avg_datetime</th>
      <th>freq</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>video</td>
      <td>2016-08-17</td>
      <td>825</td>
    </tr>
    <tr>
      <th>1</th>
      <td>removed</td>
      <td>2015-11-23</td>
      <td>239</td>
    </tr>
    <tr>
      <th>2</th>
      <td>new</td>
      <td>2017-04-05</td>
      <td>212</td>
    </tr>
    <tr>
      <th>3</th>
      <td>official video</td>
      <td>2016-10-05</td>
      <td>209</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1</td>
      <td>2016-09-18</td>
      <td>198</td>
    </tr>
  </tbody>
</table>


As you can see, some of the most popular keywords relate to common suffixs of music videos or indications that the video was removed by the uploader. Therefore we'll remove a few different keywords that have no useful meaning such as "video", "official video", and numbers.


```python
REMOVALS = []
REMOVALS.extend([str(x) for x in list(range(100))])
REMOVALS.extend(["one", "two", "three", "four", "five", "six", "seven", "eight", "nine", "ten"])
REMOVALS.extend(["video", "trailer", "new", "best", "official", "removed", "music", "ft", "feat"])
REMOVALS.extend(['official video', 'official trailer', 'music video', 'official music'])
data = data.drop(data[data['keyword'].isin(REMOVALS)].index)
```

<table>
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>keyword</th>
      <th>avg_datetime</th>
      <th>freq</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>6</th>
      <td>vs</td>
      <td>2016-12-05</td>
      <td>168</td>
    </tr>
    <tr>
      <th>7</th>
      <td>hd</td>
      <td>2015-09-25</td>
      <td>187</td>
    </tr>
    <tr>
      <th>8</th>
      <td>day</td>
      <td>2017-07-18</td>
      <td>186</td>
    </tr>
    <tr>
      <th>9</th>
      <td>life</td>
      <td>2017-05-25</td>
      <td>175</td>
    </tr>
    <tr>
      <th>13</th>
      <td>tutorial</td>
      <td>2017-02-08</td>
      <td>131</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2018</td>
      <td>2018-07-29</td>
      <td>133</td>
    </tr>
    <tr>
      <th>17</th>
      <td>time</td>
      <td>2017-02-22</td>
      <td>117</td>
    </tr>
    <tr>
      <th>19</th>
      <td>coldplay</td>
      <td>2018-01-02</td>
      <td>109</td>
    </tr>
    <tr>
      <th>22</th>
      <td>get</td>
      <td>2017-03-09</td>
      <td>98</td>
    </tr>
    <tr>
      <th>23</th>
      <td>google</td>
      <td>2017-08-16</td>
      <td>98</td>
    </tr>
    <tr>
      <th>24</th>
      <td>year</td>
      <td>2017-09-06</td>
      <td>98</td>
    </tr>
    <tr>
      <th>25</th>
      <td>world</td>
      <td>2016-08-15</td>
      <td>95</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>94</th>
      <td>workout</td>
      <td>2018-01-22</td>
      <td>44</td>
    </tr>
    <tr>
      <th>95</th>
      <td>us</td>
      <td>2017-02-25</td>
      <td>44</td>
    </tr>
    <tr>
      <th>96</th>
      <td>top</td>
      <td>2016-10-01</td>
      <td>44</td>
    </tr>
    <tr>
      <th>97</th>
      <td>lets</td>
      <td>2017-01-19</td>
      <td>44</td>
    </tr>
    <tr>
      <th>98</th>
      <td>bad</td>
      <td>2016-07-13</td>
      <td>44</td>
    </tr>
    <tr>
      <th>99</th>
      <td>trailer hd</td>
      <td>2016-10-05</td>
      <td>43</td>
    </tr>
  </tbody>
</table>

## Visualisation

Now that we have the data in the format we want, we can visualise it. Originally I thought that displaying each theme in a scatter graph would be good, however having to hover over each point to see what the theme was wasn't quite so effective. Instead I decided to create a word cloud that had the following design:

* Each word is centered to align with the average occurance of that theme
* The size of the word is proportional to the frequency that the theme occured in the dataset

I decided to use [Plotly](https://plot.ly/) for this visualisation as it has great options for going beyond the visualisation in Jupyter. They have a program called Dash that allows you to create interactive dashboards within Python, and their visualisations are serialised in JSON which would make integration with an API easier.

```python
# Plot
import random
import plotly.offline as py
import plotly.graph_objs as go


palette = ['darkturquoise', 'darkorange', 'darkorchid', 'mediumseagreen', 'royalblue', 'saddlebrown', 'tomato']
plotly_colors = [palette[random.randrange(0, len(palette))] for i in range(N)]

trace = go.Scatter(
    x = data["avg_datetime"],
    y = [random.uniform(0, 1) for x in range(len(data))],
    mode = "text",
    text = [x.upper() for x in data["keyword"]],
    opacity=0.75,
    textfont={
        'size': [x // 3.5 for x in list(data["freq"])],
        'color': plotly_colors,
        'family': "'Oswald', sans-serif"
    }
)

plot_data = [trace]
fig = go.FigureWidget(data=plot_data)
py.iplot(fig)
```

This generates the following visualisation. You can get a pretty good idea of my taste in music, the games I used to play when I was a teenager, and my recent interests in veganism and politics.


<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="/posts/2020-05-12_youtube-history-visualisation/youtube-histviz-01.html" height="750" width="100%"></iframe>

[View Fullscreen](/posts/2020-05-12_youtube-history-visualisation/youtube-histviz-01.html)

Similarly I gave it a shot with a friend to see how it might compare with them. As you can see below, they mostly use YouTube to watch comedy and listen to music. 

<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="/posts/2020-05-12_youtube-history-visualisation/youtube-histviz-02.html" height="750" width="100%"></iframe>

[View Fullscreen](/posts/2020-05-12_youtube-history-visualisation/youtube-histviz-02.html)

## Conclusion

This project has allowed me to see how easy it is to take data and visualise it in a meaningful and insightful way. While I have also implemented this as a Dash app, I didn't feel it was done well enough to share, perhaps I'll do so on another project. You can see this implementation in the [source code](https://github.com/oliveryh/youtube-histviz) though if you like. 



