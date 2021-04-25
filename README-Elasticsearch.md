# [Elasticsearch tutorial](https://logz.io/blog/elasticsearch-tutorial/)

## Run Elasticsearch in Docker

https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#_pulling_the_image

### Pulling the image

Obtaining Elasticsearch for Docker is as simple as issuing a `docker pull` command against the
Elastic Docker registry.

```sh
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.12.0
```

### Starting a single node cluster with Docker

To start a single-node Elasticsearch cluster for development or testing, specify
[single-node discovery](https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html#single-node-discovery "Single-node discovery")
to bypass the
[bootstrap checks](https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html "Bootstrap Checks"):

```sh
docker run -p 9200:9200  -p 9300:9300  -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.12.0
```

## Running Elasticsearch

To confirm that everything is working fine, simply point curl or your browser to
`http://localhost:9200`, and you should see something like the following output:

```json
{
  "name": "33QdmXw",
  "cluster_name": "elasticsearch",
  "cluster_uuid": "mTkBe_AlSZGbX-vDIe_vZQ",
  "version": {
    "number": "6.1.2",
    "build_hash": "5b1fea5",
    "build_date": "2018-01-10T02:35:59.208Z",
    "build_snapshot": false,
    "lucene_version": "7.1.0",
    "minimum_wire_compatibility_version": "5.6.0",
    "minimum_index_compatibility_version": "5.0.0"
  },
  "tagline": "You Know, for Search"
}
```

To debug the process of running Elasticsearch, use the Elasticsearch log files located (on Deb) in
`/var/log/elasticsearch/`.

## Creating an Elasticsearch Index

Indexing is the process of adding data to Elasticsearch. This is because when you feed data into
Elasticsearch, the data is placed into [Apache Lucene](https://lucene.apache.org/core/) indexes.
This makes sense because Elasticsearch uses the Lucene indexes to store and retrieve its data.
Although you do not need to know a lot about Lucene, it does help to know how it works when you
start getting serious with Elasticsearch. Elasticsearch behaves like a REST API, so you can use
either the `POST` or the `PUT` method to add data to it. You use `PUT` when you know the or want to
specify the `id` of the data item, or `POST` if you want Elasticsearch to generate an `id` for the
data item:

```sh
curl -XPOST 'localhost:9200/logs/my_app' -H 'Content-Type: application/json' -d'
{
	"timestamp": "2018-01-24 12:34:56",
	"message": "User logged in",
	"user_id": 4,
	"admin": false
}
'
curl -X PUT 'localhost:9200/app/users/4' -H 'Content-Type: application/json' -d '
{
  "id": 4,
  "username": "john",
  "last_login": "2018-01-25 12:34:56"
}
'
```

And the response:

```json
{"_index":"logs","_type":"my_app","_id":"ZsWdJ2EBir6MIbMWSMyF","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}
{"_index":"app","_type":"users","_id":"4","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}
```

The data for the document is sent as a JSON object. You might be wondering how we can index data
without defining the structure of the data. Well, with Elasticsearch, like with any other NoSQL
database, there is no need to define the structure of the data beforehand. To ensure optimal
performance, though, you can define Elasticsearch mappings according to data types. More on this
later.

If you are using any of the Beats shippers (e.g. Filebeat or Metricbeat), or Logstash, those parts
of the ELK Stack will automatically create the indices.

To see a list of your Elasticsearch indices, use:

```sh
curl -XGET 'localhost:9200/_cat/indices?v&pretty'
health status index               uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   logstash-2018.01.23 y_-PguqyQ02qOqKiO6mkfA   5   1      17279            0      9.9mb          9.9mb
yellow open   app                 GhzBirb-TKSUFLCZTCy-xg   5   1          1            0      5.2kb          5.2kb
yellow open   .kibana             Vne6TTWgTVeAHCSgSboa7Q   1   1          2            0      8.8kb          8.8kb
yellow open   logs                T9E6EdbMSxa8S_B7SDabTA   5   1          1            0      5.7kb          5.7kb
```

The list in this case includes the indices we created above, a Kibana index and an index created by
a Logstash pipeline.

## Elasticsearch Querying

Once you index your data into Elasticsearch, you can start searching and analyzing it. The simplest
query you can do is to fetch a single item. Read our article focused exclusively on
[Elasticsearch queries](https://logz.io/blog/elasticsearch-queries/).

Once again, via the Elasticsearch REST API, we use `GET`:

```sh
curl -XGET 'localhost:9200/app/users/4?pretty'
```

And the response:

```json
{
  "_index": "app",
  "_type": "users",
  "_id": "4",
  "_version": 1,
  "found": true,
  "_source": {
    "id": 4,
    "username": "john",
    "last_login": "2018-01-25 12:34:56"
  }
}
```

The fields starting with an underscore are all meta fields of the result. The `_source` object is
the original document that was indexed. We also use GET to do searches by calling the `_search`
endpoint:

```sh
curl -XGET 'localhost:9200/_search?q=logged'
{"took":173,"timed_out":false,"_shards":{"total":16,"successful":16,"skipped":0,"failed":0},"hits":{"total":1,"max_score":0.2876821,"hits":[{"_index":"logs","_type":"my_app","_id":"ZsWdJ2EBir6MIbMWSMyF","_score":0.2876821,"_source":
{
    "timestamp": "2018-01-24 12:34:56",
    "message": "User logged in",
    "user_id": 4,
    "admin": false
}
}]}}
```

The result contains a number of extra fields that describe both the search and the result. Here’s a
quick rundown:

- `took`: The time in milliseconds the search took
- `timed_out: If the search timed out`
- `_shards`: The number of Lucene shards searched, and their success and failure rates
- `hits`: The actual results, along with meta information for the results

The search we did above is known as a
[URI Search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-uri-request.html),
and is the simplest way to query Elasticsearch. By providing only a word, ES will search all of the
fields of all the documents for that word. You can build more specific searches by using
[Lucene queries](https://lucene.apache.org/core/5_3_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html#package_description):

- **username:johnb** – Looks for documents where the username field is equal to “johnb”
- **john\*** – Looks for documents that contain terms that start with john and is followed by zero
  or more characters such as “john,” “johnb,” and “johnson”
- **john?** – Looks for documents that contain terms that start with john followed by only one
  character. Matches “johnb” and “johns” but not “john.”

There are many other ways to search including the use of boolean logic, the boosting of terms, the
use of fuzzy and proximity searches, and the use of regular expressions.

## Elasticsearch Query DSL

URI searches are just the beginning. Elasticsearch also provides a
[request body search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html)
with a [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
for more advanced searches. There is a wide array of options available in these kinds of searches,
and you can mix and match different options to get the results that you require.

It contains two kinds of clauses: 1) leaf query clauses that look for a value in a specific field,
and 2) compound query clauses (which might contain one or several leaf query clauses).

### Elasticsearch Query Types

There is a wide array of options available in these kinds of searches, and you can mix and match
different options to get the results that you require. Query types include:

1.  Geo queries,
2.  “More like this” queries
3.  Scripted queries
4.  Full text queries
5.  Shape queries
6.  Span queries
7.  Term-level queries
8.  Specialized queries

More on the subject:

- [How To's: Log Patterns](https://logz.io/learn/video/how-tos-log-patterns/ "How To's: Log Patterns")
- [Network Security Monitoring with Suricata, Logz.io and the ELK Stack](https://logz.io/blog/network-security-monitoring/ "Network Security Monitoring with Suricata, Logz.io and the ELK Stack")
- [How to Monitor Cloud Migration and Data Transfer](https://logz.io/blog/data-monitor-cloud-migration/ "How to Monitor Cloud Migration and Data Transfer")

As of Elasticsearch 6.8, the ELK Stack has merged Elasticsearch queries and Elasticsearch filters,
but ES still differentiates them by context. The DSL distinguishes between a _filter context_ and a
_query context_ for query clauses. Clauses in a filter context test documents in a boolean fashion:
Does the document match the filter, “yes” or “no?” Filters are also generally faster than queries,
but queries can also calculate a _relevance score_ according to how closely a document matches the
query. Filters do not use a relevance score. This determines the ordering and inclusion of
documents:

```sh
curl -XGET 'localhost:9200/logs/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_phrase": {
      "message": "User logged in"
    }
  }
}
'
```

And the result:

```json
{
  "took" : 28,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.8630463,
    "hits" : [
      {
        "_index" : "logs",
        "_type" : "my_app",
        "_id" : "ZsWdJ2EBir6MIbMWSMyF",
        "_score" : 0.8630463,
        "_source" : {
          "timestamp" : "2018-01-24 12:34:56",
          "message" : "User logged in",
          "user_id" : 4,
          "admin" : false
        }
      }
    ]
  }
}
'
```

## Removing Elasticsearch Data

Deleting items from Elasticsearch is just as easy as entering data into Elasticsearch. The HTTP
method to use this time is—surprise, surprise—`DELETE`:

```sh
$ curl -XDELETE 'localhost:9200/app/users/4?pretty'
{
  "_index" : "app",
  "_type" : "users",
  "_id" : "4",
  "_version" : 2,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

To delete an index, use:

```sh
$ curl -XDELETE 'localhost:9200/logs?pretty'
```

To delete _all_ indices (use with extreme caution) use:

```sh
$ curl -XDELETE 'localhost:9200/_all?pretty'$
```

The response in both cases should be:

```json
{
  "acknowledged": true
}
```

To delete a single document:

```sh
$ curl -XDELETE 'localhost:9200/index/type/document'
```
