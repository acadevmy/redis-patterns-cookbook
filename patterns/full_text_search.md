# Full Text Search

Prior to the advent of modules, full-text search was implemented using native Redis commands. The RediSearch module provides much higher performance than this patterns. However, in some environments, RediSearch is not available. Additionally, this pattern is very interesting and can be generalized to other workloads for which RediSearch may not be ideal.

Let's say you have a number of text documents you want to search, this may not be an obvious use-case for Redis as it access via keys rather than tables. On the contrary, Redis can be used to underpin a very novel full text search engine.

First, let's take some examples:

* "Redis is very fast"
* "Cheetahs are fast"
* "Cheetahs have spots"

Let's break down these items into sets of words just limited by space for simplicity:

> [SADD ex1 redis is very fast](#run)

> [SADD ex2 cheetahs are very fast](#run)

> [SADD ex3 cheetahs have spots](#run)

Notice that we're giving each line it's own set (ex1…) and then we're adding multiple members to that set based on each word (even though it might looks we're just adding the entire line, SADD is variadic, so accepts multiple members. We've also turned all the words lowercase.

Next we need to invert this index and show which word is located in which document. To do this, we'll make a set for each word and then put the document set names as members.

> [SADD redis ex1](#run)

> [SADD is ex1](#run)

> [SADD very ex1 ex2](#run)

> [SADD fast ex1 ex2](#run)

> [SADD cheetahs ex2 ex3](#run)

> [SADD have ex3](#run)

> [SADD spots ex3](#run)

For clarity, we've split this up into individual commands, but all the commands would normally be atomicly executed with a MULTI/EXEC block.

---

To query our tiny full-text search index, we will use the SINTER command (set intersect).  To find the documents with “very” and “fast”

> [SINTER very fast](#run) => 
* "ex2"
* "ex1"

In a situation where we don't have any documents that match the query, we'll get an empty result:

> [SINTER cheetahs redis](#run)
(empty list or set)

If you wanted an logical or search, you can substitute SUNION (set union) for SINTER.

> [SUNION cheetahs redis](#run)

* "ex2"
* "ex3"
* "ex1" 

Deleting an item from the index is a little more involved. First, we'll get the document index members from the document set (SMEMBERS) then remove the document IDs from the word indexes.

> [SMEMBERS ex3](#run) =>
    1) "have"
    2) "cheetahs"
    3) "spots"

> [SREM have ex3](#run)

> [SREM cheetahs ex3](#run)

> [SREM spots ex3](#run)

This cannot be completed in a single operation in Redis, so you'll need to get the results of SMEMBERS then issue the SREM commands afterwards.

Of course, this is a very simple full text search, you create a more reflective index by using sorted set commands instead of set commands. This way, as example, if a document contains a word more than once, you can have it “rank” higher than a document that has only one occurrence. The patterns above stay more-or-less the same except use sorted set commands.

---
That wraps up the *Redis Pattern*. Please feel free to goof around with
this console as much as you'd like.

Check out the following links.

* [Redis Documentation](http://redis.io/documentation)
* [Command Reference](http://redis.io/commands)
* [Implement a Twitter Clone in Redis](http://redis.io/topics/twitter-clone)
* [Introduction to Redis Data Types](http://redis.io/topics/data-types-intro)
* [Pattern source by Redis Labs](https://redislabs.com/redis-best-practices/indexing-patterns/full-text-search/)
* [Acadevmy](https://acadevmy.it)