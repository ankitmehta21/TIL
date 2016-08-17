I had a spark job which kept getting stuck very early in it's execution, at around 47/3753 tasks.
It only happened periodically, but it was very frustrating when it did happen.

The actual code is [here](), but here's the relevant part which would get stalled:
```
case class longitudinal (
    client_id: String
  , geo_country: Option[Seq[String]]
  , session_length: Option[Seq[Long]]
)

case class crossSectional (
    client_id: String
  , modal_country: Option[String]
)

val ds = hiveContext.sql("SELECT * FROM longitudinal")
  .as[longitudinal]
val output = ds.map(xx => "Test")
output.count
```

Even weirder, the following code still created >3000 tasks, even though we limited to one row of data:
```
val ds = hiveContext.sql("SELECT * FROM longitudinal")
  .limit(1)
  .as[longitudinal]
val output = ds.map(xx => "Test")
output.count
```

In the end, it looks like the conversion from the dataframe to a dataset[longitudinal] was reading the whole dataset even though we only need a couple of columns.
We fixed the problem by limiting the columns manually:

```
val ds = hiveContext.sql("SELECT * FROM longitudinal")
  .selectExpr("client_id", "geo_country", "session_length")
  .as[longitudinal]
val output = ds.map(xx => "Test")
output.count
```

We had no problems with this locally, which is strange. 
