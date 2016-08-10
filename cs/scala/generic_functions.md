I haven't worked in typed languages for a while, but it's coming back quickly.
Since it took me a while to look up today, here's an example signature for a function with a type parameter.
This function takes two lists of category and weights, maps to (category, weight) tuples, combines like categories, and finds the modal category.

The category tuples can be anything that implements the comparable interface:

```scala
def weightedMode[T <: Comparable[T]](values: Seq[T], weights: Seq[Long]) = {
  val pairs = values zip weights
  val agg = pairs.groupBy(_._1).map(kv => (kv._1, kv._2.map(_._2).sum))
  agg.maxBy(_._2)._1
}
```
