# SFND 2D Feature Tracking - Mid-term project

This repository contains the submission for the mid-term project related to "Camera Based 2D Feature Tracking", which is part of Udacity's Sensor Fusion Nanodegree program.

To obtain the starter code, read about dependencies and/or basic build instructions, please refer to the following repository:
https://github.com/udacity/SFND_2D_Feature_Tracking.git

## Report

---

#### Task MP.0 Mid-Term Report
> Provide a Writeup / README that includes all the rubric points and how you addressed each one. You can submit your writeup as markdown or pdf.

This README is created to meet this specification.

---

#### Task MP.1 Data Buffer Optimization
> Implement a vector for dataBuffer objects whose size does not exceed a limit (e.g. 2 elements). This can be achieved by pushing in new elements on one end and removing elements on the other end.

```c++
// limit data frame buffer size by removing oldest frame
if(dataBuffer.size()>dataBufferSize){
  dataBuffer.erase(dataBuffer.begin());
}
```
---

#### Task MP.2 Keypoint Detection
> Implement detectors HARRIS, FAST, BRISK, ORB, AKAZE, and SIFT and make them selectable by setting a string accordingly.

```c++
if (detectorType.compare("SHITOMASI") == 0) {
  detKeypointsShiTomasi(keypoints, imgGray, false);
} else if (detectorType.compare("HARRIS") == 0) {
  detKeypointsHarris(keypoints, imgGray, false);
} else {
  detKeypointsModern(keypoints, imgGray, detectorType, false);
}
```
Actual detector implementations can be read in matching2D_Student.cpp
---

#### Task MP.3 Keypoint Removal
> Remove all keypoints outside of a pre-defined rectangle and only use the keypoints within the rectangle for further processing.

```c++
// only keep keypoints on the preceding vehicle
bool bFocusOnVehicle = true;
cv::Rect vehicleRect(535, 180, 180, 150);
if (bFocusOnVehicle) {

  vector<cv::KeyPoint> keypointsOfInterest;
  for (auto it=keypoints.begin(); it!=keypoints.end(); ++it) {
    if (vehicleRect.contains(it->pt)) {
      keypointsOfInterest.push_back(*it);
    }
  }
  keypoints = keypointsOfInterest;
}
```
---

#### Task MP.4 Keypoint Descriptors
> Implement descriptors BRIEF, ORB, FREAK, AKAZE and SIFT and make them selectable by setting a string accordingly.

```c++
void descKeypoints(vector<cv::KeyPoint> &keypoints, cv::Mat &img,
                   cv::Mat &descriptors, string descriptorType) {

  // select appropriate descriptor
  cv::Ptr<cv::DescriptorExtractor> extractor;
  if (descriptorType.compare("BRISK") == 0) {

    int threshold = 30;        // FAST/AGAST detection threshold score.
    int octaves = 3;           // detection octaves (use 0 to do single scale)
    float patternScale = 1.0f; // apply this scale to the pattern used for
                               // sampling the neighbourhood of a keypoint.
    extractor = cv::BRISK::create(threshold, octaves, patternScale);
  } else if (descriptorType.compare("BRIEF") == 0) {
    extractor = cv::xfeatures2d::BriefDescriptorExtractor::create();
  } else if (descriptorType.compare("ORB") == 0) {
    extractor = cv::ORB::create();
  } else if (descriptorType.compare("FREAK") == 0) {
    extractor = cv::xfeatures2d::FREAK::create();
  } else if (descriptorType.compare("AKAZE") == 0) {
    extractor = cv::AKAZE::create();
  } else if (descriptorType.compare("SIFT") == 0) {
    extractor = cv::xfeatures2d::SIFT::create();
  }
    
  // perform feature description
  double t = (double)cv::getTickCount();
  extractor->compute(img, keypoints, descriptors);
  t = ((double)cv::getTickCount() - t) / cv::getTickFrequency();
  cout << descriptorType << " descriptor extraction in " << 1000 * t / 1.0 << " ms" << endl;
}
```
---

#### Task MP.5 Descriptor Matching
> Implement FLANN matching as well as k-nearest neighbor selection. Both methods must be selectable using the respective strings in the main function.

