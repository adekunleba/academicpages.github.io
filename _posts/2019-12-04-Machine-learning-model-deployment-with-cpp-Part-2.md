Picking up my initial article where I build a PCA model using CPP, In this article, I will be loading the saved model whose values are stored in a yaml file. This script will be developed to form an inference engine bundled as a `.so` file for deployment on a linux based environment. For other platforms, the cpp codes can be compiled to produce either a `dll` or `dylib` for windows and mac respectively.
Since we are more concerned about deploying the model on an android application, the focus will be building the `.so` file inference engine from our saved model.

## Building `.so` inference file.

### Load a PCA model
For inference, we let OpenCV load the existing model file from the saved `.yml` file after which we feed the eigenvalues and mean to a new `PCA` object which we can then call a `pca->project` on to create a new image's projection.

Here is a sample code that load a saved OpenCV FileStorage model.

```cpp
//Declare an empty model.
cv::PCA newPcaModel;

cv::PCA Facepca::loadmodel(cv::PCA newPcaModel, const std::string filename){

    cv::FileStorage fs(filename,cv::FileStorage::READ);
    fs["mean"] >> newPcaModel.mean ;
    fs["e_vectors"] >> newPcaModel.eigenvectors ;
    fs["e_values"] >> newPcaModel.eigenvalues ;
    fs.release();

    return newPcaModel;

}

loadmodel(newPcaModel, "path-to-saved-yml-file");
```

Once the model is loaded, the `newPcaModel` object now contains the saved model from the existing training. Hence when a face projection is done, it is guaranteed that the data returned is relative to the training dataset.

### Create new image preprocessing and prediction stage.
During inference of a machine learning model, it is important that the incoming image also passes through the same preprocessing as the training dataset.
Also, several approaches can be used to pass image to the inference engine, it could be that the image is loaded from disk or that the image is passed as a base64 string.
In our case, the likely approach is to use a base64 string since we are also taking into consideration two factors, that a `jni` will be exposed and also that our final application is for an android application.

With this in mind, we then need to ensure that are able to retrieve image from a base64 string and send it to OpenCV.

