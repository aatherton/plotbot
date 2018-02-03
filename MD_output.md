

```python
# import dependancies
import tweepy, pandas, requests, os, json
from matplotlib import pyplot as plt
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
```


```python
# declare globals
ball = os.path.dirname(os.path.dirname(os.path.realpath('__file__')))
ball = os.path.join(ball, "keyring.json")
keyring = json.load(open(ball))["twitter"]
ball = tweepy.OAuthHandler(keyring["consumer"]["key"], keyring["consumer"]["secret"])
ball.set_access_token(keyring["token"], keyring["secret"])
# string containing the bot's account
MY_ACCOUNT = "@TAT_bot"
# container for the tweepy api
API = tweepy.API(ball, parser=tweepy.parsers.JSONParser())
# container for vader analyzer:
ANALYZER = SentimentIntensityAnalyzer()
# number of tweets to pull
PULL_NUM = 500
# place to save pics
LOCATION = "placeholder.png"
# time to wait, in seconds
WAIT = 5*60
# stores bad requests
nogo = []
```


```python
# declare functions
```


```python
def scan(account, api):
    """scans the account for new mentions. returns a list: [success boolean, most recent tweet]"""
    result = [False]
    ball = api.search(account, count=1, result_type = "recent")
    if ball["search_metadata"]["count"] >= 1:
        result[0] = True
        result.append(ball["statuses"][0])
    return(result)
```


```python
def parse(tweet, account):
    """takes a tweet. looks for an accounts' mention and returns the first one that's not the input account"""
    result = []
    for each in tweet["entities"]["user_mentions"]:
        result.append(f"@{each['screen_name']}")
    result.remove(account)
    return(result[0])
```


```python
def request(account, n, api):
    """requests the n most recent tweets for the given account. returns list of tweets"""
    result = []
    ball = 0
    while n >= 20:
        result.extend(api.user_timeline(account, page= ball))
        n -= 20
        ball += 1
    if n > 0:
        result.extend(api.user_timeline(account, page= ball+1, count= n))
    return(result)
```


```python
def crunch(tweets):
    """processes data based on input. returns a Pandas DataFrame"""
    result = []
    for each in range(len(tweets)):
        result.append({})
        result[-1]["Sentiment"] = ANALYZER.polarity_scores(tweets[each]["text"])["compound"]
        result[-1]["Tweets Ago"] = 0-each
    return(pandas.DataFrame(result))
```


```python
def picture_this(data, title, location):
    """takes a Pandas DataFrame. makes a plot out of it. saves it to location"""
    plt.scatter(data["Tweets Ago"], data["Sentiment"], marker = "o")
    plt.xlabel("Tweets Ago")
    plt.ylabel("Sentiment")
    plt.grid(alpha = .25)
    plt.title("Sentiment Analysis of Tweets for {}".format(title))
    plt.savefig(location)
    return(True)
```


```python
def chirp(location, account, api):
    """posts a tweet mentioning account with a picture at location attached"""
    api.update_with_media(location, "here's your stuff, @{account}")
```


```python
def sanity_check(tweet, nogo):
    """returns boolean"""
    result = tweet[0]
    try:
        if parse(tweet[1]) in nogo:
            result = False
        request(parse(tweet[1]))
    except:
        result = False
    return(result)
```


```python
def main():
    ball = scan(MY_ACCOUNT, API)
    if sanity_check(ball, nogo):
        nogo.append(parse(ball[1]))
        picture_this(crunch(request(parse(ball[1]), PULL_NUM, API)), LOCATION)
        chirp(LOCATION, ball[1]["user"]["screen_name"], API)
```


```python
# deploy functions
while True:
    time.sleep(WAIT)
    main()
```


```python

```




    True