```c++
if (matcherType.compare("MAT_BF") == 0) {
  matcher = cv::BFMatcher::create(cv::NORM_HAMMING, crossCheck);
} else if (matcherType.compare("MAT_FLANN") == 0) {
  matcher = cv::FlannBasedMatcher::create();
}
```
---

#### Task MP.6 Discriptor Distance Ratio
> Use the K-Nearest-Neighbor matching to implement the descriptor distance ratio test, which looks at the ratio of best vs. second-best match to decide whether to keep an associated pair of keypoints.

```c++
// perform matching task
if (selectorType.compare("SEL_NN") == 0) { // nearest neighbor (best match)

  matcher->match(
      descSource, descRef,
      matches); // Finds the best match for each descriptor in desc1
    
} else if (selectorType.compare("SEL_KNN") ==
           0) { // k nearest neighbors (k=2)

  vector<vector<cv::DMatch>> kmatches;
  matcher->knnMatch(descSource, descRef, kmatches, 2);
  for (auto kmatch : kmatches) {
    if (kmatch.size() == 2 && kmatch[0].distance < 0.8 * kmatch[1].distance) {
      matches.push_back(kmatch[0]);
    }
  }
}
```
---

#### Task MP.7 Performance Evaluation 1
> Count the number of keypoints on the preceding vehicle for all 10 images and take note of the distribution of their neighborhood size. Do this for all the detectors you have implemented.

|         |1  |2  |3  |4  |5  |6  |7  |8  |9  |10 |
|---------|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
|SHITOMASI|125|118|123|120|120|113|114|123|111|112|
|HARRIS   |17 |14 |18 |21 |26 |43 |18 |31 |26 |34 |
|FAST     |149|152|150|155|149|149|156|150|138|143|
|BRISK    |264|282|282|277|297|279|289|272|266|254|
|ORB      |92 |102|106|113|109|125|130|129|127|128|
|AKAZE    |166|157|161|155|163|164|173|175|177|179|
|SIFT     |138|132|124|137|134|140|137|148|159|137|

---

#### Task MP.8 Performance Evaluation 2
> Count the number of matched keypoints for all 10 images using all possible combinations of detectors and descriptors. In the matching step, the BF approach is used with the descriptor distance ratio set to 0.8.

|Detector-Descriptor|2  |3  |4  |5  |6  |7  |8  |9  |10 |
|-------------------|---|---|---|---|---|---|---|---|---|
|SHITOMASI-BRISK    |84 |80 |73 |77 |74 |70 |79 |81 |72 |
|SHITOMASI-BRIEF    |96 |93 |92 |89 |92 |93 |85 |91 |85 |
|SHITOMASI-ORB      |86 |84 |87 |91 |87 |76 |81 |88 |88 |
|SHITOMASI-FREAK    |66 |66 |64 |63 |62 |64 |61 |65 |63 |
|SHITOMASI-SIFT     |112|109|104|103|99 |101|96 |106|97 |
|HARRIS-BRISK       |11 |9  |10 |11 |16 |14 |12 |21 |17 |
|HARRIS-BRIEF       |12 |12 |14 |17 |17 |16 |12 |20 |21 |
|HARRIS-ORB         |11 |11 |14 |17 |19 |19 |13 |21 |20 |
|HARRIS-FREAK       |11 |9  |13 |14 |13 |18 |10 |17 |18 |
|HARRIS-SIFT        |14 |11 |16 |19 |22 |22 |13 |24 |22 |
|FAST-BRISK         |80 |91 |85 |94 |71 |83 |92 |86 |94 |
|FAST-BRIEF         |88 |102|88 |102|94 |98 |112|107|92 |
|FAST-ORB           |96 |102|87 |94 |87 |100|101|96 |99 |
|FAST-FREAK         |64 |80 |65 |79 |61 |76 |83 |77 |82 |
|FAST-SIFT          |118|123|110|119|114|119|123|117|103|
|BRISK-BRISK        |138|144|133|144|139|155|137|150|158|
|BRISK-BRIEF        |138|166|129|141|148|155|158|161|148|
|BRISK-ORB          |94 |107|88 |97 |85 |114|112|114|122|
|BRISK-FREAK        |114|121|113|118|103|129|135|129|131|
|BRISK-SIFT         |182|193|169|183|171|195|194|176|183|
|ORB-BRISK          |60 |65 |65 |76 |72 |83 |83 |73 |72 |
|ORB-BRIEF          |37 |38 |37 |53 |42 |64 |58 |62 |59 |
|ORB-ORB            |40 |57 |49 |54 |57 |68 |71 |62 |72 |
|ORB-FREAK          |39 |33 |37 |40 |33 |40 |41 |39 |44 |
|ORB-SIFT           |67 |79 |78 |79 |82 |95 |95 |94 |94 |
|AKAZE-BRISK        |126|112|121|117|114|119|134|140|127|
|AKAZE-BRIEF        |108|116|110|109|116|129|133|135|131|
|AKAZE-ORB          |102|96 |96 |84 |90 |116|103|113|118|
|AKAZE-FREAK        |103|105|93 |99 |97 |115|126|118|117|
|AKAZE-SIFT         |134|134|130|136|137|147|147|154|151|
|SIFT-BRISK         |57 |63 |58 |61 |55 |52 |54 |63 |73 |
|SIFT-BRIEF         |63 |72 |64 |66 |52 |57 |72 |67 |84 |
|SIFT-FREAK         |59 |63 |54 |64 |51 |50 |47 |53 |65 |
|SIFT-SIFT          |82 |81 |85 |93 |90 |81 |82 |102|104|

