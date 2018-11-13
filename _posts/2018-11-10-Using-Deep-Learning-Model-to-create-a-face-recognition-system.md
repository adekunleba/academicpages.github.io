I recently had to work on a project to build a face-recognition engine that will be used in production. Here I am going to describe on an high level things that were done.

## What is a Face Recognition system
A face recognition system is a system that has the ability to use a person's facial properties for identification, verication or recognition. Early facial recognition systems (FCS) makes use of Principal Component Analysis in generating face features. Using this method, the features generated were termed Eigenfaces. Eigen faces are more of a lower dimensional representations of a face image i.e Consider a cropped face image, you then make use of Principal component analysis to make a lower dimension representation of the pixel values in the face image.

## Deep Learning for better Face Features.

With the Advent of Deep Learning model, feature generation from faces are now done in a much effective and accurate way. The state of the art face recognition technologies now employ the use of Deep Neural networks as observed from Labeled Faces in the Wild which is one of the benchmarks that is use in comparing the effectivenes of Face Recognition systems. Currently the leading models are all Deep Learning models, Facebook's DeepFace has an accuracy of 0.9735, Google's FaceNet has an accuracy of 0.9963 compared to the original EigenFaces which has an accuracy of 0.6002.

Thus to build a production ready face recognition system, there are some basics components that your application should have.

1. Face Detection and Alignment system
2. A face feature generating model
3. Verification/Identification/Recognition layer.

All these three components must be coupled together to have a functional state of the art Face Recognition system.

## Components of a face Recognition system
### a. Face Detection and Alignment Component:
For most face recognition system, it's important to extract the face portion in images before passing it to your model. In DeepFace paper, the first line in the abstract writes thus:

>In modern face recognition, the conventional pipeline consists of four stages: detect ⇒ align ⇒ represent ⇒ classify

There are many technologies used in face detection and alignment.
For Detection we can explore the use of Weak classifier cascades . Open CV have couple of Haar features that worked well for our use case.
Also there is the Multi-task Cascaded Convolutional Networks which can both do face detection and alignment. 
Here is a link to learn about both [Haar Features](https://facedetection.com/algorithms/) and [MTCNN](https://kpzhang93.github.io/MTCNN_face_detection_alignment/index.html)

An ongoing project from my end is to write MTCNN in Scala, but currently my project made use of the Haars Cascade Classifiers.

### b. A Face Feature generating model
As recently mentioned above, the most accurate face feature generating model for a face recognition system is a Deep Learning model. Also, this is also a vital part that determines a lot in the whole system. Many models are available and have been open sourced. Facebook's DeepFace and Google's Facenet are very prominent open sourced face feature generating model.
For Facebook's Deep Face it contains two Convolutions with a Max poolings in-between them, Local Convolutions and Fully connected network. Local Convolutions basically use a different set of learned weights at every pixel, this is compared to a Normal Convolution which uses same set of weights at all locations. In simple explanation, you basically just use a giant filter and neglect doing much convolution. [Here]https://prateekvjoshi.com/2016/04/12/understanding-locally-connected-layers-in-convolutional-neural-networks/) you can read more about Local Convolution.


### c. A Final Metric learning layer for Verification/Identification/Recognition:
A face passed through a signature generating model generates a D-Dimensional feature vector which is representative of a person's face, once the model generates the face signatures, a metric learning algorithm or some other distance calculating algorithms compares the generated features for closeness in distance.
Some metrics used for this includes:

* Cosine Metrics
* Siamese Networks

## Conclusion:
For deployment of this composed method, you definitely need to implement this components facing a database, Serve your Tensorflow model from a point, and wrap the whole project around a backend service, for my case I used Akka-http to build the backend and this forced the project towards the use of JVM for tensorflow model serving.

*I hopes this article benefits someone who is willing to build a simple face recognition engine for themselves.*