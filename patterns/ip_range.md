# IP Range Indexing

Finding the actual location of a connected service can be very useful. IP to location tables tend to be quite large and difficult to manage efficiently. We can use the sorted set data type of implement a quick and efficient IP to location service.

IPv4 addresses are most often referenced by humans in dot-decimal notation, 74.125.43.99, as a random example. However, networking services see this same address as a 32-bit number, with each byte representing one of the four numbers in dot-decimal notation. Our example above would be 0x4A7D2B63 in hex or 1249717091 in decimal.

IP address to location datasets are widely available and usually take the form of a simple table with three columns [start, end, location]. The start and end are represented as the decimal representation of the IPv4 addresses. In Redis we can adapt a sorted set to this format because the IP ranges lack “holes” so the (safe) assumption is that the “end” of one address range is the start of another.

For this simplified example, let’s add a few ranges to a sorted set:

> [ZADD ip-loc 1249716479 us:1](#run)

> [ZADD ip-loc 1249716735 taiwan:1](#run)

> [ZADD ip-loc 1249717759 us:2](#run)

> [ZADD ip-loc 1249718015 finland:1](#run)

The first argument is our sorted set key, the second is the decimal representation of the IP address range end and the final argument is the member. Notice the member of the sorted sets have a number concatenated after the final colon. This is just to simplify the example. A real IP location table would likely have unique identifiers for each range (and for more information than just country).

To query the table versus a given IP address, we can use the Redis command ZRANGEBYSCORE with a few extra arguments. First, though, we’ll take a human-readable IP address and convert it into decimal. This can be done in the language of your choice. To start off, we’ll use the address from the initial example, 74.125.43.99 or 1249717091 in decimal. If we take this number as the minimum score and have no maximum then limit the result to just the first entry, we’ll find the location.

>[ZRANGEBYSCORE ip-loc 1249717091 +inf LIMIT 0 1](#run) => "us:2"

The first argument is our sorted set key, the second is the decimal representation of the IP address in decimal form the third (+inf) tells redis to not have a upper range ceiling and finally the last three arguments just indicate we want an offset 0 and only return one result.
---
That wraps up the *Redis Pattern*. Please feel free to goof around with
this console as much as you'd like.

Check out the following links.

* [Redis Documentation](http://redis.io/documentation)
* [Command Reference](http://redis.io/commands)
* [Implement a Twitter Clone in Redis](http://redis.io/topics/twitter-clone)
* [Introduction to Redis Data Types](http://redis.io/topics/data-types-intro)
* [Pattern source by Redis Labs](https://redislabs.com/redis-best-practices/indexing-patterns/ip-range-indexing/)
* [Acadevmy](https://acadevmy.it)