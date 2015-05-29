+++
date = "2015-05-15T20:00:00+02:00"
title = "How to Follow pmarca on Twitter"

+++

It started with a question: how do you follow @pmarca on Twitter?
@pmarca is Marc Andreessen, investor and cofounder of
Netscape. Averaging 100 tweets a day, he is hard to keep up with.

<!--more-->

When I asked this question to a group of people I got essentially two
types of answers: (a) don't try to keep up (b) make judicious use of
lists. While useful, I don't think these suggestion hit the core of
the problem:

**Twitter Attention Inequality**: A person tweeting 100 times a day
gets 100 times more exposure than someone tweeting once a day, even
though you care equally about what they have to say.

When I first tried to solve this problem a few months ago, I wrote a
bunch of code that tried to solve the problem in a general way,
essentially making a Twitter app that displays anyone's timeline
filtered through a some kind of relevance score. The details don't
matter - the gist is that I wasted a bunch of time dealing with
Twitter API restrictions (OAuth 1, rate limits, broken libraries,
unable to use certain APIs, more rate limits, etc) rather than testing
my assumptions. I attacked the problem twice, and ended up abandoning
both approaches.

Yesterday I started thinking about the problem again, and realized
there was a bunch of assumptions I could test without writing a single
line of code.

## Hypotheses

First some observations: (a) I ignore most tweets and (b) unlike
email, there are no can't-miss tweets in a timeline.

Here are some assumptions I wanted to test:

1. Some people tweet 10x more than average.
2. Some people tweet more relevant things than others.
3. Removing heavy tweeters will increase timeline relevance.

## Experiment

I looked at the last 200 tweets that came up in my timeline, which
roughly corresponds to a 20 hour window. For each tweet I put them in
one of three categories:

1. Ignore
2. Engage (click, reply, think briefly about)
3. Recommend (favourite, retweet, or write down with other means)

This was a snap decision, and my measured engagement rate was probably
higher than my actual one, since I don't normally look carefully at
every tweet.

## Data

Out of the 200 tweets, I engaged with 20% (35) of them. Less than 5%
(1.5%) were marked as recommended. The number of 3s were so negliable,
just two in that time period, that I decided to bake it into the
"Engage" category. Thus, 80% of tweets were simply noise.

Looking at @pmarca's tweets during the same 20 hour period, I found
that he tweeted 30-35 times. I follow around 150 people, which makes
@pmarca 30x more prolific of a tweeter than the average person I
follow.

However, I noticed something surprising. The engagement rate for
@pmarca's tweets was 20%, just as it was with my normal timeline! This
invalidated my third assumption.

Is it possible to increase the relevance of @pmarca's tweets in a
straighforward way? I tried filtering out his retweets, as well as
only looking at the ones with a favourite count of 100 or
more. Neither resulted in a higher engagement ratio (sample size 10),
and both were implicit assumptions that I had in my initial
prototypes.

I also looked at three people I knew were high-quality tweeters. My
engagement for their tweets was over 50% (sample size 10), which
suggests there is such a thing as more relevant tweeters [1].

## A smaller version

If my engagement ratio with @pmarca's tweets is the same as my normal
timeline, why don't I follow him? One way to think about it is in
terms of attentions. If I have 100 attentions, where one attention is
one tweet, then I don't want to spend a large portion of them on one
person. Outside of its conversation-like nature, one of the main
benefits of Twitter is that it allows for a plurality of views.

After having freed my mind from thinking I have to predict the quality
of a tweet or tweeter, I arrived at an obvious and simple solution:
just remove 90% randomly of his tweets, to reduce the volume! So I
created a small Twitter bot that does exactly this, in just 50 lines
of code of Clojure.

```
(ns pmarca-chen.core
  (:require [clojure.set :as set]
            [twitter.oauth :as oauth]
            [twitter.api.restful :as api]))

(def old-tweets (atom #{}))

(def my-creds (oauth/make-oauth-creds
               (System/getenv "PMARCACHEN_CONSUMER_KEY")
               (System/getenv "PMARCACHEN_CONSUMER_SEC")
               (System/getenv "PMARCACHEN_ACCESS_TOKEN")
               (System/getenv "PMARCACHEN_ACCESS_TOKEN_SEC")))

(defn timeline []
  (api/statuses-home-timeline :oauth-creds my-creds))

(defn fetch-tweets []
  (set (map :id (:body (timeline)))))

(defn retweet! [tweet]
  (do (api/statuses-retweet-id :oauth-creds my-creds
                               :params {:id tweet})
      (prn "Tweeted " tweet)))

(defn maybe-retweet!
  "Retweet a tweet 10% of the time."
  [tweet]
  (if (= (rand-int 10) 9)
    (retweet! tweet)
    (prn (str "Discarded tweet " tweet))))

(defn fetch-and-retweet!
  "Fetches tweets, retweets some, and calculates new tweets, and maybe
  retweets them. Adds new tweets to old tweets set."
  []
  (let [all-tweets (fetch-tweets)
        new-tweets (set/difference all-tweets @old-tweets)]
    (do (println (str "Found " (count new-tweets) " new tweets."))
        (dorun (map maybe-retweet! new-tweets))
        (swap! old-tweets set/union new-tweets))))

(defn periodically! [f ms]
  (future (while true (do (Thread/sleep ms) (f)))))

(comment
  (def pmarca-chen (periodically! fetch-and-retweet! (* 1000 60 5)))
  (future-cancel pmarca-chen)
  )
```

This checks for the latest tweets every five minutes, diffs the
fetched tweets with tweets that have been seen already to figure out
the new tweets, and retweets a new tweet with a probablity of 10%.

You can find the source code
[here](https://github.com/oskarth/pmarca-chen).

## Conclusion

You can follow @pmarca_chen on Twitter
[here](https://twitter.com/pmarca_chen). I hope people find it useful,
and if you create a bot on your own, please let me know and I'll add
it here.

I would like to end this article with a quote by Herbert Simon:

*A wealth of information creates a poverty of attention*.

Ever since I first read that quote a few years, it stuck with me. In a
world where information is abundant, we should fight for our
attention, and come up with new ways of preserving and enriching it.

## Notes

[1] This is outside the scope of this article, but one interesting
idea is to use your own favourite count per user as a proxy for how
relevant a person is to you, and then change your timeline
accordingly.
