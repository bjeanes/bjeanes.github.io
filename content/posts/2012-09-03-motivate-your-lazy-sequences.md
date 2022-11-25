+++
aliases = ["2012/motivate-your-lazy-sequences", "2012/09/motivate-your-lazy-sequences"]
date = 2012-09-03T18:00:53Z
slug = "motivate-your-lazy-sequences"
title = "Motivate your lazy sequences"
updated = 2012-09-04T00:17:41Z
+++

I *love* Clojure’s laziness.

Recently, I’ve been using `lazy-seq`  to consume remote collections via APIs, fetching pages of data transparently and only as needed. [Gary Fredericks](http://gfredericks.com/), [Mike Busch](http://mikelikesbikes.com/), and I applied this to a project of ours that had to crunch tens of thousands of records from Salesforce. I’m also doing something similar with a personal project that has to fetch a lot of Pivotal Tracker stories to `reduce` them.

In cases like these, consuming (potentially unbounded) resources in a lazy manner allows one to start processing data earlier and to make as few requests as possible to get only the data you need.

## *Mostly* Lazy

I want to talk about a neat little thing I did in my project to get a nice little performance boost on top of this laziness, without having to think about any low-level concurrency concerns.

### Laziness

Here’s a piece of code that provides an “infinite” lazy sequence. In this case, it is of tweets:

```clojure
(require '[clj-http.client :as http])

(defn tweets-for
  ([user] (tweets-for user nil))
  ([user last-tweet-id]
     (lazy-seq
      (let [url "http://api.twitter.com/1/statuses/user_timeline.json"
            params {:limit 10 :screen_name user}
            params (if last-tweet-id (assoc params :max_id last-tweet-id) params)
            response (http/get url {:query-params params :as :json})
            tweets (:body response)]

        (when (not-empty tweets)
          (concat tweets 
                  (tweets-for user (:id (last tweets)))))))))
```

So that’s cool. Now, note the following performance characteristics when contemplating the next section:

```clojure
(def my-tweets (tweets-for "bjeanes"))

;; The following returns after a delay while we fetch the first page:
(first my-tweets) ;=> {:text "Tweet 0" ...}

;; This returns instantly because our `tweets-for` function fetches 10 tweets per page:
(nth my-tweets 9) ;=> {:text "Tweet 9" ...}

;; This returns after a delay because this tweet is on the next (still lazily unfetched) page:
(nth my-tweets 10) ;=> {:text "Tweet 10" ...}
```

### Motivation

So laziness is pretty cool. But, sometimes, things can improve if you are ever so slightly less lazy. What if we could remove that little pause between the 9th and the 10th items in the list where we are just waiting around for the network request to Twitter to complete? We could be using our time to do more CPU-melting tweet crunching! Well, it turns out we can easily do it.

Assume for a moment that we have some calculation (`process`) that takes a considerable amount of CPU time to process:

```clojure
(defn process
  "Do some really hard work with our tweets"
  [tweets]
  (map #(Thread/sleep 100) tweets))
```

If we know we will be consuming a substantial amount of the lazy sequence, we could encourage the sequence to go ahead and start realizing the next chunk of our sequence. 

This would mean that instead of processing 10 tweets, waiting, processing 10 tweets, waiting, etc.:

![lazy](/lazy.png)

... we would be able to process tweets continuously back-to-back:

![motivated](/motivated.png)

Wouldn't also be great if we didn't have to think about the parallelism at all? To this end, I present `motivate`:

```clojure
(defn motivate
  "Motivate a lazy sequence to seek slightly ahead of the sequence consumer's position."
  ([coll] (motivate coll 1))
  ([coll motivation]
     (lazy-seq
      (future (nth coll motivation))
      (cons (first coll) (motivate (rest coll) motivation)))))
```

Let’s compare:

```clojure
(time (process (take 100 (tweets-for "riblah"))))
;=> “Elapsed time: 11545.011 msecs"
(time (process (take 100 (motivate (tweets-for "riblah") 5))))
;=> "Elapsed time: 10394.769 msecs"
```

The speed difference is noticeable even when processing only a 100 tweets. If we were doing more than 100 milliseconds/tweet of processing, fetching a lot more data, or dealing with a slow upstream dependency, the speed improvements would be even clearer.

The last (optional) parameter to `motivate` is the “motivation factor”. If your CPU-bound work is long-running, this number can be smaller without a noticeable difference. The ideal number depends on how long each IO operation takes and much processing you do with each chunk. 

Essentially, the motivation you give to the lazy sequence is a trade-off between waiting for IO and wasting IO; that is, the lower the number, the more likely you are to wait on IO but the higher the number, the more IO you’ll perform unnecessarily (at least, if you aren’t guaranteed to consume the whole sequence.

Hopefully this is handy to someone else out there. I wouldn’t at all be surprised if something like this already existed (**UPDATE** Yup: [`seque`](http://clojuredocs.org/clojure_core/clojure.core/seque)) or if this completely obvious to seasoned Clojurian, but it was a pleasant moment discovering this possibility on my own.