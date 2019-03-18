---
layout: post
title: "Murmur Hashing Simplified"
date: 2019-03-15
excerpt: "Basics of Murmur hashing."
tags: [Murmur hashing, chi-squared test]
comments: true
---

#### Distributed system problem:-
We want to dynamically add/remove cache servers based on usage load.

As these are cache servers, we have a set of keys and values. This could be Memcached, Redis, Hazelcast, Ignite, etc.

Such setups consist of a pool of caching servers that host many key/value pairs and are used to provide fast access to data originally stored (or computed) elsewhere. For example, to reduce the load on a database server and at the same time improve performance, an application can be designed to first fetch data from the cache servers, and only if it’s not present there — a situation known as cache miss — resort to the database, running the relevant query and caching the results with an appropriate key, so that it can be found next time it’s needed. We want to distribute the keys across the servers so that we can find them again.

Our goal is to design a system such that:

1. We should be able to distribute the keys uniformly among the set of “n” servers.
2. We should be able to dynamically add or remove a server.
3. When we add/remove a server, we need to move the minimal amount of data between the servers.

Here is the simplest approach:-
1. Generate a hash of the key from the incoming data. For example, in python, we would use the hash function.<br>
`hashValue = hash(key)`
2. Find out the server to send the data to by taking the modulo of the hashValue using the number of current servers(n):<br>
`serverIndex = hashValue % n`

Now consider the following scenario:-

* Imagine we have 4 servers
* Imagine our hash function returns a value from 0 to 7
* We’ll assume that “key0” when passed through our hash function, generates a hash value or 0, “key1” generates 1 and so on.
* The serverIndex for “key0” is 0, “key1” is 1 and so on.

The situation assuming that the key data is uniformly distributed is shown in the image below. We receive 8 pieces of data and our hashing algorithm distributes it evenly across our four database servers.

<figure>
	<img src="https://github.com/rohts-patil/me/blob/master/assets/img/consistent%20hashing/original%20server.JPG?raw=true">
	<figcaption><center>Sharding data across 4 servers</center>.</figcaption>
</figure>

Problem solved, right? Not quite — there are two major drawbacks with this approach, namely, Horizontal Scalability and Non-Uniform data distribution across servers.

#### Horizontal Scalability:-
The above scheme is not horizontally scalable. If we add or remove servers from the set, all our existing mappings are broken. This is because the value of “n” in our function that calculates the serverIndex changes. The result is that all existing data needs to be remapped and migrated to different servers. This might be a humungous task.

Let us see what happens when we add another server (server4) to the original pool of server. Notice that we’ll need to update 3 out of the original 4 servers which mean 75% of servers need to be updated.

<figure>
	<img src="https://github.com/rohts-patil/me/blob/master/assets/img/consistent%20hashing/adde%20new%20server.JPG?raw=true">
	<figcaption><center>Effect of adding a new server to the cluster and the redistribution of the keys.</center></figcaption>
</figure>

The effect is more severe when a server goes down as shown below. In this case, we’ll need to update ALL servers.

<figure>
	<img src="https://github.com/rohts-patil/me/blob/master/assets/img/consistent%20hashing/delete%20server.JPG?raw=true">
	<figcaption><center>Effect of removing a server from the cluster and the redistribution of the keys.</center></figcaption>
</figure>

#### Data Distribution — Avoiding “Data Hot Spots” in Cluster:-
We cannot expect a uniform distribution of data coming in all the time. There may be many more keys whose hashValue maps to server number 1 than any other servers, in which case server number 1 will become a hotspot for keys.

*Consistent hashing allows up to solve both these problems.*

#### What exactly is Consistent Hashing?
So, how can this problem be solved? We need a distribution scheme that does not depend directly on the number of servers, so that, when adding or removing servers, the number of keys that need to be relocated is minimized. Consistent hashing facilitates the distribution of data across a set of nodes in such a way that minimizes the re-mapping/ reorganization of data when nodes are added or removed. Here’s how it works:

Consistent Hashing is a distributed hashing scheme that operates independently of the number of servers or objects in a distributed hash table by assigning them a position on a hash ring. This allows servers and objects to scale without affecting the overall system. Here’s how it works:

1. Creating the Hash Key Space: Consider we have a hash function that generates hash values in the range [0,2³²-1). We can represent this as an array of integers with 2³² -1 slot. We’ll call the first slot x0 and the last slot x^n — 1.

<figure>
	<img src="https://github.com/rohts-patil/me/blob/master/assets/img/consistent%20hashing/horizontal%20hash.JPG?raw=true">
	<figcaption><center>Linear Hash Key Space.</center></figcaption>
</figure>

 2. Representing the hash space as a Ring: Imagine that these integers generated after hashing are placed on a ring such that the last value wraps around and forms a cycle.

 <figure>
	<img src="https://github.com/rohts-patil/me/blob/master/assets/img/consistent%20hashing/circular%20hash.JPG?raw=true">
	<figcaption><center>Circular Hash Key Space.</center></figcaption>
