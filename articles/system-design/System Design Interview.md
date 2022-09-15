    # CHAPTER 1: SCALE FROM ZERO TO MILLIONS OF USERS

## Web and Data Tiers 

**Failover**: switch to a redundant server upon the failure or abnormal termination of the previously active server.

**Failback**: to restore service from failure back to working state.

**Database replication**

1. master node only supports write operations, and slave nodes get copies from the master and support read operations
2. most applications require a much higher ratio of reads to writes 

**Tier Design**

1. A user gets the IP address of the load balancer from DNS.				
2. A user connects the load balancer with this IP address.
3. The HTTP request is routed to either Server 1 or Server 2.
4. web server reads user data from a slave database.
5. A web server routes any data-modifying operations to the master database. This includes write, update, and delete operations. 

## Cache Tier

**read-through cache**: After receiving a request, a server first tries to fetch data from the cache. If the data exists in the cache, it sends the data back to clients, otherwise, it queries the database, and stores the response in cache, followed by sending it back to the client.

**Considerations for using cache**

1. **expiration policy**: It is advisable neither to make the expiration date too long as the data can become stale nor to make the expiration date too short as it will cause the system to reload data from the database too frequently
2. **Eviction Policy**: Once the cache is full, any requests to add items to the cache might cause existing items to be removed. Least-recently-used (LRU) is the most popular cache eviction policy

**how CDN works**: when a client requests for static content, a CDN server closest to the client will deliver the data. If the CDN server does not have the data in cache, it will request the data from the original server. Static assets are no longer served by web servers. They are fetched from the CDN for better performance.

![reveser-proxy](https://raw.githubusercontent.com/ZintrulCre/warehouse/master/resources/system-design/service-tier.png)