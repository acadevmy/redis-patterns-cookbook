# Redis in Advertising Networks

In this chapter, we will explore various features where Redis can be effectively used in
a social network.
we will cover the following topics:

* Building a simple social network with authentication
* Designing friendship circles
* Handling posts and news feeds in our social network

we will build a simple social network using Redis as the backend.
We will also see how to design a social graph defining the social interactions or
relationships between the users.

To keep the system simple we will store the following information for a user: e-mail address, location, gender, and age.

---
## Storing user information

we will store basic information such as age, location,
gender, and e-mail address for our users. In the following commands, we will have
a global key named nextUid to create an auto incremented unique ID for every user:

>[INCR gbl:nextUid](#run) => 1000

>[SET uid:1000:email john.smith@packtpub.com](#run)

>[SET uid:1000:password d41d8cd98f00b204e9800998ecf8427e](#run)

We have created two keys, one with the e-mail ID and another key with the hashed password for a particular user.

For the login, we need to get the user ID from the e-mail ID of the user. To easily
get the user ID from an e-mail ID instead of iterating all the keys, we need to
create another reverse lookup key in Redis as follows:

>[SET email:john.smith@packtpub.com:uid 1000](#run)

We can store other information such as age, gender, and location of the user,
which we will not use to look up, as hashes:

>[HMSET uid:1000:info email john.smith@packtpub.com age 30 gender M location "Rome"](#run)

We have a global key, nextUid, which is used to increment and assign a user ID
to new users. All information about the user will be accessed using the user ID.

---

## Choosing the best ad to serve

Now, when a request comes in with website_id and slot_id, we need to process
the request and then decide what is the most compatible ad to serve that request
while also maximizing the eCPM for that request.

The following command on execution will give the ad unit, which has
maximum eCPM:

>[ZREVRANGEBYSCORE slot:banner +inf -inf LIMIT 0 1](#run)

---

## Authenticating a user

The basic steps to perform user authentication are:

1. Check if the e-mail exists in our system by retrieving the user ID for the
e-mail ID:

    >[GET email:john.smith@packtpub.com:uid](#run) => 1000

2. If the user ID does not exist, we will create a new user with the e-mail ID
and newly incremented value of gbl:nextUid.

3. If the user ID exists, we will validate the password provided by the user
by typing the following command:

    >[GET uid:1000:password](#run)

4. If the password matches, log in the user using the authentication method
of your application, such as cookie-based or session-based authentication.

---
## Adding a friend

To send a new friend request to a user, we will add the user ID of the requesting
user into the set. For example, if a user with ID 1000 wants to add a user with ID 453
as a friend, we will add the user ID 1000 into the friend request set of user 453 as
follows:

>[SADD uid:453:requests 1000](#run)

Also, we need to maintain all the relationships in a bidirectional manner. In order
to show all the pending friend requests, we need to have a set for requested users
as well. To do so, run the following command:

>[SADD uid:1000:pendingrequests 453](#run)

Now, to show all the new friend requests received by any user, we just need to get
the list of user IDs from the set and show these to the user. To show the new friend
requests to user 453, we will execute [SMEMBERS](#help) on the uid:453:requests key and
show the user's information.

---
## Accepting a friend request

Once the user accepts the friend request, we need to add the user ID to the friends
list of both the users. If a user with ID 453 accepts the friend request from the user
with ID 1000, we will run the following commands:

>[SADD uid:1000:friendslist 453](#run)

>[SADD uid:453:friendslist 1000](#run)

>[SREM uid:453:requests 1000](#run)

>[SREM uid:1000:pendingrequests 453](#run)

In the case where the user rejects the friend request, we will only delete the request
from the user's set and skip adding to the friends list as follows:

>[SREM uid:453:requests 1000](#run)

To avoid the racing condition in which two users send friend requests to each other
at the same time, we need to check user 1's request queue before adding user 1's ID
into user 2's request queue. If user 2's ID already exists in user 1's request queue,
we will make the users friends. As Redis is atomic and single threaded, both the
keys cannot be added at the same time and the previous check will avoid the racing
condition.

---

## Unfriending a user

If a user wants to unfriend another user, the operation is simple. We need to remove
the ID of the users from each other's friendslist set as follows:

>[SREM uid:1000:friendslist 453](#run)

>[SREM uid:453:friendslist 1000](#run)

---

## Updating posts and status

In a real-world social network, depending on the type of network, various kinds of
activities such as status updates, photos, links, and videos are allowed. However, for
this section, we will restrict activity types to posts and status updates only. The way
posts and status updates are handled is identical. So, we will store status updates as
posts in Redis.

The basic steps to be performed when someone is creating a new post are as follows:

1. The post information is saved into hash.
2. The post ID is stored into the author's post list.
3. The post ID is also updated into the newsfeed list of all the friends
of the person who created the post.

To create a new unique post ID every time, we will use a global key like we did
earlier for the user ID:

>[INCR gbl:nextPostId](#run) => 282

After getting the unique post ID, we will store the information related to the post in
a hash as follows:

>[HMSET post:282 by 1000 content "Hi all" posted_at "2014-08-10 23:28:14"](#run)

Once the hash is created, we will push the post ID into the status list of the user who
created it as follows:

>[LPUSH uid:1000:statuses 282](#run)

Also, we will push the status update to all the friends of the user. Using the
SMEMBERS function, get all the friends of the user and iterate through each user and
push the status update into their updates list:

>[SMEMBERS uid:1000:friendslist](#run)

>[LPUSH uid:453:updates 282](#run)

Now, if the user wants to edit the post that is already created, the operation is
simply a case of updating the hash values for the post using HSET. We can even
add modified time as another value to our post data just in case we want to notify
the users about last modified time of the post.

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