Recently I have been fascinated with how interesting it is to build mathematically inclined application and deploy at scale and without any restriction of model size, platform or need for api calls. I know that python has enough library for working with prototypes of machine learning project, however not many are talking about scaling this project especially when you don't want to do that over a web api.
I believe true intelligence shouldn't rely only on calls to an api for a model to be available in scale, this fascination led me to research into what it will take to use C++ for machine learning and general intelligence.
My convitiction is that both matlab and python's mathematical strenght are based on an underlying c/c++ code hence fundamentally scaling technology to work with mathematical computations involved in machine learning with blazing fast scenrio in mind will likely require that you are able to dig into low level programming and most especially with c/c++, I choose c++.
Also, I wondered why fundamentally most computer science schools ensures that there is a c/c++ curricullum in there study, this emphasizes the reasoning of using c/c++ for scalable technology.

After learning c++ using [a udemy handson course](https://www.udemy.com/course/free-learn-c-tutorial-beginners/) the challenge is now to integrate a simple face recognition application in an android.

The write-up will include some prelimnary approach of what you need to build a c++ project and deploy in android or any other os environment.

Components:
- Set up a c++ project for machine learning with opencv.
- Learning a PCA to generate eigen faces
- Setting up a .so inference library for multiplatform deployment 
- Developing a jni wrapper for the inference library.
- Deployment in Android using ndk and android studio.

The first part will be to learn a machine learning algorithm with openCV. In this case we are going to explore the most basic of face recognition algorithm, using Principal component for eigenfaces. The machine learning community is very familiar with this in python especially with tools such as scikit learn, but when production and most especially offline/on-device production comes to mind, the need to do this from a different dimension is expedient.
OpenCV comes with a very good api for learning principal component analysis and it is quite straight forward to learn once you have your data all set up.
Here are the steps:

- Set up a cmake project c++ using Opencv.
The key parts of your CMakeFile.txt is ensuring that there is an opencv library in your project and available for your library to compile. Ideally, before requesting that Cmake find Opencv for you, it is important to have the opencv library installed on your machine.

```bash
find_package(OpenCV REQUIRED)

include_directories(${OpenCV_INCLUDE_DIRS})

set(LIBS ${OpenCV_LIBS}) 

target_link_libraries(featureCalculation ${LIBS})

```

_PS: I had to set up my cmake differently for training and inference. I will share both_ 

- Learn a Principal component analysis.
Now that Opencv is available, learning PCA is quite straight forward. Here is the logic invole:
* Read all image data as an array.
Using `cv::glob` from opencv, all filenames ending with a `.jpg, .png or/and .jpeg` can be read with `cv::imread`, and the data preprocessing of the image data can proceed.
* Crop faces as PCA does much better with the face image than the whole image.
I have found Multi-task Cascaded Convolutional Networks (MTCNN) to be the most reliable yet simple and minimal face detection and cropping model out there. There is an implementation in C++ using the original model with a Caffe network in Opencv (_topic for another article - Face detection and cropping in production using Opencv_).
* Convert cropped faces to gray scale.
This part is pretty straight forward. Using `cv::cvtColor(originalimagemat, grayscaleimagematcontainer, cv::COLOR_BGR2GRAY)` we can convert an original BGR image to grayscale in opencv.
* Other preprocessing - One other preprocessing is to ensure that the data types are right. This is very important because C++ is heavy on precision of data types. It is quite easy to introduce bugs at this point hence the reason to carefully ensure that your data types are right. Apart from this, it is a good idea to normalize your image data and resize all images into a consistent shape. PCA works only if the data are in the same dimension. Out of the bos from opencv we can employ the following functions to take care of the preprocessing:
`cv::resize(originalImage, containermatofnewimage, size)` for resizing the image and `originalmat::convertTo(newmatnormalized, CV_32FC3, 1/255.0)`
* Convert all images to a data table - A data table is somewhat like a single table of data where each element is represented as a row, interestingly we can think of each row of our data table as individual images in it's flattened format. The essence of PCA is to project the image values to few columns with distinct representation of that image. Therefore, the data table will have rows equals to the number of images in the training dataset while the columns will be the normalized grayscale values of each image.
To create the data table, `std::vector` can be used to hold all the images (with the hope that they fit in memory) which is then copied to every row of the data matrix. Here is an helper function that does exactly that from a vector of images mat.

```cpp
cv::Mat createdatamatrix(std::vector<cv::Mat> imageArray) {

    cv::Mat datamatrix(static_cast<int>(imageArray.size()), imageArray[0].rows * imageArray[0].cols, CV_32F);
    unsigned int i;
    for(i=0; i < imageArray.size(); i++) {
        
        cv::Mat imageRow = imageArray[i].clone().reshape(1, 1);
        cv::Mat rowIData = datamatrix.row(i);
        imageRow.copyTo(rowIData);
    }

    return datamatrix;

}
```

`cv::reshape()` helps transform mat arrays to different shapes, with `(1, 1)` it literally means we want the data to exist in a single row.
* Learn the actual Pricipal component analysis algorithm.
Now that we have created the data table with the preprocessed face images, learning the PCA model is usually smooth. As smooth as passing the data table to an opencv pca instance with your expected maximum components like such `cv::PCA pca(datatable, cv::Mat(), cv::PCA::DATA_AS_ROW, number_of_components)`. With this we have a learned PCA written in C++ which is ready for production use.

To transfer this model for use in any environment, open cv has a `FileStorage` object that allows you to save a mat as it is. Thus I can save this file and pass its filename over jni for opencv to recreate the model instances for inferenct. Well it simply as sweet as that to serve the model.
To conclude this part of the article, I will simply show how to write the mat object using opencv. At the end of the day, the values in the saved model comes out as either a YAML file or XML depending on the choice most pleasant to the user.
* Save the pca model object for inference on production environment.
What exactly needs to be saved in the pca object are the mean and eigenvectors of the trained pca, sometimes it may be a good idea to also save the eigen values in case you want to construct your own eigenfaces projection, but opencv already implemented a `pca->project` instance that helps in inference and eigenfaces generation. In any case here is how to save your model:

```cpp
void Facepca::savemodel(cv::PCA pcaModel, const std::string filename){

    cv::FileStorage fs(filename,cv::FileStorage::WRITE);
    fs << "mean" << pcaModel.mean;
    fs << "e_vectors" << pcaModel.eigenvectors;
    fs << "e_values" << pcaModel.eigenvalues;
    fs.release();

}
```

In the next part of this article, I will explain how to set up an inference library in c++ while using jni as your connector to your inference library.