---

#### Task MP.9 Performance Evaluation 3
> Log the time it takes for keypoint detection and descriptor extraction. The results must be entered into a spreadsheet and based on this data, the TOP3 detector / descriptor combinations must be recommended as the best choice for our purpose of detecting keypoints on vehicles.

|Detector-Descriptor|Average nr. of keypoints detected|Average time needed to extract detectors and descriptors|
|-------------------|---------------------------------|--------------------------------------------------------|
|SHITOMASI-BRISK    |117                              |23.1491                                                 |
|SHITOMASI-BRIEF    |117                              |20.5273                                                 |
|SHITOMASI-ORB      |117                              |22.1362                                                 |
|SHITOMASI-FREAK    |117                              |58.0598                                                 |
|SHITOMASI-SIFT     |117                              |40.025                                                  |
|HARRIS-BRISK       |24                               |18.2584                                                 |
|HARRIS-BRIEF       |24                               |18.1649                                                 |
|HARRIS-ORB         |24                               |18.0999                                                 |
|HARRIS-FREAK       |24                               |59.5445                                                 |
|HARRIS-SIFT        |24                               |37.7598                                                 |
|FAST-BRISK         |149                              |3.23004                                                 |
|FAST-BRIEF         |149                              |2.10957                                                 |
|FAST-ORB           |149                              |2.40153                                                 |
|FAST-FREAK         |149                              |44.1012                                                 |
|FAST-SIFT          |149                              |35.2295                                                 |
|BRISK-BRISK        |276                              |48.3499                                                 |
|BRISK-BRIEF        |276                              |46.4767                                                 |
|BRISK-ORB          |276                              |49.8446                                                 |
|BRISK-FREAK        |276                              |89.7543                                                 |
|BRISK-SIFT         |276                              |117.31                                                  |
|ORB-BRISK          |116                              |11.0091                                                 |
|ORB-BRIEF          |116                              |10.2022                                                 |
|ORB-ORB            |116                              |15.1415                                                 |
|ORB-FREAK          |116                              |51.5156                                                 |
|ORB-SIFT           |116                              |89.0825                                                 |
|AKAZE-BRISK        |167                              |123.897                                                 |
|AKAZE-BRIEF        |167                              |123.141                                                 |
|AKAZE-ORB          |167                              |124.86                                                  |
|AKAZE-FREAK        |167                              |170.615                                                 |
|AKAZE-SIFT         |167                              |156.503                                                 |
|SIFT-BRISK         |138                              |188.016                                                 |
|SIFT-BRIEF         |138                              |185.874                                                 |
|SIFT-FREAK         |138                              |227.351                                                 |
|SIFT-SIFT          |138                              |260.772                                                 |


This leads to the following Top 3:

|Detector-Descriptor|Average nr. of keypoints detected|Average time needed to extract detectors and descriptors|
|-------------------|---------------------------------|--------------------------------------------------------|
|FAST-BRIEF         |149                              |2.10957                                                 |
|FAST-ORB           |149                              |2.40153                                                 |
|FAST-BRISK         |149                              |3.23004                                                 |