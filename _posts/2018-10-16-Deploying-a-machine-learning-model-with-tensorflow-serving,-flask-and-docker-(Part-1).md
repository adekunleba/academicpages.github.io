Having worked with Machine Learning model for quite sometimes, the basic challenge has been deployment of the model in production. With this in mind google created Tensorflow Serving which is supposed to be an ideal environment for running models in production.

Tensorflow serving main points include the ability to build a serverable which is the fundamental abstraction of Tensorflow serving. Servables are built using `SavedModelBundle` in Tensorflow. Servables are basically there to add flexibility to serving model in production, you can basically serve multiple models to multiple products at the same time with an instance of Tensorflow Serving, you can also use this to do some form of A/B Testing.

Tensorflow serving uses a gRPC protocol to connect to the server from your client application, although now they have the REST API version. gRPC is a service approach that makes use of Protocol Buffers which is a powerful binary serialization toolset. The claim for using gRPC for Tensorflow serving is that it is fast, a recent article made a comparison between using gRPC and the REST API version, you can check it [here](https://medium.com/@avidaneran/tensorflow-serving-rest-vs-grpc-e8cef9d4ff62).

Usually you will need a client to convert your data to a Protocol buffer so that it can be sent to the Tensorflow server. In this article I will be writing about how I used Flask-restPlus to build a POC to connect to Tensorflow serving via gRPC. In a latter article I will write how we migrated this to scala for production.

We are going to use a pretrained model i.e DeepLabv3 that google produced to build a servable and build a flask-restplus around it.

a. Converting DeepLabV3 to a Servable

Here is a simple script that prepares a frozen Tensorflow model for Tensorflow Serving model.

<script src="https://gist.github.com/adekunleba/a147d1df892014f37624d5e4c699556f.js">
</script>

We can then serve this model using docker by running the following simple command:

<script src="https://gist.github.com/adekunleba/3b1946ba8843ac9a02d58bf77d522ed3.js">
</script>

By replacing `path/to/model` with the folder containing your saved models generated from the builder, and the model name with the name of your latest model, you can run the above docker script and it will pull a tensorflow server docker and exposes the gRPC port which is 8501, which your application can connect to for serving your model.

In part II, I will show how flask helps to convert your incoming data request to a gRPC format for your model to be able to predict on.

_I hope this article benefits someone trying to take models to production in their applications with Tensorflow Serving_


.