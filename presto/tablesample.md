# TABLESAMPLE in Presto

Presto provides three major ways to downsample your data, the classic LIMIT statement and two varieties of TABLESAMPLE.
LIMIT is useful for truncating data and debugging queries, but it's generally not guaranteed to be a statistically sound sample of your data.
This is where TABLESAMPLE is useful.

TABLESAMPLE BERNOULLI is the most understandable method for a statistician and is the most likely to produce a valid sample.
For every row, flip a biased coin, if it's heads include the row, otherwise discard.
However, this requires Presto to read your entire DB, which can be a significant time cost.

TABLESAMPLE SYSTEM is usually more efficient and can be just as accurate in certain situations.
Instead of choosing each row individually, this method groups rows into chunks and decides whether to include at the chunk level.
This saves Presto from having to read every line of your DB.
However, the independence of the SYSTEM sample depends on how you're storing your data, so this can get interesting quickly. 
I need to do more research to figure out how this is done, but my intuition is that every shard is either included or not included at some confidence.
If your shards are huge or don't contain a random subset of data, then you're out of luck.

Note that LIMIT is the only solution that guarantees a sample of a given size.
TABLESAMPLE is not deterministic.
