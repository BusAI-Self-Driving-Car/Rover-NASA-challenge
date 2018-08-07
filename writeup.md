## Project: Search and Sample Return

---


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./misc/rover_image.jpg
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg 
[image4]: ./output/warped_example.jpg
[image5]: ./output/warped_threshed.jpg
[image6]: ./output/warped_threshed_obstacle.jpg
[image7]: ./output/warped_threshed_rock.jpg
[image8]: ./output/four_compare.jpg


## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

The test images given are processed by a series of image processing functions.

For example, we have a test image with grid as below,

![alt text][image2]

a) Perspective transformation is applied for location and projection on the bird-view map.

![alt text][image4]

b）As in common, color transformation is used to separate the navigable area and the non-navigable. There is a clear boundary between the two since the navigable part is much lighter in color.

The thresh values [160, 160, 160] are carefully picked up using trial and error. From the original RGB image, the navigable areas are much brighter than else. So we can use a lower limit to remain the navigable areas. An efficient way is to use binary search to find the proper value. Here we believe the values we choose are perfect enough since it capture all the bright areas.

![alt text][image5]

c) Similarly, the obstacles and the rock can be colored with adjusted the thresholds and the below are processed results.

Differing from the one in b), the one below highlights the obstacle area as white instead.

![alt text][image6]

The left image contains a yellow rock. Yellow pixels or those close to yellow are remained to reflect the rock in the right image.

![alt text][image3]
![alt text][image7]

c) The pixels are originally described in a global coordinate which would be transformed into a local coordinate in which the rover lies on the origin. A side result this operation can generate is the reference heading by averaging the angles of the navigable pixels.

The left top, right top and the left bottom are the origianl, the warped and the warped and color threshed respectively. The last one is coordinate transformed into a rover centered view. The arrow is the average angle of all the navigable pixels.

![alt text][image8]

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 

The `process_image()` function is generally follow the procedure described in above but in one function. The function accepts a RGB image captured by the camera. After perspective transformation, color filtering and coordinate transformation, the navigable terrain, the obstacles and the rock are represented in the output image and provide a reference heading for the next move of the rover.

### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

The `perception_step()` function is populated in a similar way with `procerss_image()` as described above. The main difference is the input argument is a `RoberState` object called `Rover` instead of an image. In `Rover`, there is everything we need including the camera image, the immediate position of the rover and the according angle.

In the end of the `perception_step()` function a world map is generated including the view in the rover centered coordinate. Red, Green and Blue pixels are representing obstacles, rocks and navigable terrains respectively. Also on the map, the explored areas are colored in the same way. The rock color finally will be in white by color overlapping.

The `decesion_step()` gets the perception information and decide the next steering. Originally, the mean angle of all the pixels is used but in practice, it often gets stuck in a circular track. Given a bias to the left or the right, the problem could be solved. The steering choosing is more aggressive in this way so it can explore more areas.

Here I tried to check if it was get stuck by measuring the steering angle and the time duration. If it keep an upper limit or a lower limit angle for quite a long time, I would assume it get stuck and a command would be sent to switch the steering to the opposite side only to escape the situation.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

My simulator was running at ~30 FPS with 1280x768 resolution and 'Beautiful' graphics quality.

The current solution is able to drive the rover to explore most of the navigable areas with considerable coverage (over 75%) and fidelity (around 59%) but not every run.

It sometime falls into the turning round forever in a circle and loses the chance to explore which can hopefully solved by changing the rule of steering.
