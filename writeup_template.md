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
[image14]: ./calibration_images/udacity_perception_notes.jpg
[image15]: ./calibration_images/youtube_robond.jpg

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

I got stuck sometimes and had to figure out why. Learning is exploring. I think I might have tried to debug something for 2 hours but when I finally figured it out I jumped around the room. No one was watching.

![alt text][image9]


**I also took lots of notes. That helped quite a bit.**

![alt text][image14]


**In the Notebook I added this function from the Walthrough Video as a rock finding test and it worked out. I incorporated it into the perception.py file later on.**

**# Let's discover some rocks!
    rock_map = discover_rocks(warped, levels=(110, 50, 60))
    if rock_map.any():
        rock_x, rock_y = rover_coords(rock_map)
        rock_x_world, rock_y_world = pix_to_world(rock_x, rock_y, xpos, ypos, yaw, world_size, scale)
        data.worldmap[rock_y_world, rock_x_world, :] = 255**

Everything else was pretty straightforward from the exercises that I put into the notebook.


#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result.

Everything I did within **def process_image(img):** was fairly straightforward except for twiddling a few knobs.

**1) Define source and destination stuff**
**2) Apply perspective transform + mask**
    
    warped, mask = perspect_transform(img, source, destination)

**3) Apply color threshold to identify navigable terrain/obstacles/rock samples
threshed = color_thresh(warped)**
    
    obs_map = np.absolute(np.float32(threshed) - 1) * mask

**4) Convert thresholded image pixel values to rover-centric coords**
    
    xpix, ypix = rover_coords(threshed)

**5) Convert rover-centric pixel values to world coords**
    
    world_size = data.worldmap.shape[0]
    scale = 2 * dst_size
    xpos = data.xpos[data.count]
    ypos = data.ypos[data.count]
    yaw = data.yaw[data.count]
    x_world, y_world = pix_to_world(xpix, ypix, xpos, ypos, yaw, world_size, scale)
    obsxpix, obsypix = rover_coords(obs_map)
    obs_x_world, obs_y_world = pix_to_world(obsxpix, obsypix, xpos, ypos, yaw, world_size, scale)

**6) Update worldmap (to be displayed on right side of screen)**

    data.worldmap[y_world, x_world, 2] = 255
    data.worldmap[obs_y_world, obs_x_world, 0] = 255
    nav_pix = data.worldmap[:,:,2] > 0
    data.worldmap[nav_pix, 0] = 0

**6a) Let's discover some rocks!**
    
    rock_map = discover_rocks(warped, levels=(110, 50, 60))
    if rock_map.any():
        rock_x, rock_y = rover_coords(rock_map)
        rock_x_world, rock_y_world = pix_to_world(rock_x, rock_y, xpos, ypos, yaw, world_size, scale)
        data.worldmap[rock_y_world, rock_x_world, :] = 255

**7) Make a mosaic image**
    
    output_image = np.zeros((img.shape[0] + data.worldmap.shape[0], img.shape[1]*2, 3))
    output_image[0:img.shape[0], 0:img.shape[1]] = img
    warped, mask = perspect_transform(img, source, destination)
    output_image[0:img.shape[0], img.shape[1]:] = warped
    map_add = cv2.addWeighted(data.worldmap, 1, data.ground_truth, 0.5, 0)
    output_image[img.shape[0]:, 0:data.worldmap.shape[1]] = np.flipud(map_add)
    
    
### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

The only things I did differently in perception.py were the following:

**Step 7: Update Rover worldmap (to be displayed on right side of screen)**
    
    Example: Rover.worldmap[obstacle_y_world, obstacle_x_world, 0] += 1
            Rover.worldmap[rock_y_world, rock_x_world, 1] += 1
            Rover.worldmap[navigable_y_world, navigable_x_world, 2] += 1
    
    **Favoring navigable terrain (Blue Channel) here:**
    
    Rover.worldmap[rock_y_world, rock_x_world, 2] += 10
    Rover.worldmap[obs_y_world, obs_x_world, 0] += 1
    nav_pix = Rover.worldmap[:,:,2] > 0
    Rover.worldmap[nav_pix, 0] = 0

**Find rocks: I changed this up from Notebook to include the rock_idx and xcen and ycen then add to the Rover.vision_image[:,:,1] * 255**
    
    rock_map = discover_rocks(warped, levels=(110, 110, 50))
    if rock_map.any():
        rock_x, rock_y = rover_coords(rock_map)

        rock_x_world, rock_y_world = pix_to_world(rock_x_world, rock_y_world, Rover.pos[0], Rover.pos[1], Rover.yaw, world_size, scale)

        rock_dist, rock_ang = to_polar_coords(rock_x_world, rock_y_world)

        rock_idx = np.argmin(rock_dist)
        rock_xcen = rock_x_world[rock_idx]
        rock_ycen = rock_y_world[rock_idx]

        Rover.worldmap[rock_ycen, rock_xcen, 1] = 255
        Rover.vision_image[:,:,1] = rock_map * 255
    else:
        Rover.vision_image[:,:,1] = 0

** In the decision.py file I did change the Rover.rocks_angles function up a bit but there is still a bunch of work to do since I dont think it's working so well currently.**
    
    If in a state where want to pickup a rock send pickup command
    if Rover.rocks_angles is not None and len(Rover.rocks_angles) > 0:
        Rover.steer = np.clip(np.mean(Rover.rocks_angles * 180/np.pi), -15, 15)
        if not Rover.near_sample:
            if Rover.vel < 1:
                Rover.brake = 0
                Rover.throttle = 0.1
        else:
            Rover.throttle = 0
            Rover.brake = Rover.brake_set

    if Rover.near_sample:
        Rover.brake = Rover.brake_set

    if Rover.near_sample and Rover.vel == 0 and not Rover.picking_up:
        Rover.send_pickup = True


#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.

Here's my Rover in it's current state:

[![IMAGE ALT TEXT](http://img.youtube.com/vi/KfE-j5eh840/0.jpg)](http://www.youtube.com/watch?v=KfE-j5eh840 "Udacity RoboND Simulator")

The settings used are 1280 x 800 windows mode on "Fantastic" quality with an average FPS of 26. It was run on Windows 10 64bit.

**I am going to pursue this project further with more time. Some things I want to implement:**

    1. Left side wall crawling so the Rover picks up samples.
    2. Looking for the first wall to follow.
    3. Picking Up Sample - Picking up a rock sample.
    4. Going Home after picking up all samples.
    5. I'm Home! Notify on arriving at Home Base.
    6. Unsticking myself from the smaller rocks in the center of the Map.
    7. Learning more about socketio and Flask and how they work together with the Unity Simulator or ROS/Gazebo.


