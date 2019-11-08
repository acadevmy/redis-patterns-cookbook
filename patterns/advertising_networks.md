# Redis in Advertising Networks

In this pattern, we will examine how to build an online advertising network using
Redis. Advertising networks connect the advertisers and publisher websites.

Advertisers provide the display ads along with a set of attributes for the ad that should be displayed.

Publisher websites provide the content pages with regions marked for ad displays.

When the page is displayed, the request is made to the ad network and relevant ads are shown in the designated ad slot.

Typically, an ad network tracks and records the number of page views, the number
of times each ad is displayed, the number of clicks on ads, and other statistics that are required to optimize and bill the advertiser.

The key performance criteria for an ad-serving network is based on the time taken
from receiving a request to returning a targeted advertisement to be displayed.
Redis, being blazingly fast, is well suited for this requirement.

The basic steps are as follows:

1. The network receives a request from a website with its website_id and
slot_id for which the ad is to be served.
2. The network searches in its ad catalog, which contains all active ad
campaigns, and chooses the best ad based on business rules.
3. The network sends the ad to be displayed and implements a view for the ad.

For simplicity, slot_id will be of these types: banner, skyscraper, and square ad
blocks. In order to store the different ads, we will use the sorted set for each type, ordered and based on its effective cost per mile (eCPM) values (eCPM = CPC × CTR × 1000).

---

## The ad inventory

We will create a sorted set for each type of ads (slot_id) and the sorted set
will contain ad_id, which will be assigned to any ad that is added to the system
ordered by the eCPM value for that ad:

>[ZADD slot:banner 2.25 ad:1](#run)

>[ZADD slot:banner 4.75 ad:2](#run)

>[ZADD slot:banner 1.65 ad:3](#run)

Other information related to the ad will be stored as hashes. The information
includes campaign_id and the ad snippet to be served. We will add more
information related to the ad in this hash:

>[HMSET ad:1 campaign_id 100 slot_id banner ad_snippet {html-fragment}](#run)

>[HMSET ad:2 campaign_id 100 slot_id banner ad_snippet {html-fragment}](#run)

>[HMSET ad:3 campaign_id 101 slot_id banner ad_snippet {html-fragment}](#run)

A single campaign can have multiple ad units. In order to manage the status of
the campaign, we can create a set that will contain all the ad_id belonging to
that campaign:

>[SADD campaign:100 ad:1 ad:2](#run)

>[SADD campaign:101 ad:3](#run)

---

## Choosing the best ad to serve

Now, when a request comes in with website_id and slot_id, we need to process
the request and then decide what is the most compatible ad to serve that request
while also maximizing the eCPM for that request.

The following command on execution will give the ad unit, which has
maximum eCPM:

>[ZREVRANGEBYSCORE slot:banner +inf -inf LIMIT 0 1](#run)

---

## Pausing or stopping a campaign

In order to stop or pause a campaign, all we need to do is remove the ad from the
respective slot's sorted set. To stop campaign:100, type the following command:

>[SMEMBERS campaign:100](#run)

* ad:1
* ad:2

Iterate through ad_ids, and remove the ad from the selection pool:

>[ZREM slot:banner ad:1 ad:2](#run)

From the next request, the ad units as part of the specific campaign will not be
delivered to the websites.

---

## Frequency capping

In our previous solution, there was a possibility of the same ad being shown
to the same user over and over again until the budget was exhausted as we deliver
in order to maximize eCPM for the website. To solve this issue in the advertising
world, the advertiser is given the ability to limit the frequency with which the same user is presented with the same ad. This methodology is called frequency capping.

As frequency capping is based on the users, we need to maintain a data store to
manage the user profile information, such as the impression count for each ad and
conversion. The user_id will usually be maintained as a cookie in the user's browser in order to track the user across the session. Every time a request for an ad comes to the network, the user_id from the cookie is used in the ad-serving decision.

Impression information for a user should be stored per campaign in order to
make sure that the ad from the same campaign is not shown every time. This
can be achieved with the help of the following commands:

>[INCR user:id1:campaign:100](#run)

>[INCR user:id1:campaign:101](#run)

>[INCR user:id2:campaign:102](#run)

Every time an ad is served to the user, we increment the impression count for that
particular user and for that campaign. Also, we will store the frequency capping
for each campaign provided by the advertiser.

---

## Selecting an ad with frequency capping

Now we need to iterate through the ad units ordered by the eCPM and find the best possible ad based on the frequency capping. To do so, we need to fetch all the ad units available for the particular slot id by typing the following command:

>[ZREVRANGEBYSCORE slot:banner +inf -inf](#run)

iterate through the ad units and check whether the user has reached the
capping limit for that campaign yet:

>[HGET ad:1 campaign_id](#run) => 100

>[GET user:id1:campaign:100](#run)

If the user has already reached the limit, continue the iteration; else, deliver
the ad snippet to the website and increment the impression count for that
user to the particular campaign.

---

## Keyword targeting

Keyword targeting is another important feature for any advertising network.
Advertisers would like to show their advertisements on related websites where
the audience of the website closely matches their target audience.

For the sake of simplicity, we will consider
ad targeting based on the search keyword provided by the user in the website and
will match only the exact matches of a keyword.

To accommodate keyword targeting in our system, we need to store the keywords
provided by the advertisers for each of the campaigns. In order to improve the
real-time performance, we will create a reverse index for the keywords and store
the campaign IDs as members of the keywords.

For example, keywords for the campaigns are as follows:
* Campaign_100 - android, phones, holo design
* Campaign_101 - windows, phones, flat design

Now, we will store the information as a reverse index in Redis:

>[SADD android campaign:100](#run)

>[SADD phones campaign:100 campaign:101](#run)

>[SADD "holo design" campaign:100](#run)

>[SADD "flat design" campaign:101](#run)

To add or remove the keyword from a campaign, we need to remove the campaign
ID from the keyword in the set or add it. To add the material design as the keyword to the campaign, run the following command:

>[SADD "material design" campaign:100](#run)

To remove holo design from the keyword list of a campaign, run the
following command:

>[SREM "holo design" campaign:100](#run)

---

## Selecting an ad based on keyword targeting

We will extend the previous solution of frequency capping and perform the following steps:

Once the request comes in with the keyword (phones), we will fetch all the
campaigns applicable for the keyword by typing the following command:

>[SMEMBERS phones](#run)

1. campaign:100
2. campaign:101

We will get all the ad units that are relevant, based on the keyword
targeting with the help of the following command:

>[SUNION campaign:100 campaign:101](#run)

The preceding command will unite all the ad units that are available as
part of both the campaigns.

We will fetch all the ad units applicable for the particular slot_id, the same
as we did earlier, by typing the following command:

>[ZREVRANGEBYSCORE slot:banner +inf -inf](#run)

Before we start iterating, we will perform the following operation:

* Intersect the two lists that we have from step 2 and step 3 to get
all the ad units matching both the keyword targeting and the slot ID
* Now, iterate through each ad unit id
* Get the campaign ID of the ad unit from the hash of the ad unit
* Check the user frequency for the particular campaign. If the
frequency capping limit is not met, deliver the ad unit. Else,
continue with the iteration

The ad unit delivered earlier will meet all the targeting parameters, including
frequency capping.

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