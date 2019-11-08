# Redis in Autosuggest

In this pattern, we are going to see how to use Redis to build a basic autocomplete
or autosuggest server.
To build such a system, we will use sorted sets and operations involving ranges and
intersections. To summarize, we will focus on the following topics in this pattern:

1. Autocompletion for words
2. Multiword autosuggestion using a sorted set
3. Faceted search using sets and operations such as union and intersection

---

## Simple word completition

In this section, we want to provide a simple word completion feature through Redis.
We will use a sorted set for this exercise. The reason behind using a sorted set is that italways guarantees O(log(N)) operations.
In order to complete a word (for example  jack, smith, scott, jacob, and jackeline), we need to use n-gram. Every word needs to be written as a contiguous sequence.
In order to signify the completed word, we can use a delimiter such as * or $.
To add the word into a sorted set, we will be using [ZADD](#help) in the following way:

> [zadd autocomplete 0 j](#run)

> [zadd autocomplete 0 ja](#run)

> [zadd autocomplete 0 jac](#run)

> [zadd autocomplete 0 jack$](#run)

Likewise, we need to add all the words we want to index for autocompletion.
Once we are done, our sorted set will look as follows:

> [zrange autocomplete 0 -1](#run)

* "j"
* "ja"
* "jac"
* "jack$"
* "jacke"

> [zrank autocomplete jac](#run) => 2

> [zrange autocomplete 3 50](#run)

* "jack$"
* "jacke"
* "jackel"
* "jackeli"

---
Now, in our program, we have to do the following tasks:

1. Iterate through the results set.
2. Check if the word starts with the query and only use the words with $ as
the last character.

Though it looks like a lot of operations are performed, both [ZRANGE](#help) and [ZRANK](#help) are O(log(N)) operations. Therefore, there should be virtually no problem in handling a huge list of words.
When it comes to memory usage, we will have n+1 elements for
every word, where n is the number of characters in the word. 
For M words, we will have M(avg(n) + 1) records where avg(n) is the average characters in a word. The more the collision of characters in our universe, the less the memory usage.

---

## Multiword phrase completion

In the previous section, we have seen how to use the autocomplete for a single word.
However, in most real world scenarios, we will have to deal with multiword phrases.
For this case, we will use a sorted set along with hashes. We will use a sorted set
to store the n-gram of the indexed data followed by getting the complete title from
hashes. Instead of storing the n-grams into the same sorted set, we will store them
in different sorted sets.
Example model names:

1. Samsung Galaxy Note 3 -> popularity 8
2. Samsung Galaxy Nexus -> popularity 7
3. Apple iPhone 5S -> popularity 9
4. Apple iPhone 5C -> popularity 6

For this set, we will create multiple sorted sets. Let's take Apple iPhone 5S:

> [ZADD a 9 apple_iphone_5s](#run)

> [ZADD ap 9 apple_iphone_5s](#run)

> [ZADD app 9 apple_iphone_5s](#run)

> [ZADD apple 9 apple_iphone_5s](#run)

> [ZADD i 9 apple_iphone_5s](#run)

> [ZADD ip 9 apple_iphone_5s](#run)

> [ZADD iph 9 apple_iphone_5s](#run)

> [ZADD ipho 9 apple_iphone_5s](#run)

> [ZADD iphon 9 apple_iphone_5s](#run)

> [ZADD iphone 9 apple_iphone_5s](#run)

> [ZADD 5 9 apple_iphone_5s](#run)

> [ZADD 5s 9 apple_iphone_5s](#run)

> [HSET titles apple_iphone_5s "Apple iPhone 5S"](#run)

---
In the preceding scenario, we have added every n-gram value as a sorted set and
created a hash that holds the original title. Likewise, we have to add all the titles
into our index.
Now that we have indexed the titles, we are ready to perform a search.
To achieve this:

> [zrevrange apple 0 4 withscores](#run)

1. "apple_iphone_5s"
2. 9.0
3. "apple_iphone_5c"
4. 6.0

As the elements inside the sorted set are ordered by the element score, we get the
matches ordered by the popularity which we inserted.
To get the original title type this command:

> [hmget titles apple_iphone_5s](#run) => "Apple iPhone 5S"

---
In the preceding scenario case, the query was a single word. Now imagine if the user
has multiple words such as Samsung nex, and we have to suggest the autocomplete
as Samsung Galaxy Nexus. To achieve this, we will use [ZINTERSTORE](#help) as follows:

> [zinterstore samsung_nex 2 samsung nex aggregate max](#run)

The previous command will compute the intersection of two sorted sets, samsung and
nex, and stores it in another sorted set, samsung_nex. To see the result, type the
following commands:

> [zrevrange samsung_nex 0 4 withscores](#run)

1. samsung_galaxy_nexus
2. 7

> [hmget titles samsung_galaxy_nexus](#run) => Samsung Galaxy Nexus

---

## The faceted search

In this part, we will see how Redis can be used for faceted search.
We want to create a facet search on mobile phones based on
manufacturer, operating system, and SIM type.
As our access pattern is to showcase the properties of the product, we have to create a reverse index on properties and store individual product details in a hash.

> [HMSET phone:1 title "Apple iPhone 5S"](#run)

> [SADD OS:iOS phone:1](#run)

> [SADD Manufacturer:Apple phone:1](#run)

> [SADD SIM:Single phone:1](#run)

---

> [HMSET phone:2 title "Samsung Galaxy S5"](#run)

> [SADD OS:Android phone:2](#run)

> [SADD Manufacturer:Samsung phone:2](#run)

> [SADD SIM:Single phone:2](#run)

``` 
```

> [HMSET phone:3 title "Samsung Galaxy Grand 2"](#run)

> [SADD OS:Android phone:3](#run)

> [SADD Manufacturer:Samsung phone:3](#run)

> [SADD SIM:Dual phone:3](#run)

``` 
```

> [HMSET phone:4 title "Samsung Wave 3"](#run)

> [SADD OS:Bada phone:4](#run)

> [SADD Manufacturer:Samsung phone:4](#run)

> [SADD SIM:Single phone:4](#run)

---
Now, let's say that the user wants to see all the phones manufactured by
Samsung. It will be a simple lookup in Redis to get all the phones by using
set Manufacturer:Samsung as follows:

> [SMEMBERS Manufacturer:Samsung](#run)

1. "phone:2"
2. "phone:3"
3. "phone:4"

We can perform [HMGET](#help) on each ID to get the required information about the phones.
Now that we have all mobiles manufactured by Samsung, let's look at how to further
categorize the list and only show Samsung devices that run on Google's Android
operating system. Now, we need to intersect between the two sorted sets as follows:

> [SINTER Manufacturer:Samsung OS:Android](#run)

1. "phone:2"
2. "phone:3"

To further refine the search, we can add more filters. For instance, to retrieve a
list of Android dual-SIM mobiles manufactured by Samsung, we can use the
following command:

> [SINTER Manufacturer:Samsung OS:Android SIM:Dual](#run)

1. "phone:3"

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