</figure>

3.  Placing servers on the HashRing: We’re given a list of servers to start with. Using the hash function, we map each server to a specific place on the ring. This simulates placing the four servers into a different place on the ring as shown below.

 <figure>
	<img src="https://github.com/rohts-patil/me/blob/master/assets/img/consistent%20hashing/servers%20on%20hashring.JPG?raw=true">
	<figcaption><center>Placing servers on a hash ring.</center></figcaption>
</figure>

4. Determining Placement of Keys on Servers: To find which server an incoming key resides on, we do the following:

* Calculate the hash for the key using the hash function.
* After hashing the key, we’ll get an integer value which will be contained in the hash space, i.e., it can be mapped to some position in the hash ring. There can be two cases:
1. The hash value maps to a place on the ring which does not have a server. In this case, we travel clockwise on the ring from the point where the key is mapped to until we find the first server. Once we find the first server traveling clockwise on the ring, we insert the key there. The same logic would apply while trying to find a key in the ring.
2. The hash value of the key maps directly onto the same hash value of a server — in which case we place it on that server.

Example: Assume we have 4 incoming keys: key0, key1, key2, key3 and none of them directly maps to the hash value of any of the 4 servers on our hash ring. So we travel clockwise from the point these keys maps to in our ring till we find the first server and insert the key there. This is shown below diagram.

<figure>
	<img src="https://github.com/rohts-patil/me/blob/master/assets/img/consistent%20hashing/keys%20on%20hashring.JPG?raw=true">
	<figcaption><center>Key placements on servers in a hash ring.</center></figcaption>
</figure>

5. Adding a server to the Ring: If we add another server to the hash Ring, server 4, we’ll need to remap the keys. However, only the keys that reside between server 3 and server 0 needs to be remapped to server 4. On average, we’ll need to remap only k/n keys, where k is the number of keys and n is the number of servers. In modulo based approach we needed to remap nearly all the keys.<br>The figure below shows the effect of inserting a new server4. As server 4 is between key3 and server4, key3 will be remapped from server0 to server4.

<figure>
	<img src="https://github.com/rohts-patil/me/blob/master/assets/img/consistent%20hashing/adding%20new%20server%20on%20hashring.JPG?raw=true">
	<figcaption><center>Effect of adding a server to the hash ring.</center></figcaption>
</figure>

6. Removing a server from the ring: A server might go down and consistent hashing scheme ensures that it has minimal effect on the number of keys and servers affected. <br>As we can see in the figure below, if server0 goes down, only the keys in between server3 and server 0 will need to be remapped to server 1. The rest of the keys are unaffected.

<figure>
	<img src="https://github.com/rohts-patil/me/blob/master/assets/img/consistent%20hashing/delete%20server%20hashring.jpg?raw=true">
	<figcaption><center>Effect of removing a server from the hash ring</center>.</figcaption>
</figure>

Hence we can say that consistent hashing successfully solves the horizontal scalability problem by ensuring that every time we scale up or down, we do not have to redistribute all the keys.

Now let us talk about the second problem of non-uniform distribution of data across servers.

To ensure object keys are evenly distributed among servers, we need to apply a simple trick: To assign not one, but many labels to each server on the hash ring.

So instead of having labels server0, server1, server2, server3 we could have, say server00…server03, server10…server13, server20…server23, and server30…server33 all interspersed along the circle.

As the number of replicas or virtual nodes in the hash ring increase, the key distribution becomes more and more uniform.

The factor by which to increase the number of labels (server keys), known as weight, depends on the situation (and may even be different for each server) to adjust the probability of keys ending up on each. For example, if server0 were twice as powerful as the rest, it could be assigned twice as many labels, and as a result, it would end up holding twice as many objects (on average).

<figure>
	<img src="https://github.com/rohts-patil/me/blob/master/assets/img/consistent%20hashing/multiserver%20on%20hashring.JPG?raw=true">
	<figcaption><center>Using virtual nodes/ replication to create a better key distribution in a hash ring.</center></figcaption>
</figure>

Now imagine server0 is removed. To account for this, we must remove labels server00…server03 from the circle. This results in the object keys formerly adjacent to the deleted labels now being randomly labeled server 3x and sever1x, reassigning them to server3 and server1.

But what happens with the other object keys, the ones that originally belonged in server3 and server1? Nothing! That’s the beauty of it: The absence of server0 labels does not affect those keys in any way. So, removing a server results in its object keys being randomly reassigned to the rest of the servers, leaving all other keys untouched.

And this is how consistent hashing solves the non-uniform distribution problem.

#### References:-
I found that the authors never published an extended version with proofs, even though they said they would. The closest thing to an extended paper is,

1. [Relieving Hot Spots on the World Wide Web](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.17.8503&rep=rep1&type=pdf)