Decoding a base64 string in c++ is quite non-trivial, however reader can refer to [this link](https://renenyffenegger.ch/notes/development/Base64/Encoding-and-decoding-base-64-with-cpp) on a code snippet that does this.

Once the base64 image string is decoded, we then convert the string to a vector of unsigned character (uchar) which can be thought of as Image values.
OpenCV can decode the vector of uchar into an image using the function call to `cv::imdecode(vectorUchar, flag)`. This process returns a `Mat` image with which further preprocessing can be done.

The image can then pass through the phase of
- Face extraction
- Convert cropped faces to gray scale.
- Image resize
- Image normalization
- Create data matrix

Just as described in the first part of the article.
The last leg of inference on the new image is using the _loaded_ pca object to create a projection of the face in the new image.
The snippet that does that will look like this:
```cpp
newPcaModel->project(datamatrix.row(0));
```
The part of recognition or verication happens when you project  with a _loaded_ pca object on two faces and compare the projections (eigenfaces) using a distance metrics e.g euclidean distance or cosine similarity.


### Package the library with an exposed jni
This part is quite straightforward, once the inference code has been properly structured, most probably within a class, then the jni body can literally call the exposed functions for the model loading, preprocessing and prediction.
However, to create a jni, it is important to understand how the link occurs with java.
The first path is to create a Java class and the functions you want to use in your java class with the input parameters.
This will be what the jni will use as the function name when creating the cpp code. The consitency between the class and methods in Java with the name in cpp is very important.

Let us assume the method we will be using for our pca feature match is named `matchpcafeatures()` and our java class can then look like this.

```java

package org.example.code

class MatchFeatures {

    static float matchpcafeatures(modelfilename: String, image: String, projectionToCompare: Array[Float])
}

```

With the above java class and method, our jni header file will look something like this.

```cpp

#include <jni.h>
/* Header for class com_example_code_MatchFeatures */

#ifndef _Included_com_example_code_MatchFeatures
#define _Included_com_example_code_MatchFeatures
#ifdef __cplusplus
extern "C" {
#endif
JNIEXPORT jfloat JNICALL Java_com_example_code_MatchFeatures_matchpcafeatures
  (JNIEnv *, jobject, jstring, jstring, jfloatArray);

#ifdef __cplusplus
}
#endif
#endif

```
You don't need to bother yet about the details of the `extern C` the focus is on the name of the method in the jni header file.
Further down, we will look at how to use the java code above to make an inference. Let us first develop the jni bridge for this method.
The last 3 parameters of the jni header is just the same and in exact position as the parameters in the java method.

Therefore, we will process on thosee parameters as those are the key requirements from a client to our c++ inference engine.

Below shows how to connect those input parameters and return a value which the java code can take and continue its other processes.


```cpp

/**
 * Match features of pca
 * */
JNIEXPORT jfloat JNICALL Java_com_seamfix_qrcode_FaceFeatures_matchpcafeatures
  (JNIEnv * env, jobject obj, jstring pcafilename, jstring imagestring, jfloatArray projectionToCompare){

    const char *pcastring_char;
    pcastring_char = env->GetStringUTFChars(pcafilename, 0);

    if(pcastring_char == NULL) {
      return 0.0f;
    }


    const char *imagestring_char;
    imagestring_char = env->GetStringUTFChars(imagestring, 0);

    if(imagestring_char == NULL) {
      return 0.0f;
    }

    //Get file name string as a string for cpp
    std::string stdfilename(pcastring_char);

    cv::PCA pca;

    //Class InferencPca holds the preprocesing and inference methods
    InferencePca ef;
    std::vector<cv::Mat> imagevec;
    //Get image as base64
    cv::Mat image = ef.readBase64Image(imagestring_char);


    ef.loadmodel(pca, stdfilename);
    imagevec.push_back(image);
    cv::Mat datamatrix = ef.createdatamatrix(imagevec);
    cv::Mat projection = ef.project(datamatrix.row(0));



    //Load the existing vector.
    std::vector<float> initialProjectionPoints;



    //Load existing face features
    jsize  intArrayLen = env->GetArrayLength(existingfeatures);
    jfloat *pointvecBody = env->GetFloatArrayElements(existingfeatures, 0);


    for (int i =0; i < intArrayLen; i++) {
          initialProjectionPoints.push_back(pointvecBody[i]);

      }

    std::vector<float> newProjectionpoints = ef.matToVector(projection);
    float comparisonScores =  ef.compareProjections(newProjectionpoints, initialProjectionPoints);
    env->ReleaseFloatArrayElements(existingfeatures, pointvecBody, 0);
    env->ReleaseStringUTFChars(pcafilename, pcastring_char);
    //Comparte

    return comparisonScores;
  }

```

With this we are set to build our `.so` library and give it to a java code to make use successfully.

The protocol to build the library is stated in the `CMakeLists.txt`.
It looks like below:

```bash
find_package(JNI REQUIRED)
#Include jni directories
include_directories(${JNI_INCLUDE_DIRS})
file (GLOB_RECURSE SOURCE_FILE src/*.h src/*.cpp)
set(LIBS ${JNI_LIBRARIES}) 
add_library(matchprojection SHARED ${SOURCE_FILE})
target_link_libraries(matchprojection ${LIBS})

```
Building the project should generate a `lib-matchprojection.so` file which can be added to your java project.

However, for android, it is a little tricky in the sense that the build tool is different rather than the official cmake build toolchain, Android has its own build tool for native cpp codes. This is called Native development kit (NDK). This is what will be used to build you c++ native codes for the generated `.so` to be compatible with Android.
Building `.so` for android using NDK will be a whole tutorial on its own hence I will be skipping that for now.
But generally, once the build is complete using an NDK, you will have the same `lib-matchprojection.so` which can be used in your android application.

### Using the `.so` file within Android application.
Using the generated library within the Android application is just the same as using it in any java application.
The idea is to load the native library and then make a call with the required parameter to the methods in the initially created Classes that correspond to the native jni method.
To load the library in any java program including android, ensure the `.so` library is in the class path of your program, some will put it under a folder called `lib`. With this I can use a function call to load the library as below:

```java
System.loads("native-lib")
```
Finally I can make a call to my methods created earlier with the parameters necessary and the native code then can execute for me.

```java

MatchFeatures mf = new MatchFeatures();

float matchscores = mf.matchpcafeatures(storedpacfilepath, imagebase64string, anotherimageprojectionarray);

```
If you notice carefully, the method was declared native and there is no body for the method, this is because the programmes understand that there is a native cpp method that has been defined with this classpath name.

### Conclusion:
This approach is a fundamental way to build and deploy project that include native codes to a java environment. Also,most complex algorithmic problems including machine learning anc core computer vision project can easily be reason around in cpp most because it is fast and there are production ready libraries available.
Even tensorflow has an api for loading deep learning models in c++ as well as using `tflite` models in c++.
Hence I see this run through approach as a way to build robust production ready engines that will leverage high precision mathematics and most especially deployment of machine learning models to various environment and in particular android environment in an offline environment.

