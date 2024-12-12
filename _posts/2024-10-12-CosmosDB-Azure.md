---
layout: post
title:  "Creating a New Collection with TTL in Cosmos MongoDB"
date:   2024-12-12 14:00:00 -0400
categories: azure cosmosdb mongodb
tags: azure cosmosdb mongodb
---

When working on my dashboard application, I needed to set up a new collection in Cosmos DB using the MongoDB API. One important feature I wanted was a **Time to Live (TTL)** policy for documents, ensuring automatic expiration after 35 days.

I was looking into just writing a script, but this seemed like the best way forward.  The TTL does not affect existing documents, so you either have to do an update to all documents and remove and then the TTL will be available.

### Steps to Implement TTL:

1. **Calculate TTL in Seconds**  
   TTL is specified in seconds. For 35 days, the value is:
   ```plaintext
   35 days × 24 hours/day × 60 minutes/hour × 60 seconds/minute = 3024000 seconds
Using the Mongo Shell
I accessed the Mongo shell in the Azure portal and ran the following command to create an index with the TTL policy:

```javascript
db.coll.createIndex({"_ts": 1}, {expireAfterSeconds: 3024000})
```
Executing the command returned the following response, confirming the index creation:
```json
{
    "_t" : "CreateIndexesResponse",
    "ok" : 1,
    "createdCollectionAutomatically" : true,
    "numIndexesBefore" : 1,
    "numIndexesAfter" : 4
}
```
Key Points:
The _ts field in Cosmos DB automatically tracks the timestamp of document creation or updates.
The expireAfterSeconds option ensures documents older than the specified duration are automatically removed.
This setup simplifies managing the lifecycle of documents in the collection and keeps the database clean and performant.