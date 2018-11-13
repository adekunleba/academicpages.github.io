### Introduction
Apache Kafka messaging system with Apache Spark are two platforms that came with the Big Data Wave and they have been very useful in managing Big Data expecially when it comes to Streaming Data or *fast data*. 
I recently had to make use of this two framework for a project and I am going to explain briefly how I set up both projects locally and deploying to production.
The project makes use of two machine learning model, One is a Neural Network and the other is Clustering. I won't go into the details of the project as at yet, I will leave that for another post. But here is the summary of the project, my neural netowrk is supposed to generate signatures on some images and then my Clustering should then group them together into segments as they are been uploaded, such that at a particular time, I should be able to upload an image and predict the clusters which it will belong. I hope this summary gives an idea where these frameworks will be important.

### Why Spark and Kafka
As the summary of my project propose, I will need a continous update of my clustering model to take advantage of new image **enrollment** as I will like to think about it for when I am just introuducing the data to the clustering model, this is a perfect example for a Streaming Machine learning project.
Also, I will need to seperate the images that are intended to be used for training and those that are intended for prediction, I will need a framework to know which task my machine learning model should do, Is it updating the model or predicting on new Images, for this I saw a messaging system as a perfect option and the perfect one for this, probably my bias though, is Kafka.
Kafka is a publish and Subscribe framework that allows you to read and write data streams, process the streams and also store the data, my emphasis is on the read and write data streams for my project. Apache Spark Stremaing Api on the other hand is an extension of Spark API that enables scalable processing of live data streams that is ingested from different sources, it also affords you the opportunity to use complex algorithms on your flowing data, my emphasis for Apache Spark Streaming API is it's ability to process live data stream.


### Zookeeper manager for Kafka
Now that we have established the reason behind this frame work and how they relate to my project can we get started with setup:
Apache Kafka relies heavily on another Apache Framework called Zookeeper for coordinating it's activity, why is this important, because kafka is supposed to be a Fault tolerant and scalable engine i.e running on Distributed systems, it needs an external manager such that once any of it's component dies off, the managers know what to do for data loss to be prevented and also keep other running tasks, also, this explanation is as high-level as it can get.

### Installing Confluent
So to deploy kafka locally, we will employ using Confluent platform, which is a contained environment that has the various components that will effectively help run Kafka successfully, let's think of Confluent platform as a contained Kafka Virtual Environment.

And it's very easy to install and start, here is a simple guide to do just that. Run the following scripts from a unix terminal

<script src="https://gist.github.com/adekunleba/f0d6f8ed79fffd4db77be4404ba52396.js">
</script>


### Publishing Message
I will be using scala to publish my message into kafka.
To do this you will need to set up your scala application

Configuring your app to use the confluent on your localhost will be something like this:

<script 
src="https://gist.github.com/adekunleba/1697e1ad1db389e6d0318217f3866044.js">
</script>

With this configuration is easy to call method `send()` on `streams` to send message to a running kafka cluster.

<script src="https://gist.github.com/adekunleba/fcb0b7cc7d948fd19bceee1ab8a7f9ad.js">
</script>

Once the message is in the server, then our spark machine learning application can continously update as well as do a prediction on the testData as they are incoming. Something to keep in mind is that, for every restart of your model update by the consumer application, the model state is lost, basically I need to implement an approach to write the model to a file so as to save the update state, such that if I need to restart the consumer application I can easily read from a state and use that to update my model, this is reserved for a later article.

### Consuming messages from Kafka with Scala
First we need to configure our spark application to communicate with the Kakfa cluster.

Here is a sample configuration to manage how spark consumes data from the kafka cluster.

<script src="https://gist.github.com/adekunleba/5063b311aeb0209b0bbfa4057d99e2a5.js">
</script>

Now that the configuration for the consumer is done, then we need to implement how to do both model updates and prediction on the incoming messages.

<script 
src="https://gist.github.com/adekunleba/5058c0227f19f009046cf359ad79719b.js">
</script>

What I love regarding this two framework is their ease of implementation. One can scale this simple approach to work with a range of system.

I hope someone enjoys this and find this very useful.
