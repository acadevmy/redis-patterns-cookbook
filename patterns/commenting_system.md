# Redis in a Commenting System

In this pattern, we will look at how to use Redis as the primary data store.
In particular, we will see how to design a commenting system using Redis.
Any basic commenting system includes the ability
to post comments, and store and display the user-submitted comments along with
the normal blog content. Redis, being a flexible data store, provides many different
approaches for storing comments, depending on our product design.

---

## A nonthreaded comment system

We will see how to implement
a simple nonthreaded system followed by the threaded comments that need a few
architectural requirements.

Consider that we want to store the following information for any comment:

* `Comment ID:` This is a unique ID for comments in our system
* `Author:` This is the name of the user who has posted the comment
* `Comment text:` This is the full text of the comment along with any styling the user has provided
* `Comment timestamp:` This is the time when the comment was posted
* `URL Slug:` This is the URL link to the comment in case we want to
link comments

## Posting a new comment

Apart from this information related to comments, we should be able to find the post
for which the comments were made. As this is a nonthreaded comment system, we
can use lists to store the comment IDs for each post and store the comments in hashes:

>[LPUSH post:121:comments comment:4](#run)

>[HMSET comment:4 author "John Smith" text "Very informative post" timestamp "2014-05-17 23:00:34" Slug "/comment/4"](#run)

The commands above can be used to create a comment object for a post.

---

## Updating an existing comment

In case we want to update an existing comment, we can only update the hashes
without updating the set:

>[HMSET comment:4 author "John Smith" text "Very informative post but can be better." timestamp "2014-05-17 23:07:28" Slug "/comment/4"](#run)

In order to delete a comment from the system, we need to delete it from both sets
and also the hashes. Even though deleting the member from a set will suffice, it is
necessary to delete it from the hashes in order to avoid ghost keys that stay in Redis
without any reference to them:

>[DEL comment:4](#run)

>[LREM post:121:comments 0 comment:4](#run)

---

## Displaying the comments

Assuming we have not deleted the comment in the previous section, to retrieve the
comments for any post, we only need to know the post ID:

>[LRANGE post:121:comments 0 -1](#run) => "comment:4"

Now, iterate through all the members and fetch the comment hashes to display
the information:

>[HGETALL comment:4](#run)

1. "author"
2. "John Smith"
3. "text"
4. "Very informational post but can be better"
5. "timestamp"
6. "2014-05-17 23:07:28"
7. "Slug"
8. "/comment/4"


In order to paginate the comments, we can use LRANGE to fetch only a specific
number of comments from the list using the start and stop parameters of LRANGE.

>[LRANGE post:121:comments 0 10](#run)

The preceding command will give 10 comments for the post starting from the latest
comment. To get the next set of 10 comments, use the following command:

>[LRANGE post:121:comments 10 10](#run)

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