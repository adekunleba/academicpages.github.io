---
title: "Confirming a Better Model - Importance of Precision and Recall"
excerpt: "Given a modified machine learning model, how do you know if your newly trained model will make any difference?"
collection: datascience
---

Recently, I was charged with the responsibility of having to re-train a model we had already released into production but found out there were complaints about False positive results, which is actually critical to the process flow of our application. We developed a simple  model to detect if a captured image is a spoof or a live image. This is to be used in our normal day to day application process, the end business goal is to be able to catch Spoof images correctly and yes we did that, however the challenge was that we now have a lot of Live images being recognized as spoof, the onus is to check how to make the model better without so much lose to it's ability to catch spoof.

We did retrained the model and did a test. Here is a sample(figures are changed but reflects the error distribution anyways) result of our test using different thresholds around the probability between the two classes to guage the threshold to be used in production:

Normal acuracy based results:
**Old Model Threshold results on test data:**


| Threshold      | Spoof Image Error | Live Image Error|
| ---------------| ------------------|-----------------|
|  0.2           |       13 %        |     3 %         |
|  0.25          |       11 %        |     4 %         |
|  0.3           |       10 %        |     6 %         |
|  0.35          |       9 %         |     7 %         |
|  0.5           |       7 %         |     8 %         |


**New Model Threshold results on test data after retraining:**

| Threshold      | Spoof Image Error | Live Image Error|
| ---------------| ------------------|-----------------|
|  0.2           |        4 %        |     4 %         |
|  0.25          |        4 %        |     5 %         |
|  0.3           |        4 %        |     6 %         |
|  0.35          |        3 %        |     7 %         |
|  0.5           |        2 %        |     8 %         |


From the above test on both data, it looked like there was no much difference after the training, except that we now have a balanced error on accuracy on the new model than the old model, Well not too fast, here is where metrics used in Data Science comes to shine, so to further convince the stakeholder that the new training actually had an effect, I did the following test:
* Precision check
* Recall check
* F1 Score

This is how that test looks for the new model and the old one.

**Old Model Metrics**

                precision    recall  f1-score   support
        spoof    0.89         0.93      0.91    299000
        live     0.96         0.93      0.95    340000     
        total    0.93         0.93      0.93    639000

**New Model Metrics:**

                precision    recall  f1-score   support
        spoof     0.87        0.98    0.92       299000
        live      0.98        0.91    0.95       340000
    avg/total     0.94        0.94    0.94       639000


The result basically confirm that we have a better model that will likely reduce our false positive rate we are experiencing in production as well carefully help us detects spoof images.
How did I know this:

Consider the Precision for Live Images, we have 98% compared to the former model with 96% this shows that the ability for our model to actually capture live images which is the positive result in this case has increased and remember the formular for precision is given by:

<a href="https://www.codecogs.com/eqnedit.php?latex=Precision&space;=&space;\frac{truepositive}{true&space;positive&space;&plus;&space;false&space;positive}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?Precision&space;=&space;\frac{truepositive}{true&space;positive&space;&plus;&space;false&space;positive}" title="Precision = \frac{truepositive}{true positive + false positive}" /></a>


And to know if our model still works well for spoof images, consider the recall, which is the metric of a classification model in predicting actual positive or negatives, let's say for our use case spoof is the negaitive, calculating recall then becomes:

<a href="https://www.codecogs.com/eqnedit.php?latex=Recall&space;=&space;\frac{truenegative}{true&space;negative&space;&plus;&space;false&space;positive}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?Recall&space;=&space;\frac{truenegative}{true&space;negative&space;&plus;&space;false&space;positive}" title="Recall = \frac{truenegative}{true negative + false positive}" /></a>

With this we can successfully convince our stakeholder that we have done the job well and can then look forward to other approaches to make the model even better but at least we are able to solve the current problem.

_Did this article interests you or spark a cord in your day to day activity, kindly drop a comment will love to learn how you are solving data science problems in your corner._







