---
title: Learning DB optimizations
date: 2019-07-30 09:00:01 Z
categories:
  - python
  - django
  - postgresSQL
layout: post
summary: I haven't done any db optimizations in the past, here I describe the process and the decisions I made with one.
---

It all started with the following query:

```sql
SELECT * FROM "downloader_jobs"
WHERE (
  "downloader_jobs"."created_at" > '2017-02-05T00:00:00+00:00'::timestamptz AND
  "downloader_jobs"."no_retry" = false AND
  "downloader_jobs"."retried" = false AND
  "downloader_jobs"."success" = false
)
ORDER BY "downloader_jobs"."id" ASC
LIMIT 2001
```

We got it from [the logs](https://github.com/AlexsLemonade/refinebio/issues/1378#issuecomment-511816738) and it was taking more than 2 seconds. At the time of writing, there were `4,376,662` rows in the `downloader_jobs` table, which in the era of big data doesn't seem like much.

Taking a look at the query plan with [explain analyze](https://www.postgresql.org/docs/current/sql-explain.html) we get:

```
 Seq Scan on public.downloader_jobs  (cost=0.00..149446.10 rows=857608 width=177) (actual time=716.124..716.124 rows=0 loops=1)
   Output: id, downloader_task, accession_code, ...
   Filter: ((NOT downloader_jobs.success) AND (NOT downloader_jobs.no_retry) AND (NOT downloader_jobs.retried) AND (downloader_jobs.created_at > '2017-02-05 00:00:00'::timestamp without time zone))
   Rows Removed by Filter: 4376662
 Planning time: 0.082 ms
 Execution time: 716.151 ms
(6 rows)
```

That is a slightly modified query without the `order by` clause, but still starts to give some insights on the problem. A [sequential scan](https://devcenter.heroku.com/articles/expensive-queries#solutions-to-expensive-queries) on a large table is often a bad sign, since all of the rows are being loaded to filter out the ones that don't match the filter.

In Django, [creating indexes](https://docs.djangoproject.com/en/2.2/ref/models/options/#django.db.models.Options.indexes) is not complicated, the real question is what to index? It's been a while since I'm out of school and I haven't done much database stuff ever since, except for maybe discussions about the theory. I started reading this [sql indexing and tunning book](https://use-the-index-luke.com/) to gain a better understanding of the whole thing, and there were a lot of things that I learned.

Turns out that each index is a BTree (which I think I knew) that sorts whatever keys you specify into a tree structure. The main advantage is that filtering on the keys has logarithmic time. All of the rows are connected with a double linked list, and the BTree can be used to find a range of rows that might fulfil a given query. Databases have a query planner module that decides how to execute a given query. The basic gist related to indexes is that from the conditions of a query, the best ones can be used to search in the keys of the BTree because that's logarithmic time. The filters that can't be used like this have to be applied sequentially on ALL ROWS that the BTree founds and this is the big problem, because all these rows have to (potentially) be loaded in memory to apply the other operations.

There's still the question of what fields to select to add to the index, for this it's very important to know the data and how it's queried. A quick search for usages of the `downloader_jobs` table reveals that the following parameters are used together, each set used in a different query.

- `success`, `retried`, `no_retry`, `created_at`
- `success`, `retried`, `no_retry`, `created_at`, `start_time`, `end_time`
- `success`
- `id`

`id` is a primary key, and already has an index that is [added automatically](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-PRIMARY-KEYS) by postgresql when the table was created. For the others seems like an index with the fields `success`, `retried`, `no_retry`, `created_at` in that order would suffice.

However, upon further investigation, it's not a good idea to [create an index on a boolean column](https://stackoverflow.com/a/42972924/763705), because "it is cheaper to sequentially read the whole table than to use random I/O on the index and the table if a high percentage of the table has to be retrieved". So seems like the ideal solution could be a [partial index](https://use-the-index-luke.com/sql/where-clause/partial-and-filtered-indexes) on `created_at` with the conditions that are applied to `success`, `retried`, `no_retry`.

I was tempted to have three fields in the filter, `created_at`, `start_time` and `end_time` but `created_at` is always used as a range where we select all jobs that were created after a given date. This might have been a good idea if there were many rows that were created at the same time, and the queries didn't use a range but selected with an equality. The main issue with this index is that the sorting of rows by `start_time` after `created_at` is [irrelevant](https://use-the-index-luke.com/sql/where-clause/searching-for-ranges/greater-less-between-tuning-sql-access-filter-predicates) because it's very unlikely that two jobs will be created at the same time.

### Random key points I saw while reading

- It's important not to add too many indexes because it degrades the performance of `INSERT`, `UPDATE` and `DELETE` operations.

- Rule of thumb: Index for equality first—then for ranges.

- Leaf node transversal is the big problem

### Conclusion

Submitted everything in a [pull request](https://github.com/AlexsLemonade/refinebio/pull/1436) to [refine.bio](https://www.refine.bio), it's fairly simple after all the reading, but I think it was important to get a good background in order to be more confident on the decision.

I just discovered these tools, and it already feels like a super power. I'm gonna try to be more mindful when writing queries in the future, and try to think of how they might be executed in the DB.

<div style="width:100%;height:0;padding-bottom:100%;position:relative;"><iframe src="https://giphy.com/embed/mVQEwcRpxamnS" width="100%" height="100%" style="position:absolute" frameBorder="0" class="giphy-embed" allowFullScreen></iframe></div>
