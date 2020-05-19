# Sorted Sets as Indexes

Sorted Sets (ZSETs) are a native Redis data type that can be thought of a set of unique members (repeats are not stored) with each member being attached to a number (termed score) that acts as a natural sorting mechanism. 

While members cannot be repeated, any number of members can share the same score. With relatively low time complexity to add, remove and retrieve ranges (by rank or score), this lends itself naturally to being an index. As an example, let’s say you have the world’s countries ranked by population:

>[ZADD countries-by-pop china 1409517397](#run) => ... every country population ...

>[ZADD countries-by-pop vatican-city 792](#run)

To retrieve the top five countries (e.g. rank), it would be simply:

>[ZRANGE countries-by-pop 0 4](#run)

While getting the countries with a population between 1,000,000 and 100,000 (e.g. by score) would be:

>[ZRANGEBYSCORE countries-by-pop 100000 1000000](#run)

Multiple indexes can be created to represent different ways of sorting the data. In our example we could use the same members but instead of population, we could use population density, geographic size, number of internet users, etc. This would create high-performance indexes for multiple facets of the data. Additionally, by sharing the member name with a detailed item either stored in Redis itself (in a Hash, as example) or in another datastore, a secondary process could get additional information about each item as needed.

---
That wraps up the *Redis Pattern*. Please feel free to goof around with
this console as much as you'd like.

Check out the following links.

* [Redis Documentation](http://redis.io/documentation)
* [Command Reference](http://redis.io/commands)
* [Implement a Twitter Clone in Redis](http://redis.io/topics/twitter-clone)
* [Introduction to Redis Data Types](http://redis.io/topics/data-types-intro)
* [Pattern source by Redis Labs](https://redislabs.com/redis-best-practices/indexing-patterns/sorted-sets-indexes/)
* [Acadevmy](https://acadevmy.it)