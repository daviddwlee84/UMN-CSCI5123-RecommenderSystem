# Notes of Additional Reading

## UX Research: 5 Requirements for the ‘Ratings Distribution Summary’ on the Product Page (65% Get it Wrong)

* Qualitative Information: users describe their experiences with a product, and those descriptions are read by other users interested in buying the same product
* Quantitative Information: ratings or distribution; provides data on the total number of reviews and the ratings breakdown as an illustration (usually a bar chart), allows users to see at a glance how a product has been rated

Conclusion

* be graphical illustrations (normally horizontal bar charts),
* act as “star” filters,
* be exposed or expanded by default,
* use “or” radio button logic, rather than “and” checkbox logic, and
* be hidden when there are only 5 or fewer reviews.

## How Not To Sort By Average Rating - Wilson score confidence interval

PROBLEM: You are a web programmer. You have users. Your users rate stuff on your site. You want to put the highest-rated stuff at the top and lowest-rated at the bottom. You need some sort of “score” to sort by.

* WRONG SOLUTION #1: Score = (Positive ratings) − (Negative ratings)
* WRONG SOLUTION #2: Score = Average rating = (Positive ratings) / (Total ratings)
* CORRECT SOLUTION: Score = Lower bound of Wilson score confidence interval for a Bernoulli parameter

OTHER APPLICATIONS

The Wilson score confidence interval isn’t just for sorting, of course. It is useful whenever you want to know with confidence what percentage of people took some sort of action. For example, it could be used to:

* Detect spam/abuse: What percentage of people who see this item will mark it as spam?
* Create a “best of” list: What percentage of people who see this item will mark it as “best of”?
* Create a “Most emailed” list: What percentage of people who see this page will click “Email”?

## How Hacker News ranking algorithm works

```txt
Score = (P-1) / (T+2)^G

where,
P = points of an item (and -1 is to negate submitters vote)
T = time since submission (in hours)
G = Gravity, defaults to 1.8 in news.arc
```

* the score decreases as T increases, meaning that older items will get lower and lower scores
* the score decreases much faster for older items if gravity is increased

```py
def calculate_score(votes, item_hour_age, gravity=1.8):
    return (votes - 1) / pow((item_hour_age+2), gravity)
```

## How Reddit ranking algorithms work

### Story Ranking

* Submission time has a big impact on the ranking and the algorithm will rank newer stories higher than older
* The score won’t decrease as time goes by, but newer stories will get a higher score than older. This is a different approach than the Hacker News’s algorithm which decreases the score as time goes by

```py
from datetime import datetime, timedelta
from math import log

epoch = datetime(1970, 1, 1)

def epoch_seconds(date):
    td = date - epoch
    return td.days * 86400 + td.seconds + (float(td.microseconds) / 1000000)

def score(ups, downs):
    return ups - downs

def hot(ups, downs, date):
    s = score(ups, downs)
    order = log(max(abs(s), 1), 10)
    sign = 1 if s > 0 else -1 if s < 0 else 0
    seconds = epoch_seconds(date) - 1134028003
    return round(sign * order + seconds / 45000, 7)
```

* Submission time is a very important parameter, generally newer stories will rank higher than older
* The first 10 upvotes count as high as the next 100. E.g. a story that has 10 upvotes and a story that has 50 upvotes will have a similar ranking
* Controversial stories that get similar amounts of upvotes and downvotes will get a low ranking compared to stories that mainly get upvotes

> * [Upvoted – The official Reddit blog](https://redditblog.com/)

* Using the hot algorithm for comments isn’t that smart since it seems to be heavily biased toward comments posted early
* In a comment system you want to rank the best comments highest regardless of their submission time
* A solution for this has been found in 1927 by Edwin B. Wilson and it’s called “Wilson score interval”, Wilson’s score interval can be made into “the confidence sort”
* The confidence sort treats the vote count as a statistical sampling of a hypothetical full vote by everyone — like in an opinion poll
* How Not To Sort By Average Rating outlines the confidence ranking in higher detail, definitely recommended reading!

### Comment Ranking

```py
from math import sqrt

def _confidence(ups, downs):
    n = ups + downs

    if n == 0:
        return 0

    z = 1.281551565545
    p = float(ups) / n

    left = p + 1/(2*n)*z*z
    right = z*sqrt(p*(1-p)/n + z*z/(4*n*n))
    under = 1+1/n*z*z

    return (left - right) / under

def confidence(ups, downs):
    if ups + downs == 0:
        return 0
    else:
        return _confidence(ups, downs)
```

* The confidence sort treats the vote count as a statistical sampling of a hypothetical full vote by everyone
* The confidence sort gives a comment a provisional ranking that it is 85% sure it will get to
* The more votes, the closer the 85% confidence score gets to the actual score
* Wilson’s interval has good properties for a small number of trials and/or an extreme probability

> Comments are ranked by confidence and by data sampling — — i.e. the more votes a comment gets the more accurate its score will become.
