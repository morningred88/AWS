## Amazon Elasticsearch (Amazon ES)

Elasticsearch is a distributed search and analytics engine built on Apache Lucene

## How does Elasticsearch work?

You can send data in the form of JSON documents to Elasticsearch using the API or ingestion tools such as [Logstash](https://aws.amazon.com/opensearch-service/the-elk-stack/logstash/) and [Amazon Kinesis Firehose.](https://aws.amazon.com/kinesis/data-firehose/) Elasticsearch automatically stores the original document and adds a searchable reference to the document in the cluster’s index. You can then search and retrieve the document using the Elasticsearch API. You can also use [Kibana](https://aws.amazon.com/opensearch-service/the-elk-stack/kibana/), a visualization tool, with Elasticsearch to visualize your data and build interactive dashboards.

**Reference:**

https://aws.amazon.com/opensearch-service/the-elk-stack/what-is-elasticsearch/

## Kibana

Many enterprise customers who want to use these capabilities find it challenging to secure access to Kibana. Kibana users have direct access to data stored in Amazon ES—so it’s important that only authorized users have access to Kibana.

**Reference:**

https://aws.amazon.com/blogs/security/how-to-enable-secure-access-to-kibana-using-aws-single-sign-on/
