---
layout: default
---

Scalding 102

# [](#header-1)Should I read this?

If you don't know what scalding is and how to code a hello world project in it, I don't think you'll find this post useful.

# [](#header-1)What is covered?

This post goes over some challenges you may face as a scalding developer beyond the elementary and generic documentation you may find online. It covers neither elementary nor advanced material, just some practical tips to help you get through your first couple of weeks of coding in scalding and save you some time.

# [](#header-1)What is your preferred way of coding in scalding?

Just my preference. I prefer to build my code using [Apache Maven](https://maven.apache.org/). In terms of the IDE, if I'm working on a project with more than one class, I use [ScalaIDE](http://scala-ide.org/) otherwise I simply use [VIM](http://www.vim.org/). I don't use the IDE for building the code and I only use it for autocompletion  and project browsing. You may need to install some plugins for autocompletion though.

When I code I usually have two pages opened in my browser and 80% of the times they can provide me with what I'm looking for:

1. [scalding scaladoc page](http://twitter.github.io/scalding/api/index.html): com.twitter.scalding.typed.TypedPipe could be a good starting point.

2. [Fields based API Reference](https://github.com/twitter/scalding/wiki/Fields-based-API-Reference): This page contains answers to a lot of your questions. You'll find it very useful.

# [](#header-1)Okay, let's get started

## [](#header-2)I don't know how to filter records based on multiple fields, should I be ashamed?

Absolutely not. It's very straightforward though:

```scala
//assume you have two fields, one String and one Integer and you want to keep
//the records whose (lower cased) field1 is equal to "something" and field2
//module 10  is equal to 9
pipe.filter('field1, 'field2){r: (String, Int) => {
  val (field1, field2) = r
  (field1.toLowerCase().equals("something") && field2 % 10 == 9 )}
}
```

## [](#header-2)You say it's easy but what if I have to write a filter based on more than 22 fields?

Don't worry, I answer this for the "filter" case but my answer applies to any other operation in scalding as well.

You're gonna need to import cascading.tuple.Fields and then use a piece of code like the one below:

```scala
//code is not compileable. "..." means a lot of fields in between.
pipe.filter(new Fields("field1", "field1", ..., "field100"))
{r: Tuple => {
  (r.getString(0).toLowerCase().equals("something") && r.getInteger(1) % 10 == 9
  && a lot of other conditions)}
}
```

The limitation is inherited from Scala's (I guess 2.10 and earlier) case classes that could not exceed 22 parameters.

## [](#header-2)What is this _Pivot_ thing? Does it ever come handy? Show us an example.

It's basically a way to transform rows to columns in your pipe, and yes, it is very likely that at some point you need pivot or a similar logic.

Imagine your data is organized like this:

> **name, fruit_name, count**
>
> Tom, apple, 10
>
> Tom, orange, 5
>
> Joe, pear, 15

and you want to transform it to:

> **name, apple_count, orange_count, pear_count**
>
> Tom, 10, 5, 0
>
> Joe, 0, 0, 15

```scala
//0 is the default value here, meaning that if there is no matching row, a 0 will be put
//in the corresponding column.
pipe.groupBy('name)
{_.pivot(('fruit_name,'count) -> ('apple_count, 'orange_count, 'pear_count), 0)}
```

## [](#header-2)Okay, what if the field name contains space or something?

map is your friend (not sure what you mean by "something" though). You can even implement pivot with map if you feel like doing it.
There could be other answers to this question. I believe you can also use Fields for this purpose (e.g. new Fields("name with space")) but here is my answer using map:

```scala
pipe.map('fruit_name -> 'fruit_name){r:String => r.replaceAll(" ", "")}
.groupBy('name)
{_.pivot(('fruit_name,'count) -> ('namewithspace, some other fields), 0)}
```

## [](#header-2)How do I group by and create multiple aggregations? Should I be embarrassed to ask?

Use "dot".

```scala
pipe.groupBy(('field1, 'field2))
{_.sum[Double]('some_numeric_field -> 'you_can_name_the_aggregation_differently)
.average('another_field -> 'another_field)}
```

The final schema will be something like this:

field1, field2, you_can_name_the_aggregation_differently, another_field

And now that you know how to do it no reason to be embarrassed anymore.

## [](#header-2)How do I implement inequality join?

As far as I know there is no built-in mechanism for this. You can implement your inequality join with a normal join followed by a filter.
Since your joint initially can only happen on equality find a column to use in your join that is common across all records that you want to join. For example, in some scenarios "date" can be a good candidate for this purpose. If you don't have such field use map to artificially create it.

## [](#header-2)What happens when I groupBy and sum values that contain nulls?

Honestly I don't know. Run some tests and figure it out under your settings. I guess it depends on Scala's default behavior in handling nulls.

## [](#header-2)I want to write my pipe's content to multiple files by partitioning my data based on some criterion. Is it too much to ask for?

No it's not. Take a look at this:

```scala
//your data will be partitioned based on field_to_be_used_for_partitioning and
//for every value of that field a separate file will be created
//in /path/to/your/folder.
import com.twitter.scalding.{ PartitionedTsv => StandardPartitionedTsv, _ }
val DelimitedPartitionedTsv = StandardPartitionedTsv("/path/to/my/folder", "/", 'field_to_be_used_for_partitioning)
pipe.write(DelimitedPartitionedTsv)
```

## [](#header-2)I have a partitioned hive table and the partition field is not included in the hdfs file, how should I read the data?

I believe the support for reading partitioned files is added to the latest version of scalding (I was not able to get it working though). If you have an older version of scalding and you want scalding to pick up the partition field from the folder structure and add it as a field to your pipe, I guess your best bet is to use a var pipe, read from individual files and keep unioning your var pipe with itself. Of course this is not the most efficient way but it'll do the job.
