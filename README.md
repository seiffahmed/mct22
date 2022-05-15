**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/example_data.png
[image2]: ./output_images/hog_example.png
[image3]: ./output_images/diff_windows.png
[image4]: ./output_images/all_possible_windows.png
[image5]: ./output_images/detect_and_heat_examples.png
[image6]: ./output_images/label_bound_example.png
[image7]: ./output_images/multi_window_detect.png
[video1]: ./project_video_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is in P5.ipynb under the title 'Display HOG'

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of some of the `vehicle` and `non-vehicle` classes:

![example images][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=15`, `pixels_per_cell=(16, 16)` and `cells_per_block=(2, 2)`:


![HOG Example][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and I chose the YUV colorspace due to success in previous projects and it's preference for how humans see color. I later changed this to YCrCb since this seems better when working in the digital rather than analog realm. I had success in the classes using orientations: 15, pix_per_block: 8 and cell_per_block: 2. I later changed pix_per_block to 16 since it didn't seem to effect accuracy and the HOG calculations were much faster. If I had more time I would like to have implemented grid search to try out all the different possible combinations of parameters in case there is a better one. At 99% accuracy I'm happy to continue with what I've got.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM in P5.ipynb under the title 'Train Classifier'. For feature extraction I combined the HOG values for each of the three image channels with the spatial and histogram features for a total feature vector length of 2580. I also used the StandardScaler during feature extraction to determine the mean and range over the entire dataset and then normalize all the data to be mean centered with a normal range. I re-use the scaler thoughout the remaining steps so that the same modifications are made to standardize new test images and video frames. I just used the linear SVM and before training I randomly split the data into 80% training and 20% validation. With these settings I achieved 99% accuracy 

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I first implemented the simple sliding window method from class under the title 'Algorithm to Determine Sliding Windows'. This worked succesfully on my test image but was unlikey to work in other scenarios due to the narrow scale and search size. I then implemented the find cars method from the lessons which just calculates the HOG features once, then sub-samples those features for each window of interest. This method takes a y-start, y-stop and scale parameter to determine where in the image the windows should be placed and how large they need to be. There is also an inherent overlap of the windows in this method. I experimented with a set of different values that only focused on the road region with smaller windows searching closer to the horizon and larger windows search closer to the front of the car. I also tweaked the algorithm a little to make sure the search windows occured right up to the edges of the left and right of the image. Below are examples of each set of search windows along with a final image of all the search windows drawn at once. I used multiple random colors to try and give a better sense of which lines belong to which search windows.

![different search windows][image3]

![all search windows][image4]

And here is an example of the search windows and classifer applied to an image:

![multi search window classification][image7]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on five scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images (which also show the thresholded heap map that I explain below):

![alt text][image5]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. I collected 30 frames of detection windows to generate my heatmap, then I used a threshold of 0.75 * 30, which is slightly less than a threshold of 1 for each individual frame. I found this gave the best results of strong lables for vehicles and no false positives.

You can see examples of heat maps in the previous image, here is another where the label has been applied to a heat map, then bounding boxes determined from said labels

![labels to bounding boxes][image6]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I'm not 100% confident that I chose the best parameters and given more time I'd like to perform a gridsearch to determine if there are better options available. I also tried to pre-process the data such that differences in brightness would be ignored by equalizing the histogram, based on review feedback from a previous project. This gave me a very marginally higher accuracy in the classifier, but when applied to images it seemed to be very bad at detecting vehicles. This is something else I'd like to return to. It would be interesting to test my approach with different weather and lighting conditions, it may fail in that scenario and require more training data.

I also found that it took a long time to process the video on a pretty tricked out laptop, far from real-time. It would be interested to do a type of grid search where I can compare the run-time and accuracy trade of an perhaps choose a classifier and feature extraction that's far faster with no major loss in accuracy.

I also only tried out the standard linear SVM. With more time I would have liked to try the radial with various parameters, perhaps again in some kind of grid search harness.

