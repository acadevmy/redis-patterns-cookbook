# Object->Hash Storage

The native Redis datatype hash(map) may, at first glance, seem very similar to a JSON object or other record data type. It is quite a bit simpler allowing only for a each field to be either a string or number and not allowing for sub-fields. However, by pre-computing the ‘path' of each field, you can flatten an object and store it in a Redis Hashmap.

Take the this example:

```
{
    "colour":"blue",
    "make":"saab",
    "model":{
       "trim" : "aero",
       "name" : 93
    },
    "features":[ "powerlocks", "moonroof" ]
}
```

Using JSONPath we can represent each item in one level with a hashmap.
> [HSET car3 colour blue](#run)

> [HSET car3 make saab](#run)

> [HSET car3 model.trim aero](#run)

> [HSET car3 model.name 93](#run)

> [HSET car3 features[0] powerlocks](#run)

> [HSET car3 features[1] moonroof](#run)

(Shown as individual commands for clarity, in a script you would want to variadicly use HSET)

This provides a way to can get the entire object:

> [HGETALL car3](#run)

* "colour"
* "blue"
* "make"
* "saab"
* "model.trim"
* "aero"
* "model.name"
* "93"
* "features[0]"
* "powerlocks"
* "features[1]"
* "moonroof"

Or individual parts of the object:

> [HGET car3 model.trim](#run) => "aero"

---

To query our tiny full-text search index, we will use the SINTER command (set intersect).  To find the documents with “very” andWhile this can provide a quick, useful method to access store objects in Redis, it does have downsides:

* Different languages and libraries may have slightly different implementations of JSONPath causing incompatibilities. To this point, you should serialize and deserialize with the same library.
* Arrays support can be spotty
    * Sparse arrays can be problematic
    * It's not possible to do many operations such as inserting an item in the middle of the array
* There is some unnecessary overhead in the JSONPath keys

This pattern has significant overlap with ReJSON. If ReJSON is available, it is very often a better choice. Storing objects this way does have one advantage over ReJSON of being integrated with another Redis command: SORT. However, the SORT command is both a complex topic (beyond the scope of this pattern) and computationally complex which can be wielded in ways that don't scale well if not very carefully managed.

---
That wraps up the *Redis Pattern*. Please feel free to goof around with
this console as much as you'd like.

Check out the following links.

* [Redis Documentation](http://redis.io/documentation)
* [Command Reference](http://redis.io/commands)
* [Implement a Twitter Clone in Redis](http://redis.io/topics/twitter-clone)
* [Introduction to Redis Data Types](http://redis.io/topics/data-types-intro)
* [Pattern source by Redis Labs](https://redislabs.com/redis-best-practices/data-storage-patterns/object-hash-storage/)
* [Acadevmy](https://acadevmy.it)