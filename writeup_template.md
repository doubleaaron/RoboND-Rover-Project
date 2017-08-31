## Project: Search and Sample Return
### Rover Search and Sample Return Aaron's version!

---

Notebook Analysis

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
[image4]: ./calibration_images/aa_capture.jpg
[image5]: ./calibration_images/aa_capture_mask.jpg
[image6]: ./calibration_images/aa_capture_ground_truth.jpg
[image7]: ./calibration_images/aa_capture_rover_camera_view.jpg
[image8]: ./calibration_images/aa_capture_wall_crawling.jpg
[image9]: ./calibration_images/aa_capture_getting_stuck.jpg
[image11]: ./calibration_images/aa_capture_telemetry.jpg
[image12]: ./calibration_images/aa_capture_warped.png
[image13]: ./calibration_images/aa_capture_rover_vision_working.jpg

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
I felt like I was really bad at numpy and OpenCV at the beginning of this Project, but after going through the docs I started to feel like I was understanding numpy color channel operations, perspective warping in OpenCV, radians, polar coords and a bunch of other things I'd never worked on before. This is all new to me so it's incredibly exciting!

The first thing I worked on was the perspective transform.

Using the getPerspectiveTransform and warpPerspective functions in OpenCV opened my eyes. I utilized the mask method from the Video Walkthrough to clip the funneled outside of the camera on the Rover.

![alt text][image12]

Then I moved on to nav, obstacle, and rock sample thresholding and rover_coords using:
**threshed = color_thresh(warped)**

Obstacle thresholds are just the opposite of Navigable thresholds, so I used **obs_map** to take absolute values of threshed map and return 0's where obstacles are, multiplied by the mask: **obs_map = np.absolute(np.float32(threshed) - 1) * mask**

After that I experimented with theVideo Walkhrough function for finding rocks instead of the Rover directly in the perception.py file using: **rock_map = discover_rocks(warped, levels=(110, 110, 50))** and updated the **Rover.vision_image[:,:,1] = rock_map * 255** in that function. I played around with the levels quite a bit and used Photshop to see if I could get a better range than **levels=(110, 110, 50)** but it didn't work out as well as the provided R,G,B.

I jumped back and forth from the code in perception.py, decision.py and drive_rover.py quite a bit trying to get my head around the way things work. Between that and the notebook, I experimented as I learned. I would go back through the assignments and examples quite a bit. You can see the image below where I had the Blue Channel wrong for Rover Mapping bottom left dashboard. Things like that would happen and I'd have to track down what was going on. It was fun. 

![alt text][image8]

Blue Channel working!

![alt text][image13]

I also got stuck alot and had to figure out why.

![alt text][image9]

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
In the Notebook I added this function as a rock finding test and it worked out:

**# Let's discover some rocks!
    rock_map = discover_rocks(warped, levels=(110, 50, 60))
    if rock_map.any():
        rock_x, rock_y = rover_coords(rock_map)
        rock_x_world, rock_y_world = pix_to_world(rock_x, rock_y, xpos, ypos, yaw, world_size, scale)
        data.worldmap[rock_y_world, rock_x_world, :] = 255**

Everything else was pretty straightforward from the exercises:

# Define a function to pass stored images to
# reading rover position and yaw angle from csv file
# This function will be used by moviepy to create an output video
def process_image(img):
    # Example of how to use the Databucket() object defined above
    # to print the current x, y and yaw values 
    # print(data.xpos[data.count], data.ypos[data.count], data.yaw[data.count])

    # TODO: 
    # 1) Define source and destination points for perspective transform
    # 2) Apply perspective transform + mask
    warped, mask = perspect_transform(img, source, destination)
    
    # 3) Apply color threshold to identify navigable terrain/obstacles/rock samples
    threshed = color_thresh(warped)
    
    # 3) obs_map takes absolute value of threshed map and returns 0's where obstacles are, multiplied by the mask
    obs_map = np.absolute(np.float32(threshed) - 1) * mask
    
    # 4) Convert thresholded image pixel values to rover-centric coords
    xpix, ypix = rover_coords(threshed)
    
    # 5) Convert rover-centric pixel values to world coords
    world_size = data.worldmap.shape[0]
    scale = 2 * dst_size
    xpos = data.xpos[data.count]
    ypos = data.ypos[data.count]
    yaw = data.yaw[data.count]
    x_world, y_world = pix_to_world(xpix, ypix, xpos, ypos, yaw, world_size, scale)
    
    obsxpix, obsypix = rover_coords(obs_map)
    obs_x_world, obs_y_world = pix_to_world(obsxpix, obsypix, xpos, ypos, yaw, world_size, scale)
    
    # 6) Update worldmap (to be displayed on right side of screen)
        # Example: data.worldmap[obstacle_y_world, obstacle_x_world, 0] += 1
        #          data.worldmap[rock_y_world, rock_x_world, 1] += 1
        #          data.worldmap[navigable_y_world, navigable_x_world, 2] += 1

    data.worldmap[y_world, x_world, 2] = 255
    data.worldmap[obs_y_world, obs_x_world, 0] = 255
    nav_pix = data.worldmap[:,:,2] > 0
    data.worldmap[nav_pix, 0] = 0
    
    # Let's discover some rocks!
    rock_map = discover_rocks(warped, levels=(110, 50, 60))
    if rock_map.any():
        rock_x, rock_y = rover_coords(rock_map)
        rock_x_world, rock_y_world = pix_to_world(rock_x, rock_y, xpos, ypos, yaw, world_size, scale)
        data.worldmap[rock_y_world, rock_x_world, :] = 255

    # 7) Make a mosaic image, below is some example code
        # First create a blank image (can be whatever shape you like)
    output_image = np.zeros((img.shape[0] + data.worldmap.shape[0], img.shape[1]*2, 3))
        # Next you can populate regions of the image with various output
        # Here I'm putting the original image in the upper left hand corner
    output_image[0:img.shape[0], 0:img.shape[1]] = img

        # Let's create more images to add to the mosaic, first a warped image
    warped, mask = perspect_transform(img, source, destination)
        # Add the warped image in the upper right hand corner
    output_image[0:img.shape[0], img.shape[1]:] = warped

        # Overlay worldmap with ground truth map
    map_add = cv2.addWeighted(data.worldmap, 1, data.ground_truth, 0.5, 0)
        # Flip map overlay so y-axis points upward and add to output_image 
    output_image[img.shape[0]:, 0:data.worldmap.shape[1]] = np.flipud(map_add)


        # Then putting some text over the image
    cv2.putText(output_image,"Populate this image with your analyses to make a video!", (20, 20), 
                cv2.FONT_HERSHEY_COMPLEX, 0.4, (255, 255, 255), 1)
    if data.count < len(data.images) - 1:
        data.count += 1 # Keep track of the index in the Databucket()
    
    return output_image

![alt text][image2]
### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.


#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  



![alt text][image3]


