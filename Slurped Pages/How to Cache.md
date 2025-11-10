---
link: https://nyadgar.com/posts/how-to-cache/
excerpt: |-
  By Noam Yadgar
   
   at August 15, 2022
slurped: 2025-11-08T11:32
title: How to Cache | nyadgar.com
tags:
  - cache
---

[By Noam Yadgar](mailto:noam.g4@gmail.com) at August 15, 2022

Caching is one of those things that almost every digital service uses in some way or another, it can improve user experience, and reduce costs by preventing redundant computation. When incorrectly applied, it can be the root cause of many nasty issues such as serving stale data, creating security breaches, and eating up the memory. In this article, we will see what cache exactly is, what types of cache we can implement, and how to implement them correctly.

## What is Cache?

[Cache](https://en.wikipedia.org/wiki/Cache_%28computing%29) always comes in the context of data and it has a very straightforward reason to be applied. If data is being requested more than once, to avoid redundant computation (such as fetching it from a database, processing it, etc.) we can temporarily store the processed data in memory (or generally, locally) so it can be served faster and effortlessly. Even though most of your users will be unaware of its existence, they will appreciate it. Since good caching will improve their user experience.

## Ground rules for caching

Caching is great, however, the benefits of caching are diminished in comparison to the destructive effect of using cache incorrectly. Let’s establish some ground rules to help us avoid fatal mistakes when implementing a cache mechanism:

- **Your program should be fully functional without a cache**: In general, we shouldn’t depend on the state of the cache. For example, if you’re writing a stateless _REST API_, and you want to cache some of the resources, your API should be fully functional even if you shut down the cache feature.

- **Limit the size**: As your program is being used, cache is naturally being accumulated and growing in size. Since it’s very hard to decide (sometimes impossible) how much cache will be accumulated, you should be aware of the limits you allow the cache to grow. Even if you’re using a cache database as a cloud service with auto-scaling features, you should still set a threshold. A locally managed cache (on a running server) can drain the server’s memory and shut down the process. And an auto-scaled cache database as a cloud service, can do the opposite effect of caching by increasing complexity and cost.
    
- **Have a clear definition of permissions**: This is relevant to _server-side, distributed caching_ (more on that later). Serving a publicly available, cached blog post (_page-cache_) is easy since the whole world has permission to see the data. But a bank API might hold extremely sensitive and private data in its cache. In that case, we should make sure that we know how to structure the cache correctly, so it will be at a user-based access scope. Not being careful enough, it’s easy to accidentally create a data breach, where unauthorized users are being served with private cached data since the cached data may have bypassed all of the authentication and authorization barriers.
    
- **Pick the right cache strategy for data retention**: Understanding how persistent you want the cache to be, will help you build or use a more suitable cache mechanism. Ask yourself, if the program or session ends, do you still want the cache to be around? If so, maybe you should manage your cache in a dedicated service (such as [Redis](https://redis.io/) ) or implement a read/write cache mechanism that will persist on a disk. Ask yourself for how long you want the cache to be around.
    
- **Protect the cache as if you’re protecting the data**: If you’re caching data, you should protect the cache as if you’re protecting the data itself. To avoid _cache poisoning_ attacks, or stolen data, you should make sure that the cache is not easily accessible by unauthorized users.
    

## Where to cache?

Different types of cache can be implemented in the same program

Depending on your program’s behavior, caching can be done in different locations.

### Client-side cache

As users interact with your app, you can cache data directly on their devices. Depending on how persistent you want the data to be, you can store it directly in their devices’ memory, or dedicated locations on their disks (such as _local storage_). Media streaming apps are a great example of caching data on the client side. While you’re watching a movie or listening to an album, the app is caching the next bits of data on your device. This way, the data is being served smoothly even if the network becomes unstable. Some music app might even cache your entire playlist on your device, so you can listen to it offline.

### Server-side (backend) cache

In comparison to the _client-side cache_, caching on the server side has a broader definition. The word _server_ here really means - anywhere but the client side. Follow this example of a bank app, where a user is requesting their transaction history, the bank app is asking an API to fetch the data, which the API asks from a separate microservice with a cache mechanism.

sequenceDiagram
    Actor User
    participant Client app
    participant REST API
    participant Microservice
    participant Cache

    User ->> Client app: Clicks on "transactions" button
    Client app ->> REST API: GET /transactions
    REST API ->> Microservice: fetchProcessedData()
    Microservice ->> Cache: findInCache()
    Cache -->> Microservice: Respond with data or not found
    Opt If Not found
      Microservice ->> Microservice: Do all of the work of fetching from database and processing...
      Microservice ->> Cache: Set data
    End
      Microservice -->> REST API: Respond with data
      REST API -->> Client app: Respond with data
      Client app -->> User: Display data

Typically, we wouldn’t like to dive as deep into the stack to fetch the cache. The whole point of caching is to avoid redundant computation. That includes calling different services down the stack. Ideally, we would like to have the cache mechanism relatively close to the client. In the bank example, we could have set the cache at the REST API level, provided we can maintain security and privacy.

## Types of cache

Let’s start simple and build our way up as we define different strategies for our cache. I’ll be writing this in Go, but the same principles can be applied in other programming languages. The examples are all going to be a thread-safe, _distributed-cache_ type. That is a cache that can be accessed by many users as part of distributed software (like a web server)

I’m not going to use _off-the-shelf_ technologies such as _Redis_, since the purpose of this article is to introduce caching from under the hood.

### Simple hash-map

Storing cache in a map structure is generally better in comparison to arrays (_slices_ in go). Although maps are relatively heavier in memory, they are much faster. Fetching a cache entry from a map is an \(O(1)\) operation. Let’s start by defining an `interface` for our cache.

```
type Cache interface {
	Set(key string, value any) error
	Get(key string) (any, error)
	Delete(key string) error
	Clear() error
}
```

Let’s implement this `Cache` interface

```
type simple struct {
	size int32              // capacity
	used int32              // how much cache has been used
	data map[string]any     // content map
	mx   *sync.Mutex        // allowing concurrent access
}

func NewCache(size int32) Cache {
	s := int32(100)
	if size > 0 {
		s = size
	}
	return &simple{size: s, data: make(map[string]any, s), mx: &sync.Mutex{}}
}

func (c *simple) Set(key string, value any) error {
	c.mx.Lock()
	defer c.mx.Unlock()

	if c.used >= c.size {
		return fmt.Errorf("cache is full")
	}

	if _, ok := c.data[key]; !ok {
		c.used++
	}

	c.data[key] = value
	return nil
}

func (c *simple) Get(key string) (any, error) {
	c.mx.Lock()
	defer c.mx.Unlock()

	if v, ok := c.data[key]; ok {
		return v, nil
	}

	return nil, fmt.Errorf("key %s not found", key)
}

func (c *simple) Delete(key string) error {
	c.mx.Lock()
	defer c.mx.Unlock()

	if _, ok := c.data[key]; !ok {
		return fmt.Errorf("key %v not found", key)
	}

	delete(c.data, key)
	c.used--
	return nil
}

func (c *simple) Clear() error {
	c.mx.Lock()
	defer c.mx.Unlock()

	c.data = make(map[string]any, c.size)
	c.used = 0
	return nil
}
```

Simple yet, effective. If we haven’t reached the limit, this cache lets us set a new key-value pair in a concurrently shared `map` and get values from this `map`. If we want to discard the cache, we can invoke the `Clear()` method, which resets the cache to its initial values.

### TTL (Time To Live)

At some point, our cache will be full and we will no longer be able to append new entries to it unless we invoke `Clear()` or explicitly `Delete(key)`. In an ever-changing data system, this creates an issue with the cached data. Older and stale data might be cached and newer data might never be cached. Assuming we don’t have specific keys that we want to remove from the cache, we can set up a cycle with a timer that will `Clear()` the cache every `n` units of time, **or we can do better** by attaching a _self-destruct_ mechanism to each cache entry, in the form of _TTL (Time To Live)_.

I’ll be focusing just on the `Set` method, since the other methods stay exactly the same as the previous example.

```
type simple struct {
	size int32
	used int32
	ttl  int16 // in seconds
	data map[K]V
	mx   *sync.Mutex
}

func NewCacheWithTTL(size int32, ttl int16) Cache {
	s := int32(100)
	if size > 0 {
		s = size
	}
	return &simple[K, V]{size: s, data: make(map[string]any, s), mx: &sync.Mutex{}, ttl: ttl}
}

func (c *simple) Set(key string, value value) error {
	c.mx.Lock()
	defer c.mx.Unlock()

	if c.used >= c.size {
		return fmt.Errorf("cache is full")
	}

	if _, ok := c.data[key]; !ok {
		c.used++
	}

	c.data[key] = value

	if c.ttl > 0 {
		ctx, _ := c.setDeadline(key)
		go c.destroy(ctx, key)
	}

	return nil
}

func (c *simple) setDeadline(key string) (context.Context, context.CancelFunc) {
	return context.WithDeadline(context.Background(), time.Now().Add(time.Duration(c.ttl)*time.Second))
}

func (c *simple) destroy(ctx context.Context, key string) {
	<-ctx.Done()
	c.Delete(key)
}
```

Whenever we’re setting a new cache entry, we’re setting up a `Context` with a deadline, then, we’re spawning a new `goroutine` that will delete the cache entry (and decrement the counter by 1) when the context’s deadline has been reached. A _TTL_-based cache is a good strategy, if your program is interacting with data that is constantly changing (for example an _activity log_). It has an automatic _self-refresh_ mechanism that ensures that given enough time, users will get the latest results.

### Delete when data updates

What If the data barely changes? On one hand, it seems unnecessary to constantly delete cache entries. On the other hand, how can we make sure that our program is serving the latest changes of the requested data? We can write our program in such a way, that whenever we update a resource, we also call `Delete(Key)` to remove its entry (if it exists) from the cache. This will ensure that the next time this resource is being requested, it will not be fetched from the cache and it will be re-added to the cache with the new data.

### LRU (Least Recently Used)

Until now, we’ve only discussed strategies to preserve _data quality_ and _integrity_. What about _data priority_? In some systems (like a blog), data is being requested unequally. Statistically, some resources will be more popular than others. In many cases, they would naturally be distributed under [Zipf’s law](https://en.wikipedia.org/wiki/Zipf%27s_law) . It would be a bit wasteful to let a very unpopular piece of data, occupy a cache entry. In that case, we need a strategy that preserves this statistical nature and ensure that popular resources are more likely to be cached than unpopular resources.

We can use LRU. A [Doubly Linked List](https://en.wikipedia.org/wiki/Linked_list) can be used to prioritize cache entries since it is naturally **ordered** from _head_ to _tail._ It’s also very easy to _pop_ and _unshift_ nodes within the list. The problem with basing a cache on a linked list is searching. This is because finding a node in a linked list requires traversing the list, which is a potentially \(O(n)\) operation.

The solution is to have a hash-map that points to the list’s nodes. In this setup, we enjoy the \(O(1)\) when searching for an entry, and the ordered nature of a linked list. Off course, nothing is free here - An _LRU_ cache is relatively memory expensive in comparison to other more simple cache types. If a resource is being requested, we bring it up to the _head_ of the list. By keeping a size limit for the list, we can _pop_ nodes exceeding the list’s tail. This way, popular resources are more likely to _survive_ on the list, while unpopular are naturally discarded.

An _LRU_ cache is a good candidate for a _Page-cache_. A relatively small-capacity cache with large static resources and minimum updates (caching blog posts with _LRU_ is a good idea 🙂 ).

Let’s implement the `Cache` interface with an _LRU_ cache, we’ll start with defining the `structs`:

```
type lru struct {
	size  int32
	used  int32
	head  *node
	tail  *node
	cache map[string]*node
	mx    *sync.Mutex
}

type node struct {
	value any
	key   string
	next  *node
	prev  *node
}

func NewLRUCache(size int32) Cache {
	return &lru{
		size:  size,
		cache: make(map[string]*node, size),
		mx:    &sync.Mutex{},
	}
}
```

We will add three private methods:

- `unshift` : One that raises/puts a node to the `head`
- `pop`: One that discards the current `tail`
- `pull`: One that pulls out a node from the middle and _stitches_ its surrounding nodes

```
func (l *lru) unshift(n *node) {
	if l.head == nil {
		l.head = n
		l.tail = n
		return
	}

	n.next = l.head
	l.head.prev = n
	l.head = n
}

func (l *lru) pop() {
	delete(l.cache, l.tail.key)
	l.tail.prev.next = nil
	l.tail = l.tail.prev
}

func (l *lru) pull(n *node) {
	if n.prev != nil {
		n.prev.next = n.next
	}

	if n.next != nil {
		n.next.prev = n.prev
	}
}
```

Whenever we `Set` a new cache entry, we raise it to the head of the list (using `unshift`). If we are exceeding the cache’s capacity, we’ll `pop` the tail of the list.

```
func (l *lru) Set(key string, value any) error {
	l.mx.Lock()
	defer l.mx.Unlock()

	if n, ok := l.cache[key]; ok {
		n.value = values
        l.pull(n)
		l.unshift(n)
		return nil
	}

	n := &node{key: key, value: value}
	l.unshift(n)
	l.cache[key] = n
	l.used++

	if l.used > l.size {
		l.pop()
		l.used--
	}

	return nil
}
```

Whenever we `Get` a cache entry, we raise the requested node to the `head` of the list. However, we must be careful not to break the links of our list. If the cache’s `key` is pointing to a node that is somewhere in the middle of the list, we have to make sure we’re closing the gap between `prev` and `next` properties of the node, this is why we’re calling `pull` before `unshift`.

```
func (l *lru) Get(key string) (any, error) {
	l.mx.Lock()
	defer l.mx.Unlock()

	if n, ok := l.cache[key]; ok {
		l.pull(n)
		l.unshift(n)
		return n.value, nil
	}

	return nil, fmt.Errorf("key %v not found", key)
}
```

I will skip the `Clear` and `Delete` methods, as they hold no significant change from the previous examples.

### Writing the cache to a disk

In many cases, we’d like to decouple the content of the cache from the running process. This strategy can be useful when we like to preserve the cache even after the process has ended, so we could load it to a new process. Or, let different processes (or programs) load the same cache from a disk (local or remote). You may be tempted to append cache entries to a file when `Set` is being invoked. However, this might harm the performance of the cache, since writing to a file may be a time-consuming operation.

The effect is even more dramatic when we’re dealing with _distributed-cache_, where all of the read/write operations are performed under mutual exclusion.

The solution is to _dump_ a cache file whenever we’re calling `Clear`, and load this file whenever we’re calling the cache constructor. (We can even call `Clear` when our program receives a termination signal from the OS). Let’s write an interface to write and load a cache file, and add its logic to our `Simple` cache implementation.

```
type File interface {
	Load() ([]byte, error)
	Dump([]byte) error
}
```

Now we can add some logic to our constructor and `Clear` method (with a little bit of refactoring)

```
type simple struct {
	size int32
	used int32
	ttl  int16 // in seconds
	data map[string]any
	mx   *sync.Mutex
	file File
}

type Opts struct {
	Size int32
	TTL  int16
	File File
}

func NewCache(opts Opts) Cache {
	s := int32(defaultSize)
	if opts.Size > 0 {
		s = opts.Size
	}

	var used int32
	data := make(map[string]any, s)

	if opts.File != nil {
		if bytes, err := opts.File.Load(); err == nil {

			if err = json.Unmarshal(bytes, &data); err != nil {
				log.Printf("error unmarshal cache data: %v", err)
			} else {

				l := int32(len(data))
				if l > s {
					log.Printf("cache data size %v is larger than cache size %v", l, s)
					data = make(map[string]any, s)
				} else {
					used = l
				}

			}

		} else {
			log.Printf("loading cache data failed: %v", err)
		}
	}

	return &simple{
		size: s,
		used: used,
		data: data,
		mx:   &sync.Mutex{},
		ttl:  opts.TTL,
		file: opts.File,
	}
}

func (c *simple) Clear() error {
	c.mx.Lock()
	defer c.mx.Unlock()

	if c.file != nil {
		bytes, err := json.Marshal(c.data)
        if err != nil {
          return err
        }
		if err := c.file.Dump(bytes); err != nil {
			return err
		}
	}

	c.data = make(map[string]any, c.size)
	c.used = 0
	return nil
}
```

There are a lot of things to discuss about cache, but this is where I’m going to end this relatively long article. I hope you’ve found this article interesting and enriching. If you have any suggestions to improve the platform and content, please let me know.

Thank you for reading.

We would like to use third party cookies and scripts to improve the functionality of this website. Approve Deny [More info](https://nyadgar.com/privacy)