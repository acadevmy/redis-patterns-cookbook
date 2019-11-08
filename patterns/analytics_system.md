# Redis in an analytics system

In this pattern, we will explore how to deploy Redis in the domain of operational
intelligence, in which we convert huge chunks of data into actionable business
intelligence.
To perform any operations in analytics systems, one should perform the
following steps:

1. Capture the data.
2. Process the information.
3. Generate actionable reports or output.

Let's assume that we are developing an online deals site which provides the best deals available online. We have hundreds of live deals on the site. The objective is to calculate the popularity of each deal. We want to calculate real-time popularity using Redis. Assume that the popularity of the deal is calculated using the number of times the deal was seen by the user and the number of times the user clicked on the link
to go to the relevant store to claim the deal. Each of these actions has a weight of 1
and 5 respectively.

---
We will use sorted sets on each of the data points. In this case, we will be looking
at two data points. On every visit, we will increment the score of the deal in
`deals:visits` by one and on every link click, we will increment the score of the
deal in `deals:click` by one.
For example, if a user visits the page of deal with ID 453, increment the visit count
of the deal ID by executing the following command:

>[ZINCRBY deals:visits 1 "dealid:453"](#run)

Similarly, when the user visits other deals, we will increment the score for the
respective deal ID as follows:

>[ZINCRBY deals:visits 1 "dealid:282"](#run)

>[ZINCRBY deals:visits 1 "dealid:453"](#run)

>[ZINCRBY deals:visits 1 "dealid:1169"](#run)

>[ZINCRBY deals:visits 1 "dealid:453"](#run)

>[ZINCRBY deals:visits 1 "dealid:282"](#run)

We can do the same for link clicks:

>[ZINCRBY deals:clicks 1 "dealid:453"](#run)

>[ZINCRBY deals:clicks 1 "dealid:282"](#run)

---
To analyze the top deals based on page views, we can use [ZREVRANGE](#help), which gives
us a view of all the required data:

>[ZREVRANGE deals:visits 0 -1 WITHSCORES](#run)

* "dealid:453"
* 3.0
* "dealid:282"
* 2.0
* "dealid:1169"
* 1.0

In case we want to look into pages that have at least two views, we can use the
following command:

>[ZREVRANGEBYSCORE deals:visits +inf 2](#run)

* "dealid:453"
* "dealid:282"

We can get the click-through data in a similar manner:

>[ZREVRANGE deals:clicks 0 -1 WITHSCORES](#run)

* "dealid:453"
* 1.0
* "dealid:282"
* 1.0

---
Let's suppose that we want to compute the popularity of each of the deals based
on visits and clicks. Here, [ZUNIONSTORE](#help) comes in handy.

>[ZUNIONSTORE deals:pops 2 deals:visits deals:clicks WEIGHTS 1 5](#run)
(integer) 3

>[ZREVRANGE deals:pops 0 -1 WITHSCORES](#run)

* "dealid:453"
* "8"
* "dealid:282"
* "7"
* "dealid:1169"
* "1"

If you see the scores in our deals:pops key, it is the sum of scores of deals:
visits and deals:clicks multiplied by the weights we gave to [ZUNIONSTORE](#help).
This information can be used to provide the real-time popularity of deals based
on the user data. Overall, all of this can be achieved in under a few hundred lines
of code. Adding another data point for our popularity logic is as simple as
capturing the data in another sorted set and adding the sorted set into the
[ZUNIONSTORE](#help) command.

---
That wraps up the *Redis Pattern*. Please feel free to goof around with
this console as much as you'd like.

Check out the following links.

* [Redis Documentation](http://redis.io/documentation)
* [Command Reference](http://redis.io/commands)
* [Implement a Twitter Clone in Redis](http://redis.io/topics/twitter-clone)
* [Introduction to Redis Data Types](http://redis.io/topics/data-types-intro)
* [Pattern source](https://books.google.it/books/about/Redis_Applied_Design_Patterns.html?id=0mKZBAAAQBAJ&printsec=frontcover&source=kp_read_button)
* [Acadevmy](https://acadevmy.it)