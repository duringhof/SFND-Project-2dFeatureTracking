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

TBD

---

#### Task MP.8 Performance Evaluation 2
> Count the number of matched keypoints for all 10 images using all possible combinations of detectors and descriptors. In the matching step, the BF approach is used with the descriptor distance ratio set to 0.8.

TBD

---

#### Task MP.0 Performance Evaluation 3
> Log the time it takes for keypoint detection and descriptor extraction. The results must be entered into a spreadsheet and based on this data, the TOP3 detector / descriptor combinations must be recommended as the best choice for our purpose of detecting keypoints on vehicles.

TBD
