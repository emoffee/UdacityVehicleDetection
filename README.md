# Vehicle Detection & Tracking
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

The objective of this project is to create a robust software pipeline to continously detect vehicles in a recording video by a camera mounted right on the front of a car. Several techniques are either reviewed or combined in the pipeline to achieve robustness.

[//]: # (Image References)
[EDA]: ./lablearn/Proceedings/EDA.png
[FE]: ./lablearn/Proceedings/feature_extraction.png
[SW1]: ./lablearn/Proceedings/slidingwindows1.png
[SW2]: ./lablearn/Proceedings/slidingwindows2.png
[ws]: ./lablearn/Proceedings/windows.png
[hm1]: ./lablearn/Proceedings/heatmap1.png
[hm2]: ./lablearn/Proceedings/heatmap2.png
[HT]: ./lablearn/Proceedings/HeatThresholds.png
[parameters]: ./lablearn/Proceedings/parameters.png
[TR]: ./lablearn/Proceedings/TrainingReport.png
[video1]: ./project_video.mp4

Applied Techniques & Goals
---

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Apply a color transform and append binned color features, as well as histograms of color, to the HOG feature vector.
* Implement a sliding-window technique and use the trained linear classifier to search for vehicles in images.
* Run a pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

Data Descriptions
---

Here are links to the labeled data for [vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/vehicles.zip) and [non-vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/non-vehicles.zip) examples that I used to train my linear SVC classifier. These example images come from a combination of the [GTI vehicle image database](http://www.gti.ssr.upm.es/data/Vehicle_database.html), the [KITTI vision benchmark suite](http://www.cvlibs.net/datasets/kitti/), and examples extracted from the project video itself. 

Please notice that, all 8792 [vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/vehicles.zip) images are PNG while the [non-vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/non-vehicles.zip) are a mixture of JPEG & PNG images. Therefore, I create a function **read_img** to feed in the image data with format consistency.

Six example images for testing this pipeline on single frames are located in the `test_images` folder. Examples of the output from each stage of your pipeline are saved in the folder called `ouput_images`, and are included in this writeup for the project by describing what each image shows.   

The video called `project_video.mp4` is the video this pipeline should work well on, and the 'Project_output.mp4' is produced to detect vehicles by the pipeline in a continous basis.

Pipeline overview
---
### Preparing Data
First, the program reads in ALL `vehicle` and `non-vehicle` images by a pre-defined function 'ReadOImgs' which only accpets PNG & JPEG. As a result, 8792 images from 'vehicle' and 10093 'non-vehicle' are read into memory. To balance the sample numbers of these two classes and to do a quick model exploration, I chose a sample size of 800 to equalize them. After the training model has been tuned, I uncommented the code of creating sample data and therefore all data is used.

Next, all the vehicle and non-vehicle images are fed by a user-defined function **ReadOImgs**. Here is an example of one of each of the vehicle and non-vehicle classes:

![alt text][EDA]


### Feature Extractions with various methods

* Histograms of color
* Bin Spatial
* Histogram of Oriented Gradients (HOG)

In order to extract features from color, gradients and even raw data, I explored the three methods above and put them together into a composite function **extract_features**, which include transform color, create histogram, get hog_features and concatenate color & shape features, to generate car_features & notcar_features.
![alt text][FE]

My experience with tuning HOG parameters was very interesting. I  explored different color spaces(finally choose YUV) and different skimage.hog() parameters (orientations, pixels_per_cell, and cells_per_block). I grabbed more than 20 random images from each of the two classes and displayed them to get a feel for what the skimage.hog() output looks like. The above example shows HOG features extracted from grayscale by using 9 orientations, 8 pixels per cell, and 2 cells per block. These parameters are derived mostly by repeated experiments & tips from the lesson.

![alt text][Parameters]

After the feature extractions, I also have **car_features** & **notcar_features** ready for normalization, standardizations & train-test-split like these:

* fit a per-column scaler
* apply the scaler to X
* split training data into training & test datasets with a ratio of 80/20.



### Train a linear SVC classifier with extracted features
By visually exploring the 3D plots of images & color map, I choose YUV color space to train my liearn SVM classifier. Other parameters have been discussed in the previous section. The resulting accuracy was 98.7%, not too bad, and not surprisingly all the 10 predictions get it right. 
Another interesting fact I obeserve is that, the model had been extremely overfitting(all 100% result accuracy) with a low number of sample data, say 3000. After I increase the sample amount above 5000, the test accuracy goes off 100%, but still above 95% anyway.

![alt text][TR]

### Define sliding window search.
* Define windows to search with lesson function `slide_window`.
* Implement sliding window search with lesson function `search_windows`.

To learn how the sliding window works, I decided to search random window positions at random scales all over the image and came up with this:
![alt text][WS]

Obviously this is not we want. So i started to locate the bounds of X & Y values and to narrow valid areas for sliding windows, in order to eliminate data noises and improve pipeline processing efficiency. Several observations determines the bounds of X & Y I choose:
* The car in the project video is at the very left lane and therefore any area with a X value less than **450** can be excluded.
* The area with a Y value less than 350 is blue sky.
* The area with a Y value slightly greater than 600 should be excluded.

I then tried to find a good tuple of overlap percentage with a perspective of maximizing true positives and minimizing false positives. Too small percentages might cause a more sparse distribution of windows scattered while more then 90% overlaps produce many dense small windows that distort the "heat degree" in the area, as well as falsely high sensibility of false positives windows detection. After several experiment, I found that 85% is acceptable.

Example Test Images:
![alt text][SW1]
![alt text][SW2]
---


### III. Video Implementation

#### 1. Final video output. 

[Video result](Project_output.mp4)

#### 2. Filter for false positives and some method for combining overlapping bounding boxes.

*  Construct a heatmap from very recent frames of video
*  dismiss false positives by applying thresholds.


The windows' positions of detection are added to the test images, as a base of creating occurence heatmap. A threshold value of overlapping detections, 3 ,is choosed to determine how to draw the resulting windows.

![alt text][HT]

Here's an example result showing the heatmap from a series of frames of video, the result of scipy.ndimage.measurements.label() and the bounding boxes then overlaid on the last frame of video:

Here are six frames and their corresponding heatmaps, with resulting bounding boxes drawn onto the last frame in the series:

![alt text][hm1]
![alt text][hm2]

---

## 4. Discussion

#### Problems?

The main problem I have encountered in the video is that, the pipeline appears not to be as robust as I expected. The wierd part of this problem is that, for example, the pipeline failes to recognize the white car in the project video for 5 seconds long. I guess this problem might be caused by 
* A unreliable **LINEAR** classifier that I trained for performance convenience
* A low quality implementation of false positives approach.