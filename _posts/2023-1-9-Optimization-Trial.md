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
