---
tags: [coding, sql, nosql, databases]
type: cheatsheet
source: compiled reference (NoSQL data models)
last-verified: 2026-08-27
---

# NoSQL

## Up
- [[SQL]]

"NoSQL" ("Not only SQL") covers non-relational databases that trade the rigid table/join model for flexible schemas, horizontal scaling, and specialised access patterns. This note maps the four main families and their query styles.

---

## The Four Families

| Family | Data model | Examples | Best for |
|---|---|---|---|
| **Document** | JSON-like documents in collections | MongoDB, Couchbase, Firestore | Flexible/nested records, content, catalogs |
| **Key-Value** | Simple key → value | Redis, DynamoDB, Riak | Caching, sessions, high-throughput lookups |
| **Wide-Column** | Rows with dynamic column families | Cassandra, ScyllaDB, HBase, Bigtable | Time-series, write-heavy, huge scale |
| **Graph** | Nodes + edges + properties | Neo4j, Neptune, ArangoDB | Relationships, recommendations, fraud |

---

## SQL vs NoSQL Terminology

| SQL | Document (MongoDB) | Wide-Column |
|---|---|---|
| Database | Database | Keyspace |
| Table | Collection | Column family / table |
| Row | Document | Row |
| Column | Field | Column |
| Primary key | `_id` | Partition + clustering key |
| JOIN | `$lookup` / embed | (usually denormalised) |

---

## Document — MongoDB (query API)

```javascript
// CRUD
db.users.insertOne({ name: "Ada", age: 30, tags: ["admin"] })
db.users.insertMany([{...}, {...}])

db.users.find({ age: { $gt: 25 } })
db.users.find({ name: "Ada" }, { name: 1, _id: 0 })     // projection
db.users.findOne({ _id: ObjectId("...") })

db.users.updateOne({ name: "Ada" }, { $set: { age: 31 } })
db.users.updateMany({}, { $inc: { age: 1 } })
db.users.deleteOne({ name: "Ada" })

// operators
{ age: { $gte: 18, $lte: 65 } }        // $gt $gte $lt $lte $ne $in $nin
{ $or: [{ a: 1 }, { b: 2 }] }          // $and $or $not $nor
{ tags: { $in: ["admin"] } }
{ "address.city": "NYC" }              // nested field
{ tags: { $exists: true } }

// aggregation pipeline (like GROUP BY / joins)
db.orders.aggregate([
  { $match: { total: { $gt: 100 } } },
  { $group: { _id: "$userId", total: { $sum: "$total" }, n: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 10 },
  { $lookup: { from: "users", localField: "_id", foreignField: "_id", as: "user" } }
])

db.users.createIndex({ email: 1 }, { unique: true })
```

---

## Key-Value — Redis (commands)

```bash
SET user:1 "Ada"           # string
GET user:1
SETEX session:x 3600 "..." # with TTL (seconds)
EXPIRE key 60; TTL key; DEL key; EXISTS key

INCR counter; DECR counter; INCRBY views 10

HSET user:1 name Ada age 30    # hash (object)
HGET user:1 name; HGETALL user:1

LPUSH queue a b; RPOP queue    # list (queue/stack)
SADD tags admin user; SMEMBERS tags    # set
ZADD board 100 ada 90 bob; ZRANGE board 0 -1 WITHSCORES   # sorted set (leaderboard)
```

Redis is in-memory (with persistence options) — used as cache, session store, queue, rate limiter, leaderboard, and pub/sub bus.

---

## Wide-Column — Cassandra (CQL)

CQL looks like SQL but the data model is fundamentally different — you **model tables around queries**, denormalise, and choose partition keys carefully.

```sql
CREATE KEYSPACE shop WITH replication =
  {'class': 'SimpleStrategy', 'replication_factor': 3};

CREATE TABLE orders_by_user (
    user_id UUID,
    order_id TIMEUUID,
    total DECIMAL,
    PRIMARY KEY (user_id, order_id)      -- (partition key, clustering key)
) WITH CLUSTERING ORDER BY (order_id DESC);

INSERT INTO orders_by_user (user_id, order_id, total)
VALUES (uuid(), now(), 99.50);

SELECT * FROM orders_by_user WHERE user_id = ?;   -- must include partition key
-- no arbitrary JOINs, no ad-hoc WHERE on non-key columns (without ALLOW FILTERING)
```

---

## Graph — Cypher (Neo4j)

```cypher
CREATE (a:Person {name: 'Ada'})-[:FRIEND]->(b:Person {name: 'Bob'})

MATCH (p:Person {name: 'Ada'})-[:FRIEND]->(friend)
RETURN friend.name

MATCH (p:Person)-[:FRIEND*1..3]->(fof)     // 1–3 hops (friends of friends)
WHERE p.name = 'Ada'
RETURN DISTINCT fof.name

MATCH (a)-[r:RATED]->(m:Movie) WHERE r.score > 4 RETURN m
```

---

## Consistency & CAP

- **CAP theorem**: under a network partition you choose **Consistency** or **Availability**. Many NoSQL stores favour AP (eventual consistency) for scale.
- **BASE** (Basically Available, Soft state, Eventual consistency) vs relational **ACID**.
- Tunable consistency: DynamoDB (eventual vs strongly consistent reads), Cassandra (`ONE`/`QUORUM`/`ALL`).

---

## When to Choose NoSQL

- Schema evolves rapidly or varies per record → **document**.
- Simple, blazing-fast lookups / caching / ephemeral state → **key-value**.
- Massive write throughput, time-series, known query patterns → **wide-column**.
- Data is fundamentally about relationships/traversal → **graph**.
- Need multi-row ACID transactions, complex ad-hoc joins, strong consistency → a relational engine ([[PostgreSQL]], [[MySQL]], [[MSSQL]]) is often still the better fit.
