# System Design {docsify-ignore}
Personal System Design Notes for Interviews

## How to approach a System Design Interview?
###### Step 1 - Outline use cases, constraints and assumptions.<br>
!> Basically Gather requirements and scope. Ask questions about users/load/times/how/why/what and all.
###### Step 2 - Outline and create a high level design first.
###### Step 3 - Design Core components
###### Step 4 - Scale the Design

### System Design Topics:

#### Scalability

##### Vertical Scaling
- Definition: Increase in hardware on the same system or machine.
- Caveat - Only to a particular ceiling
- A multi core's cpu can handle multiple processing or true parallel processing
- CPU 
  - Cores, 
  -  L2 Cache, etc.,.
- DISK 
  - PATA, 
  - SATA(7200 rpm), 
  - SAS(Serial Attach Skuzzy 15000 rpm), 
  - SSD(Solid State Drive - No disk)
  - RAID
- RAM 

##### Horizontal Scaling
- Definition: Increase number of machines with some cheaper hardware maybe
- Increase the machines as needed, not adding hardware on the same machine

##### Load Balancing
- Actual Load [May be too complex to implement and not so reliable]
- Can be based on the type of data too. May be a DNS server can be used with type or ip address mapping [Not so good]
- Basically we can also implement Round robin using BIND technology
- Isse with the above is, when any of the DNS based servers are busy, then the load has to be moved to next one. Does the software take care of it. May not be.
- Round robin is not so good - only returns one ip address. May be enhance the same.
- Now the problem is session management

##### Understanding RAID
- **RAID 0** - 2 Hard drives --> Both are partially(some data here and soem data there) writing data together to the disks
- **RAID 1** - 2 Hard drives --> But diff with `RAID 0` is, `RAID 1` mirrors the data exactly.
- **RAID 5** - Same as `RAID 1` but can grow bigger as needed. From 2 to 4 to 6 and so on.
- **RAID 6** - Even 2 of them died, thats OK. --- `FIXME`
- **RAID 10** - A combination of both `RAID 0` and `RAID 1`

**Coming back to the session management issue:**
##### Sticky sessions
- Sessions should be preserved for a lot of reasibs
- Shared storage
    - Fiber Channel
    - My SQL or any other database supports sessions
- Cookies?
    - May be ID of the server like UUID (Universally Unique ID)
    
##### Software Load Balancer
- Elastic LB (ELB)
- HAProxy (High Availability Proxy)
- LVS (Linux Virtual Service)

##### Hardware Load Balancer
- Barracuda
- Cisco
- Citrix
- F5

##### Caching mechanism
- `.html` 
    - May be the pre generated HTML can be cached
    - May be the problem is, if any change needs on how this to be displayed, like CSS or anything, then all thos html's may have to be changed.
- `MySQL Query cache`
    - Mysql will cache the query and gives back the data from the cache until the query is different or params changed
- `memcached` or 'Redis' may be a good one too for in memory
    - The mechanism of `memcached` is to store anything in RAM
    - Eventually cache gets so big and RAM would be full.
    - What to do?
        - Removes or expires the Least recently used cache called `LRU` mechanism
        - memcached will take of this whole stuff
    - Cachind DB queries - May be good but what if we have to handle multiple queries for one request?
    - Caching hte final objects - A complete instance is always good in my opinion.
    
##### Data Replication
- **Master - Slave**
    - All the machines in this arechitecture are identical for data.
    - Either for redundancy or we can query the slaves as needed.
    - May be there is some downtime to convert a slave to master when the master dies.
- **Master - Master**
    - Write (or) Read through both the masters.
    - No need of any downtime to promote slave to master as the other one is already a master
    - A Load balancer should be able to route the load to these two masters. Is this acceptable?

    !> **Single Point of failure** - Something where one spot fails, everything else fails.
- **Load Balance**
    - Active-Active - Heartbeating technology can help switch
    - Active-Passive - Heartbeating technology can help witch
    - Usually Load balancers comes in pairs when we bought them.
    
##### Data Partitioning
- Data can be partitioned like, usernames A-M in one cluster or DB and M-Z in another cluster or DB.
  
##### High Availability
- Definition: With very less downtime
- This is the key property of any server architecture or apps in any given scenario
- Load balancing on DB may be a bit complicated to implement than that on the app layer.
- We can even consider having multiple `switches` for network to avoid any network redundancy.
- Also consider geography based load balancing when using the DNS based LB. This may be more for cloud based deployment.
  - May be termed as `Global Load Balancing`
  
##### Security
- Before the Load balancer
    - TCP:80, TCP:443(for SSL), TCP:22(for SSH)
    - By the time it passes through LB, everything should be already secure.
- After load balancer:
    - TCP:80
    - TCP:3306 for DB(eg. for Mysql)
    - REad about Firewall'ing
    
##### Asynchronism
- **Async #1**
    - Time consuming work in advance and serving the finished work with low request time
    - Dynamic content to static
    - Then the static html pages can be uploaded to AWS S3 or CloudFront or any other CDN.
- **Async #2**
    - Come back later kind of scenario
    - A queue of tasks or jobs that a `service worker` can process.
    - RabbitMQ, ActiveMQ, Redis List,..
   
##### Performance vs Scalability
- `Performance Problem` --> System is slow for a single user
- `Scalability problem` --> System is fast for single user but slow under heavy load.

##### Latency vs Throughput
- `Latency` --> Time to perform some actions or produce some result
- `Throughput` --> No. of such actions or results per unit of time.
        > **AIM** -- MAX throughput with acceptable latency
        
##### Availability vs Consistency
- ###### CAP Theorem
- Consistency: Every read gets most recent write
- Availability: Every request receives response without guarantee of latest write
- Partition Tolerance: Continues to work despite of the arbitrary partitioning(Its basically network redundancy)
> Networks are not reliable, so support of `partition tolerance` is needed.

CP - Consistency and Partition Tolerance
    - 