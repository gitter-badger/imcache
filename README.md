imcache
=======
Imcache is a Java Caching Library.Imcache intends to speed up applications by providing a means to manage cached data. It offers solutions ranging from small applications to large scale applications. It supports n-level caching hierarchy where it supports various kind of caching methods like heap, offheap, and more. Imcache will also support well-known caching solutions like memcache and redis. To extend, you can define a heapcache backed by offheap cache which is also backed by database. If a key is not found in heap, it will be asked to offheap and so on. In order to use imcache, you need to specify your dependency as follows.

[![Build Status](https://travis-ci.org/Cetsoft/imcache.svg)](https://travis-ci.org/Cetsoft/imcache)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.cetsoft/imcache/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.cetsoft/imcache/)
[![License](http://img.shields.io/:license-apache-brightgreen.svg)](http://www.apache.org/licenses/LICENSE-2.0.html)

###Pom Reference
```xml
<dependency>
  <groupId>com.cetsoft</groupId>
  <artifactId>imcache</artifactId>
  <version>0.1.2</version><!--Can be updated for later versions-->
</dependency>
```
###Simple Application
```java
Cache<String,User> cache = CacheBuilder.heapCache().
cacheLoader(new CacheLoader<String, User>() {
    public User load(String key) {
        return userDAO.get(key);
    }
}).evictionListener(new EvictionListener<String, User>{
    public onEviction(String key, User user){
        userDAO.save(key, user);
    }
}).capacity(10000).build();
//If there is not a user in the heap cache it'll be loaded via userDAO.
User user = cache.get("#unique identifier"); 
User newUser = new User("email", "Richard", "Murray")
//When maximum value for cache size is reached, eviction event occurs.
//In case of eviction, newUser will be saved to db.
cache.put(newUser.getEmail(), newUser);
```
###The Cache Interface
Imcache supports simple operation defined by the cache interface. Cache interface provides general methods that is implemented by all imcache caches. See the methods below.
```java
public interface Cache<K, V> {
    void put(K key, V value);
    V get(K key);
    V invalidate(K key);
    void clear()
    double hitRatio();
    String getName();
}
```
###The Cache Builder
Cache Builder is one of the core asset of the imcache. You can create simple heapCaches to complexOffHeapCaches via 
Cache Builder. Let's see Cache Builder in action below.
```java
void example(){
    Cache<Integer,Integer> cache = CacheBuilder.heapCache().
    cacheLoader(new CacheLoader<Integer, Integer>() {
  	//Here you can load the key from another cache like offheapcache
        public Integer load(Integer key) {
            return null;
        }
    }).capacity(10000).build(); 
}
```
###The Cache Loader
The CacheLoader interface for loading values with specified keys. The class that is interested in loading values 
from a resource implements this interface. When data is not found the cache, load method of CacheLoader is called.
###The Eviction Listener
The listener interface for receiving eviction events. The class that is interested in processing a eviction event
implements this interface. When the eviction event occurs,that object's onEviction method is invoked.
###The Heap Cache
HeapCache uses LRU(Least Recently Used) as eviction strategy by the help of LinkedHashMap. As a result, 
HeapCache discards the least recently used items first when eviction required. Eviction occurs if the size of
the cache is equal to the cache capacity in a put operation.
###The Concurrent Heap Cache
ConcurrentHeapCache uses LRU(Least Recently Used) as eviction strategy by the help of ConcurrentLinkedHashMap. 
As a result, ConcurrentHeapCache discards the least recently used items firstwhen eviction required.
Eviction occurs if the size of the cache is equal to the cache capacity in a put operation
###The Off Heap Cache
The Class OffHeapCache is a cache that uses offheap byte buffers to store or retrieve data by serializing
items into bytes. To do so, OffHeapCache uses pointers to point array location of an item. OffHeapCache clears
the buffers periodically to gain free space if buffers are dirty(unused memory). It also does eviction depending on
access time to the objects.
To make offheap cache work to JVM Parameters <b>"-XX:MaxDirectMemorySize=4g"</b> must be set. Buffer capacity of 8 mb 
is a good choice to start OffHeapCache. Let's see sample OffHeapCache use.
```java
void example(){
    //8388608 is 8 MB and 10 buffers. 8MB*10 = 80 MB.
    OffHeapByteBufferStore bufferStore = new OffHeapByteBufferStore(8388608, 10);
    final Cache<Integer,SimpleObject> offHeapCache = CacheBuilder.offHeapCache().
    storage(bufferStore).build();
}
```
By default configuration, OffHeapCache will try to clean the places which are not used and marked as 
dirty periodically. What is more, it will do eviction periodically, too.

###The Versioned Off Heap Cache
The Class VersionedOffHeapCache is a type of offheap cache where cache items have versions that are incremented for each update.
To make versioned off heap cache work to JVM Parameters <b>"-XX:MaxDirectMemorySize=4g"</b> must be set. Buffer capacity of 8 mb 
is a good choice to start VersionedOffHeapCache. Let's see sample VersionedOffHeapCache use.
```java
void example(){
    //8388608 is 8 MB and 10 buffers. 8MB*10 = 80 MB.
    OffHeapByteBufferStore bufferStore = new OffHeapByteBufferStore(8388608, 10);
    final Cache<Integer,VersionedItem<SimpleObject>> offHeapCache = 
    CacheBuilder.versionedOffHeapCache().storage(bufferStore).build();
    VersionedItem<SimpleObject> versionedItem = offHeapCache.get(12);
}
```

###Searching, Indexing and Query Execution
imcache provides searching for all the caches by default. Searching is done by execute method of SearchableCache.
Execute method takes a Query as an input and returns results as list. A query consists of criteria and filter. Here
is an example use for queries.
```java
void example(){
    SearchableCache<Integer, SimpleObject> cache = CacheBuilder.heapCache().
    addIndex("j", IndexType.RANGE_INDEX).build();
    cache.put(0, createObject());
    cache.put(1, createObject());
    cache.put(2, createObject());
    List<SimpleObject> objects = cache.execute(CacheQuery.newQuery().
    setCriteria(new BetweenCriteria("j",1,3).or(new ETCriteria("j", 3))).
    setFilter(new LEFilter("k", 3)));
    for (SimpleObject simpleObject : objects) {
        System.out.println(simpleObject);
    }
}
```

<i>To learn more about imcache please look at examples provided.</i>
