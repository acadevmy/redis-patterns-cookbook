# HyperLogLog

Unique items can be difficult to count. Usually this means storing every unique item then recalling this information somehow. With Redis, this can be accomplished with by using a set and a single command, however both the storage and time complexity of this with very large sets is prohibitive. HyperLogLog provides a probabilistic alternative.

HyperLogLog is similar to a Bloom filter internally as it runs items through a non-cryptographic hash and sets bits in a form of a bitfield. Unlike a Bloom filter, HyperLogLog keeps a counter of items that is incremented when new items are added that have not been previously added. This provides a very low error rate when estimating the unique items (cardinality) of a set. HyperLogLog is built right into Redis.

There are three HyperLogLog commands in Redis: PFADD, PFCOUNT and PFMERGE. Let's say you're building a web crawler and you want to estimate the number of unique URLs the page has crawled during a given day. For every page you would run a command like this:

> [PFADD crawled:20171124 "http://www.google.com/"](#run) => (integer) 1

> [PFADD crawled:20171124 "http://www.redislabs.com/"](#run) => (integer) 1

> [PFADD crawled:20171124 "http://www.redis.io/"](#run) => (integer) 1

> [PFADD crawled:20171125 "http://www.redisearch.io/"](#run) => (integer) 1

> [PFADD crawled:20171125 "http://www.redis.io/"](#run) => (integer) 1

Each key above is indexed by day. To see how many pages were crawled on 2017-11-24, we would run the following command:

>[PFCOUNT crawled:20171124](#run) => (integer) 3

To see how many pages were crawled on 2017-11-24 and 2017-11-25, we would use PFMERGE to produce a new key of the union of the two counts. Note that since http://redis.io/ was added to both keys it isn't counted twice:

> [PFMERGE crawled:20171124-25 crawled:20171124 crawled:20171125](#run) => OK

> [PFCOUNT crawled:20171124-25](#run) => (integer) 4

This is a multi-key operation, so take care in sharded environments as to make sure your keys end up on the same shard.

---
That wraps up the *Redis Pattern*. Please feel free to goof around with
this console as much as you'd like.

Check out the following links.

* [Redis Documentation](http://redis.io/documentation)
* [Command Reference](http://redis.io/commands)
* [Implement a Twitter Clone in Redis](http://redis.io/topics/twitter-clone)
* [Introduction to Redis Data Types](http://redis.io/topics/data-types-intro)
* [Pattern source by Redis Labs](https://redislabs.com/redis-best-practices/counting/hyperloglog/)
* [Acadevmy](https://acadevmy.it)