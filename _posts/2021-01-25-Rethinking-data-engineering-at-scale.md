I started 2021 with a focus on establishing myself in data engineering and building pipelines both for data science and machine learning operations. My last article was entirely me evaluating what goes on in the world of data engineering, the tools I have seen organiztion use and whether there is a case for build your own vs use an open source tool.

I recently had to do some data engineering task and as adviced in my earlier post that companies can initially start out with open source tools and if need or based on organizational requirements transition into building their in house data extraction tool.

After much research, I settled on two options (and since the focus is on open source), Apache Nifi and Apache Gobblin.

Apache Nifi is not deemed an ETL tool but rather a tool that is used for moving data at scale between systems. If we look at this from that point of view and what we do with data extraction, Apache Nifi looks like a valid tool for extracting data from various sources and moving it i.e the loading part of ETL. However, Apache Nifi is not so great at data transformation. Therefore I favour following the path of ELT when using Apache Nifi.

Apache Gobblin on the other hand is a new guy in the club of data extraction. It is a distributed data integration framework that simplifies common aspects of big data integration such as data ingestion, replication, organization and lifecycle management for both streaming and batch data ecosystems. 

The major difference between the two as at now is that Apache Nifi has been here for long approximately 10 years while Gobblin is still quite new. Furthermore, Nifi is entirely a GUI drag and drop approach to building the data integration system while Gobblin is not, this is not to say it is however much more flexible for developers compared to Apache Nifi.

In terms of beginner friendly, in as much as I haven't worked so much with Apache Gobblin, I found Apache Nifi slightly beginner friendly compared to Apache Gobblin and the fact that Apache Gobblin is only programmable with Java gives some limitation.

To add to this article is a big lesson learnt while working with Apache Nifi.

#### Apache Nifi Integration with Apache Kafka.

In the world of ELT, there is a very high chance that Kafka will be in the loop, either as a temporary data storage for replication or as a general cache engine/data distribution engine.

Since Apache Nifi is a GUI based drag and drop configuration approach to building a data extracton pipeline, the most important part of the project is usually the point at which your properly input the right configration while dragging and dropping your processors.

I will write an introductory article to Apache Nifi where I will detail the various components of the engine.

#### Specifics of Apache Nifi and Apache Kafka integration that worked.

My aim here is to document a scalable integration with Apache Nifi and Apache Kafka that worked:

1. The use of a schema registry which in my case `ConfluenceSchemaRegistry` made things quite smooth. I have initially tried to save the schema alongside the record but didn't quite worked. With this approach, we can basically Access Schema using the `Schema Name property`.

2. When consuming the Kafka Record, I favour the ConsumeKafka to ConsumeKafkaRecord. The major difference between the two being that ConsumeKafkaRecord allows you to read the message with a schema. The challenge is usually when you are not sure about the schema name and you have to supply it dynamically, hence you can just extract the record from Kafka using ConsumeKafka and then have the flexibility of processing it at your own dynamics.

3. Also, when consuming data, you want to set the Offset of your consumer to be the `earliest` it will save a lot of debugging as you would expect consumption to happen immediately you fire up (start) your consumer, but because your offset is set to default `latest` it only fires up on new message after connecting. This might not be very obvious to someone just starting out with Apache Nifi.

4. Additionally, you can TailFile a log file to use as an example data source - a very interesting trick gotten from this article [here](https://bryanbende.com/development/2017/06/20/apache-nifi-records-and-schema-registries)