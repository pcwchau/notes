# Index
- [Index](#index)
- [NoSQL](#nosql)
  - [Concept](#concept)
  - [Limitation](#limitation)
  - [Replica set](#replica-set)
- [MongoDB](#mongodb)
  - [Data Structure](#data-structure)
  - [Replication (replica set):](#replication-replica-set)
    - [Status](#status)
    - [Set up](#set-up)
    - [Read preference mode](#read-preference-mode)
    - [Potential problem](#potential-problem)
  - [mongod (daemon)](#mongod-daemon)
  - [mongosh (shell)](#mongosh-shell)
  - [Script](#script)
  - [Generate large amount data (Python)](#generate-large-amount-data-python)
- [Redis](#redis)
  - [Local Setup](#local-setup)
- [Elasticsearch (a search engine, not exactly NoSQL)](#elasticsearch-a-search-engine-not-exactly-nosql)
- [Appendix](#appendix)
  - [Docker set up for mongodb replica set](#docker-set-up-for-mongodb-replica-set)

# NoSQL
## Concept
- Different from RDBMS (relational database)
- Can scale both vertically and horizontally (add more server to store more data is easy as `no relations`)
- Each item (row) only have `key` and `value`
  - Find an item by key (using a hash table)
  - No schema is used, data structure is flexible
  - Large amount of data is stored in a few collections
  - **Great performance for mass read** (write if only update one or few collections)

## Limitation
- hard to find an item by `value` (eg. find all item with price > $1000 is inefficient compared to RD)
- hard to do `join` queries   
![](../image/Snipaste_2022-07-13_18-08-13.png)

## Replica set
- Normally primary and secondary DB have the same content, but have different right. Usually cannot write to secondary DB.
- Secondary DB can be used as backup, or share workload of reading data with primary DB

# MongoDB
## Data Structure
- Storage can in both disk and memory
- Database -> Collection (Table) -> Document (Row)
  - A collection/database that does not exist will be created automatically if you write data to it
  - Document structure is flexible (every document can have different fields)
- MongoDB stores `BSON (Binary JSON)` documents and for each key-value pairs, the `BSON` contains the key and the value along with its type.
```json
// JSON
[
  {  // object
    "_id": "H8224001_S_HKTV_C1",  // immutable, string
    "price": {  // object 
      "2022-07-26": 102.33,  // double
      "2022-07-29": 186.32,  // double
      // "2022-07-29": 199.99  // ERROR, same key
    }
  }
]

// Extended JSON, add type information, readable BSON
[
  {
    "_id": "H8224001_S_HKTV_C1",  // immutable
    "price": {
      "2022-07-26": { "$numberDouble" : 102.33},
      "2022-07-29": { "$numberDouble" : 186.32}
    }
  }
]
```
- For each document, the `_id` field with an `ObjectId` would be auto generated for us if we didn't explicitly create one here but it's good practise to set the `_id` ourselves.
- `object` data type stores embedded documents (aka nested documents). By default, no key can be duplicated inside an `object`.

## Replication (replica set):
ref: https://www.mongodb.com/docs/manual/replication/

**Note that:**
- Client cannot write data to secondary DB

### Status
You can check in `/etc/mongod.conf` or check after enter mongosh/mongo shell to see whether replica set is initialized. To verify the replica set, use `rs.status()` in `mongosh`.

Replica Set Member States ref: https://www.mongodb.com/docs/manual/reference/replica-states/

### Set up
[Docker compose set up](#docker-set-up-for-mongodb-replica-set)

### Read preference mode
  - primary
  - primaryPreferred
  - secondary
  - secondaryPreferred
  - nearest

Read from secondary: `db.getMongo().setReadPref("primaryPreferred")`

**Note that:**
- All read preference modes except primary may return stale data because secondaries replicate operations from the primary in an asynchronous process.

### Potential problem
- Cannot add index sucessfully. Even later find that the index exists, it does not work (i.e. unique index cannot avoid duplicate insert). It is found that one server of the replica set is not reachable, connection refused, so cannot add index sucessfully.
  - Suggestion: Before creating index, use `rs.status()` to check first.

## mongod (daemon)
```
mongod --replSet rs0 --port 27019 --bind_ip localhost,mongo3
```
- `--port` - specify which port to connect to the mongodb (default is `27017`)

## mongosh (shell)
Note that some `mongosh` commands cannot be run on `mongo`.
```js
mongosh // enter the MongoDB shell using localhost:27017
mongosh --host mongodb0.example.com --port 28015 // other than localhost
mongosh "mongodb://hktv_pricechart_mongodb_1:27017" // use connection url

show databases
use <database_name>

db.getCollectionNames()  // show all collections inside this db
db.<collection_name>.renameCollection("xxxx")

// ======================== Query Records ========================
db.<collection_name>.find({ name: "Kyle"}, { name: 1}) // SELECT name FROM <collection_name> where name = "Kyle"
db.<collection_name>.find({ name: {$ne: "Kyle"}}) // ... where name != "Kyle"
db.<collection_name>.find({ age: {$gt: 10}}) // ... where age > 10
db.<collection_name>.find({ name: {$nin: ["Kyle", "Jill"]}}) // ... where name not in ('Kyle', 'Jill')

db.<collection_name>.find({ $and: [{ age: 26}, { name: 'Kyle'}]}) // ... where age = 26 and name = 'Kyle'

// Show all records in an array instead of BSONs
db.<collection_name>.find(xxx).toArray()


// ======================== Insert Records ========================
db.<collection_name>.insertOne({item: "card", qty: 15 })
db.<collection_name>.insertMany({item: "card", qty: 15 })


// ======================== Delete Records ========================
db.<collection_name>.deleteOne({item: "card"})
db.<collection_name>.deleteMany({item: "card"})


// ======================== Count Records ========================
// SELECT count(*) FROM xxx
db.<collection_name>.countDocuments()

// SELECT count(*) AS count FROM xxx GROUP BY dateTime
db.<collection_name>.aggregate([{$group: {_id:"$dateTime", count:{$sum: 1}}}])

// SELECT count(*) AS count, SUM(count) AS total FROM xxx GROUP BY dateTime
db.<collection_name>.aggregate([{$group: {_id:"$dateTime", count:{$sum: 1}, total: {$sum: "$count"}}}])

// SELECT count(*) AS count FROM xxx GROUP BY dateTime ORDER BY count DESC
db.<collection_name>.aggregate([{$group: {_id:"$dateTime", count:{$sum: 1}}}, {$sort: {"count": -1}}])

// SELECT count FROM (SELECT count(*) AS count FROM xxx GROUP BY dateTime) WHERE count > 1
db.<collection_name>.aggregate([{$group: {_id:"$dateTime", count:{$sum: 1}}}, {$match: {"count": {$gt: 1}}}])


// ======================== Update Records ========================
// UPDATE xxx SET age = 27 WHERE age = 26
db.<collection_name>.updateOne({ age: 26}, { $set: { age: 27}})


// ======================== Alias Field ========================
// SELECT p AS price FROM xxx WHERE _id = "H8224001_S_HKTV_C1"
db.<collection_name>.aggregate([{$match:{_id:"H8224001_S_HKTV_C1"}},{$project:{price:"$p"}}])


// ======================== Collection (Table) Index ========================
db.<collection_name>.createIndex({date: 1, item: 1}, {name: "xxx index", unique: true})
db.<collection_name>.getIndexes()

db.<collection_name>.dropIndex("xxx index")

```

## Script
Put all `mongo` (here not `mongosh`) command in a JS file, then use `mongo <js file path>`.

The JS file:
```js
db = connect("localhost:27017/hktv_pricechart");
/* This is for replica set
 * primary_db = db.hello().primary;
 * db = connect(primary_db);
*/

// use which DB
db = db.getSiblingDB('hktv_pricechart');

// print normal string
print(db.<collection_name>.count());

// pring BSON object
index = db.<collection_name>.getIndexes();
printjson(index)

// print all items in a result cursor
cursor = db.<collection_name>.find();
while ( cursor.hasNext() ) {
   printjson( cursor.next() );
}

// Another method to iterate a result cursor:
cursor.foreach(e => {
   print(e.<field_name>);
});

```
- cursor method ref: https://www.mongodb.com/docs/manual/reference/method/js-cursor/

## Generate large amount data (Python)
```sh
# import command
# Field names must be in the form of <colName>.<type>(<arg>)
# date(<format>)
mongoimport --uri mongodb://localhost:27017/myapp --collection=productPrice --type=csv --columnsHaveTypes --fields="skuCode.string(),membership.string(),date.date(2006-01-02T15:04:05.000Z),lowestPrice.double(),yesterdayLatestSyncTime.int64(),yesterdayLatestPrice.double()" --file=output.csv
```
```python
# generate csv without field name and type
import datetime
import csv
from enum import Enum

base = datetime.datetime.today()

class member(Enum):
    NORMAL = 1
    VIP = 2
    GOLDVIP = 3
    ORIGINAL = 4
    SENIOR = 5

counter = 0
with open('output.csv', 'w') as f:
    writer = csv.writer(f)
    for skucode in range(1200):
        for mem in member:
            for day in range(400):
                daytime = base - datetime.timedelta(day)
                f.write(f'H23984234_S_HKTV_TESTING_{skucode},{mem.name},{daytime.strftime("%Y-%m-%d")}T16:00:00.000Z,174.25,1589338195236,199.99\n')
                print(counter)
                counter = counter + 1
```

# Redis
- Storage in memory
  - can be as a cache instance between the server and database
  - can also be a primary database
  - persistence options
    - RDB (Redis Database): The RDB persistence performs point-in-time snapshots of your dataset at specified intervals.
    - AOF (Append Only File): The AOF persistence logs every write operation received by the server, that will be played again at server startup, reconstructing the original dataset.
- Key-value pair (string, up to 512 MB in length per string)
  - List (sorted by insertion order)
    - constant time insertion and deletion of elements near the head and tail
    - can use `LPUSH` together with `LTRIM` to create a list that never exceeds a given number of elements, but just remembers the latest N elements.
    ```
    LPUSH mylist a   # now the list is "a"
    LPUSH mylist b   # now the list is "b","a"
    RPUSH mylist c   # now the list is "b","a","c" (RPUSH was used this time)
    ```
  - Set (unordered, no duplicate)
    - add, remove, and test for existence of members in O(1)
  - Sorted_sets (ordered by a self-defined score)
  - Hashes
    ```
    HMSET user:1000 username antirez password P1pp0 age 34
    HGETALL user:1000
    HSET user:1000 password 12345
    HGETALL user:1000
    ```
- Measure memory `MEMORY USAGE`
  - Example 1: testing `hash`, name = 255 byte, key = 8 byte, value = 9 byte, 365 records
    - 1 record: 384 byte
    - 2 record: 408 byte
    - 364 record: 8552 byte
    - 365 record: 8552 byte (in theory in 6460 byte)
    - 365 record: 8552 byte (even the name have 265 byte)
  ```
  > SET "" ""
  OK
  > MEMORY USAGE ""
  (integer) 51
  ```

## Local Setup

```sh
sudo service redis-server start
```

# Elasticsearch (a search engine, not exactly NoSQL)
- `document` (row) > different `fields` (column)
  - basically a document is a `JSON` object
    ```json
    {  // this is a document
        "sku":"H0361001_S_1131001010073",
        "originalPrice":628,
        "originalPriceMTS":1589338195236
    }
    ```
  - queries sent to `Elasticsearch` is written in `NDJSON`, using `REST API`
    ```json
    POST _bulk
    { "field1" : "value1" }
    { "delete" : { "_index" : "test", "_id" : "2" } }
    { "create" : { "_index" : "test", "_id" : "3" } }
    ```

# Appendix
## Docker set up for mongodb replica set
Flow:
1. Set up 3 containers, each open different ports
2. Map the corresponding port back to localhost
3. Use this uri in spring boot application.yml: mongodb://hktv_pricechart_mongodb_1:27017,hktv_pricechart_mongodb_2:27018,hktv_pricechart_mongodb_3:27019/hktv_pricechart?replicaSet=hktv-pricechart
4. Modify the host setting, so that the server name map to the host name

docker-compose.yml:
```yml
services:
  mongodb_1:
    container_name: hktv_pricechart_mongodb_1
    image: mongo:5.0.9
    ports:
      - "27017:27017"
    networks:
      - hktv_pricechart
    command: mongod --replSet hktv-pricechart --port 27017
    volumes:
      - ./rs-init.sh:/scripts/rs-init.sh
  mongodb_2:
    container_name: hktv_pricechart_mongodb_2
    image: mongo:5.0.9
    ports:
      - "27018:27018"
    networks:
      - hktv_pricechart
    command: mongod --replSet hktv-pricechart --port 27018
  mongodb_3:
    container_name: hktv_pricechart_mongodb_3
    image: mongo:5.0.9
    ports:
      - "27019:27019"
    networks:
      - hktv_pricechart
    command: mongod --replSet hktv-pricechart --port 27019
networks:
  hktv_pricechart:
```
rs-inti.sh:
```sh
#!/bin/bash

mongosh <<EOF
var config = {
    "_id": "hktv-pricechart",
    "version": 1,
    "members": [
        {
            "_id": 1,
            "host": "hktv_pricechart_mongodb_1:27017",
            "priority": 2
        },
        {
            "_id": 2,
            "host": "hktv_pricechart_mongodb_2:27018",
            "priority": 1
        },
        {
            "_id": 3,
            "host": "hktv_pricechart_mongodb_3:27019",
            "priority": 1
        }
    ]
};
rs.initiate(config, { force: true });
EOF
```
start-db.bat:
```bat
call docker compose up -d
call docker exec hktv_pricechart_mongodb_1 /scripts/rs-init.sh
call docker exec hktv_pricechart_mongodb_1 mongosh --eval "rs.status()"
```

/etc/host or c:\Windows\System32\Drivers\etc\hosts
```
127.0.0.1 hktv_pricechart_mongodb_1
127.0.0.1 hktv_pricechart_mongodb_2
127.0.0.1 hktv_pricechart_mongodb_3
```