# Lexicographic Sorted Set Time Series

Another way of working with time series is to use the lexicographic properties of sorted sets to store the timestamp and values. If you haven't read the section about lexicographical encoding, now would be a good time.

Effectively, in this method we're storing everything at the same score (usually 0) and encoding first the timestamp then the value as the member. Take this example:

> [ZADD lex-temperature 0 1511533205001:21](#run) => (integer) 1

> [ZADD lex-temperature 0 1511533206001:22](#run) => (integer) 1

> [ZADD lex-temperature 0 1511533207001:21](#run) => (integer) 1

> [ZRANGE lex-temperature 0 -1](#run) => "1511533205001:21", "1511533206001:22", "1511533207001:21"


In the three ZADD commands, the timestamp is separated from the value by a colon and we can see that each return a 1 indicating that all three were added. In the final command ZRANGE, we see the order is preserved. Why? In sorted sets when the score is equal, the the results with the same score are ordered by binary sorting. Since timestamps in our present time period all have the same number of digits, everything will be sorted nicely (if your timestamps go before the year 2002 or after 2285, you'll want to add padding digits).

While this doesn't rely on added uniqueness to prevent duplicates, it has a different shortcoming: multiple items during the same millisecond with the same value won't be recorded. This may or may not be an issue for your use-case, in practice millisecond resolution provides sufficient for most situations.

To get a range of values out of this type of time series model, you'll use the [ZRANGEBYLEX](#help) command. Let's say we want to get the temperatures after timestamp 1511533200001:

> [ZRANGEBYLEX lex-temperature (1511533200001 +](#run) => "1511533205001:21" ,"1511533206001:22" ,"1511533207001:21"

The second argument is prefixed with an open parenthesis to denote that it's exclusive of the actual value (inclusive is denoted with an open square bracket). In practice, the above format makes inclusive and exclusive irrelevant since the timestamp will always be followed by additional data. The third argument (+) indicates that we want no upper bound.

Let's look at a counter-intuitive gotcha. Take this code where we try to get data between 1511533200001 and 1511533207001, inclusive:

> [ZRANGEBYLEX lex-temperature [1511533200001 [1511533207001](#run) => "1511533205001:21", "1511533206001:22"

Why was the data at timestamp 1511533207001 excluded despite adding in the inclusive prefix? It's very tempting to think that Redis somehow understands that this is a timestamp – but the truth is that Redis is naive to this – Redis just sees binary sorting. In binary sorting “1511533207001:21” is larger than “1511533207001” inclusively or exclusively. The correct way to be inclusive on the upper bounds is to go one millisecond up from the desired upper boundary and use the exclusive notation:

> [ZRANGEBYLEX lex-temperature [1511533200001 (1511533207002](#run) => "1511533205001:21", "1511533206001:22", "1511533207001:21"

---

Lexicographic sorted sets time series has a similar set of useful commands as plain sorted set time series data does:

[ZRANGEBYLEX](#help) / [ZREVRANGEBYLEX](#help) Get a range of values by ascending or descending sort (respectively)
[ZREMRANGEBYLEX](#help) Remove a particular range of values by sort
[ZLEXCOUNT](#help) Get the count of items between a range of values by sort.
[ZINTERSTORE](#help) and [ZUNIONSTORE](#help) can be used on lexicographic sorted sets, however there is a risk of losing data points as duplicate timestamp and value combinations will not be duplicated in the data.

You may be wondering why you would choose sorted sets with timestamps scores vs lexicographic sorted set encoding. Normally, you'll be better served with lexicographic sorted sets for time series unless your values will always be unique then timestamp scores will be more efficient.

---
That wraps up the *Redis Pattern*. Please feel free to goof around with
this console as much as you'd like.

Check out the following links.

* [Redis Documentation](http://redis.io/documentation)
* [Command Reference](http://redis.io/commands)
* [Implement a Twitter Clone in Redis](http://redis.io/topics/twitter-clone)
* [Introduction to Redis Data Types](http://redis.io/topics/data-types-intro)
* [Pattern source by Redis Labs](https://redislabs.com/redis-best-practices/time-series/lexicographic-sorted-set-time-series/)
* [Acadevmy](https://acadevmy.it)