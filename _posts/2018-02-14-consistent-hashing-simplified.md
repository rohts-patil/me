---
layout: post
title: "Consistent Hashing Simplified"
date: 2018-02-14
excerpt: "Examples and code for displaying images in posts."
tags: [consistent hashing, caching, distributed system]
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

### Figures (for images or video)

#### One Up

<figure>
	<img src="http://farm9.staticflickr.com/8426/7758832526_cc8f681e48_c.jpg"></a>
	<figcaption><a>Morning Fog Emerging From Trees by A Guy Taking Pictures, on Flickr</a>.</figcaption>
</figure>

Vero laborum commodo occupy. Semiotics voluptate mumblecore pug. Cosby sweater ullamco quinoa ennui assumenda, sapiente occupy delectus lo-fi. Ea fashion axe Marfa cillum aliquip. Retro Bushwick keytar cliche. Before they sold out sustainable gastropub Marfa readymade, ethical Williamsburg skateboard brunch qui consectetur gentrify semiotics. Mustache cillum irony, fingerstache magna pour-over keffiyeh tousled selfies.

#### Two Up

Apply the `half` class like so to display two images side by side that share the same caption.

{% highlight html %}
<figure class="half">
    <a href="/images/image-filename-1-large.jpg"><img src="/images/image-filename-1.jpg"></a>
    <a href="/images/image-filename-2-large.jpg"><img src="/images/image-filename-2.jpg"></a>
    <figcaption>Caption describing these two images.</figcaption>
</figure>
{% endhighlight %}

And you'll get something that looks like this:

<figure class="half">
	<a href="http://placehold.it/1200x600.JPG"><img src="http://placehold.it/600x300.jpg"></a>
	<a href="http://placehold.it/1200x600.jpeg"><img src="http://placehold.it/600x300.jpg"></a>
	<figcaption>Two images.</figcaption>
</figure>

#### Three Up

Apply the `third` class like so to display three images side by side that share the same caption.

{% highlight html %}
<figure class="third">
	<img src="/images/image-filename-1.jpg">
	<img src="/images/image-filename-2.jpg">
	<img src="/images/image-filename-3.jpg">
	<figcaption>Caption describing these three images.</figcaption>
</figure>
{% endhighlight %}

And you'll get something that looks like this:

<figure class="third">
	<img src="http://placehold.it/600x300.jpg">
	<img src="http://placehold.it/600x300.jpg">
	<img src="http://placehold.it/600x300.jpg">
	<figcaption>Three images.</figcaption>
</figure>

### Alternative way

Another way to achieve the same result is to include `gallery` Liquid template. In this case you
don't have to write any HTML tags – just copy a small block of code, adjust the parameters (see below)
and fill the block with any number of links to images. You can mix relative and external links.

Here is the block you might want to use:

{% highlight liquid %}
{% raw %}
{% capture images %}
	http://vignette2.wikia.nocookie.net/naruto/images/9/97/Hinata.png
	http://vignette4.wikia.nocookie.net/naruto/images/7/79/Hinata_Part_II.png
	http://vignette1.wikia.nocookie.net/naruto/images/1/15/J%C5%ABho_S%C5%8Dshiken.png
{% endcapture %}
{% include gallery images=images caption="Test images" cols=3 %}
{% endraw %}
{% endhighlight %}

Parameters:

- `caption`: Sets the caption under the gallery (see `figcaption` HTML tag above);
- `cols`: Sets the number of columns of the gallery.
Available values: [1..3].

It will look something like this:

{% capture images %}
	http://vignette2.wikia.nocookie.net/naruto/images/9/97/Hinata.png
	http://vignette4.wikia.nocookie.net/naruto/images/7/79/Hinata_Part_II.png
	http://vignette1.wikia.nocookie.net/naruto/images/1/15/J%C5%ABho_S%C5%8Dshiken.png
{% endcapture %}
{% include gallery images=images caption="Test images" cols=3 %}
