---
layout: post
title: Optimization Trial On MongoDB Search with Atlas Search
---

# Optimization Trial On MongoDB Search with Atlas Search


## Background

Search is a very broad business scenario, especially text search.  The normal solution to improve the MongoDB search efficiency is to create the index. And if you want to improve more, you just add the compound index.

However, is there any good way to optimize more which can fit into real practice?

## Conclusion

**The mongoDB origin search with LIMIT and SORT wins in both resources and speed.**

why BO search query has a better behavior?

1. as far as I known, there is no compound index like business_title on Atlas Search ( have confirmed by Mongodb Support team), so it is the main cause why the BO search is better

2. the origin mongodb search probably has optimized the search with **SORT and LIMIT **. After comparing with the documents examined and documents returned, we could find when sort and limit is both applied in a search, the number of examined documents is much less than the total documents.

   Following is our guess based on this performance:  The search engine sort the document first, and if there is a limit to returned 20 results, once it find the 20th result can match all condition, it will return 20 results. So in most cases, the search engine will not examine the whole ducuments, which can be much more efficient.

## Atlas Search Experimentation Behavior

Following is the Atlas Search Engine I have tried, seems **lucene.standard** is still the best in my case:

lowercaser analyzers ->  executionTimeMillis around 181  

EdgeGramAnalyzer ->  executionTimeMillis around  82

lucene.standard -> executionTimeMillis around  12  (stable seems the quickest over all)

| Analyzer                 | executionTimeMillis |
| ------------------------ | ------------------- |
| Lowercaser Analyzer      | 181                 |
| EdgeGram Analyzer        | 82                  |
| lucene.standard Analyzer | 12                  |



The search Index I First created in Mongodb

```bash
{
  "analyzer": "lucene.standard",
  "searchAnalyzer": "lucene.standard",
  "mappings": {
    "dynamic": false,
    "fields": {
      "_id": [
        {
          "dynamic": true,
          "type": "document"
        },
        {
          "type": "objectId"
        }
      ],
      "barcode": [
        {
          "dynamic": true,
          "type": "document"
        },
        {
          "analyzer": "lucene.standard",
          "searchAnalyzer": "lucene.standard",
          "type": "string"
        }
      ],
      "business": [
        {
          "dynamic": true,
          "type": "document"
        },
        {
          "analyzer": "lucene.keyword",
          "searchAnalyzer": "lucene.keyword",
          "type": "string"
        }
      ],
      "isDeleted": [
        {
          "dynamic": true,
          "type": "document"
        },
        {
          "type": "boolean"
        }
      ],
      "parentProductId": [
        {
          "dynamic": true,
          "type": "document"
        },
        {
          "analyzer": "lucene.keyword",
          "searchAnalyzer": "lucene.keyword",
          "type": "string"
        }
      ],
      "skuNumber": [
        {
          "dynamic": true,
          "type": "document"
        },
        {
          "analyzer": "lucene.standard",
          "searchAnalyzer": "lucene.standard",
          "type": "string"
        }
      ],
      "tags": [
        {
          "dynamic": true,
          "type": "document"
        },
        {
          "analyzer": "lucene.keyword",
          "searchAnalyzer": "lucene.keyword",
          "type": "string"
        }
      ],
      "title": [
        {
          "dynamic": true,
          "type": "document"
        },
        {
          "analyzer": "lucene.standard",
          "searchAnalyzer": "lucene.standard",
          "type": "string"
        }
      ]
    }
  },
  "storedSource": true
}
```



### Something Interesting I find in the search Analyzer when testing analyzer

 Using the following index, the `text` operator query for `mi` returns results, but the search for `m` returns no results, because `m` is `1` character, and is less than the `minShingleSize` of `2`:

```
  "analyzers": [
    {
      "charFilters": [],
      "name": "SSAnalyzer",
      "tokenFilters": [
        {
          "type": "icuFolding"
        },
        {
          "maxGram": 10,
          "minGram": 1,
          "termNotInBounds": "include",
          "type": "edgeGram"
        },
        {
          "maxShingleSize": 3,
          "minShingleSize": 2,
          "type": "shingle"
        }
      ],
      "tokenizer": {
        "type": "standard"
      }
    }
  ],
  "storedSource": true
}
```

 Reference stated in [MongoDB doc](https://www.mongodb.com/docs/atlas/atlas-search/analyzers/token-filters/#shingle) 


