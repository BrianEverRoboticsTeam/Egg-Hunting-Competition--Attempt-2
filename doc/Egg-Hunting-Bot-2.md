# Egg Hunting Bot 2
This documentation described the approaches of our "egg hunting behavior" (target searching behavior) on turtlebot 2 (Kobuki) for the Egg Hunting Competition (attempt 2).  

**Contributors:**
- Chenrui Lei
- Siqi Yan
- Tiancheng(Tyler) Shen

**Date of the Competition:**
  April 6th, 2017

# Overview
This is the second attempt of the competition Egg Hunting within a Known Map. In this competition, our robot's general goal is to find visual targets as many as it can during the competition. The visual targets or eggs are represented by a certain type of AR Tag combination (Graph 1) or a University of Alberta emblem (Graph 2).

A competing robot will gain 5 points for finding AR Tag target or University of Alberta emblem. After the robot finds a target, it could gain 3 to 9 points additional if it successfully docked into the 50 cm by 50 cm area in front of the target: 9 points for clear sides docking, 6 points for touching one side docking, and 3 points for touching two sides docking (Graph 3).

There are random number of targets in each round of competition. For a certain target, only the first time of finding and/or docking procedure will be counted towards into the final competition grade. That being said, a robot won't get any points for docking or finding the same target more than once.

The competition has two rounds in total for each competing robot. Each round will last 5 minutes or until all the targets be found. Points that gain in two rounds will be cumulated into the final competition grade. The team with highest final competition grade wins the competition. Tie-breaking based on time of completion if all targets are found within five minutes in both runs.

The initial position and orientation of robots will be random pick by the judges at the beginning of each competition round. A pre-built map of competition environment will be allowed to use for robot's navigation.

# Background/Motivation
In order to make our robot as a competitive one during the competition, our robot will need to be able to perform in 4 separated tasks: localization, navigation, target searching, and docking.

Localization will be critical for target searching especially in a non-closed environment and a unknown initial pose. A good localization could prevent a robot goes outside of competition area, and help robot program to record correct location information of each visited target. Both of those features will definitely help a robot saving a lot of time during the competition.

Navigation gave us a chance to guide our robot in a high efficient target searching route specifically for this competition. It is guarantee that all the targets will be pasted at a certain height on a wall and within a certain area (four walls in the competition hall way). So, if we could keep a fixed distance from camera sensor to the target (or the wall) throughout the competition, the size of target detected by a camera frame will be fixed as well. If this is possible, we are able to improve our target searching algorithm by getting rid of iterations for searching multiple possible target size in each image frame that captured by camera sensor. That means, there exist a high efficient distance between camera sensor and the walls for target searching. The navigation is the key to guide our robot traveling in this high efficient route.

Successful target searching algorithm will help robots to collect points in the competition. Mostly, a robot won't have a second chance to find the same target within 5 minutes due to the size of the competition area. Therefore, good target searching algorithm should be able to prevent missing targets, meanwhile, helps a robot to gain more points within the same time duration. The higher points a robot gains, the higher chance it could win the game.

Docking procedure will make our robot gains 3 to 9 points depends on the quality of docking. Unlike the attempt 1, this docking points are actually more important compare to the points for successfully finding a target. Therefore, docking procedure could obtain the most of the points from a round which affects a lots to the final competition grade. 

To add up to those four skills of our robot, it should also able to perform as a finite state machine. By doing this, those four different skills and/or algorithms are able to coordinate with each other by translating between those four major states when the conditions of a certain state met. There are some other minor components, like the map for navigation, the control program for docking and undocking, etc, will be include in our approach.

To sum up, a good localization makes navigation possible, then makes target searching high efficiently. And, docking makes higher chance for winning. At the end, finite state machine makes those functions works all together.

# Question/Hypothesis
As shown by the pervious competition (Race with GMapping and AMCL) which took place at the same environment as this competition, GMapping and AMCL are very good for mapping and navigation, so we could just take the previous code as a module for navigation, and use the same map from the previous competition. And, the localization could be done by AMCL as well. And, we are trying to prove that AMCL is an proper solution to localize the robot itself.

For target searching, we planned to use template matching technic since it's able to find a target far away (works within about 7 meters during our tests) from the target. The problem is the efficiency of template matching algorithm. This algorithm could only detect a certain size of template in a single iteration. Therefore, if the size of template on an image frame is unknown, this template matching algorithm need multiple iterations to result a good detection in terms of accuracy. Then, the algorithm couldn't be robust enough to be used in a real time target searching. This problem won't exist if we can keep a fixed distance between our target searching camera and the visual target, since the target size on camera frame will keep the same as well in this situation which could prevent inefficient multiple iterations. But this ideally assumption is actually impossible during the competition. There are errors in the sensors of robot which makes it impossible to control a robot perfectly keeping a certain distance to the wall for all times. In order to overcome those problems and maintain a reasonable accuracy at the same time, we decided to add a ultrasonic range sensor. This sensor will provide the distance information to the template matching algorithm, and the algorithm could calculate the proportional size of target template on the camera frame base on this distance data. Then, the template matching algorithm won't need a lot of iterations to achieve a reasonable high accuracy since the size of target template on a frame becomes known. Besides, this solution could works adaptively for different distance between camera and target without any affects to the accuracy. It should turns out to be an robust solution for target searching.

Also, a finite state machine will be implemented by us as main control of the robot. This main control will subscribe all the topics from the four separated components and make decisions and/or sending command to the robot based on data that receives from components. Four modes or states be defined in the main control that will execute in a circle are Navigation and searching, Found, Docking, Undocking. There is also a localization state, but it will only execute once at the beginning and translate to Navigation and searching state once it done. We would like to prove that this kind of state machine is suit to be used in the competition like this one.

# Materials
We used two cameras, one from a PremeSence RGB-D sensor (Asus Xtion Pro) and the other camera is a Logitech HD Webcam C270, on our egg-hunting bot. Those sensor can provide us RGB image. The PremeSence RGB-D sensor can also provide simulated Lidar scan range data. The field of view from camera of our PremeSence RGB-D sensor is 58 degree horizontally and 45 degree vertically according to [the product document](https://www.asus.com/3D-Sensor/Xtion_PRO_LIVE/specifications/). The field of view of Logitech HD Webcam C270 is 60 degree horizontally according to [
Logitech Support](http://support.logitech.com/en_us/article/17556).

The PremeSence RGB-D sensor be used as our front view camera for navigation and docking. The Logitech HD Webcam C270 be used as a side view camera for target searching. We mounted our PremeSence RGB-D sensor at the rare of our turtlebbot 2 with roughly 25 cm distance to the ground and facing to the front direction of robot which is parallel to the ground. By doing this, the simulated Lidar scan data will be useful for navigation and it could also minimize the blind area of depth data ahead of robot when docking. The Logitech HD Webcam C270 be mounted on the lowest disk board of turtlebot 2, at the right side with about 16 cm to the ground, facing the right direction of the robot which is parallel to the ground.

An ultrasonic sensor ([HY-SRF05](https://www.robot-electronics.co.uk/htm/srf05tech.htm)) have been attached to work with the Logitech HD Webcam C270 camera. It could provides range data and passing those data to computer through the serial port on an Ardurino Uno that connected to the computer. The ultrasonic sensor been mounted under the lowest disk board of turtlebot 2, where just under the Logitech HD Webcam C270 camera and facing the same direction as Logitech HD Webcam C270 camera facing to. It will improve efficiency and accuracy of template matching during the target searching.

Our turtlebot 2 is distributed by Clearpath Robotics. Referred to the [Kobuki User Guide](https://www.google.ca/url?sa=t&rct=j&q=&esrc=s&source=web&cd=9&ved=0ahUKEwjvhqLH1v_RAhVOw2MKHYAFAY4QFghCMAg&url=https%3A%2F%2Fdocs.google.com%2Fdocument%2Fexport%3Fformat%3Dpdf%26id%3D15k7UBnYY_GPmKzQCjzRGCW-4dIP7zl_R_7tWPLM0zKI&usg=AFQjCNFo0O5d312q_k2JDorv5Q0cIMiZ7A&bvm=bv.146094739,d.cGc&cad=rja), the maximum linear speed of our turtlebot is 0.7 m/s, and the maximum angular speed is 180 degree/s (about 3.14 radiance/s).

# Method/Procedures/Approach
As a big picture of our program designed, it should contains the following modules or components:
- Localization
- Navigation
- Target detector
- Docking controller
- Main controller

We planned to start with the first four separated components since most of their implementations are available online or is completed in prior course demonstration and/or competition. For example, the pose detection for docking and the navigation by Gmapping and AMCL.

Localization details, parameters, etc.!!!!!!!!!!!!!!!!!!!

Navigation parameters, etc.!!!!!!!!!!!!!!!!!!!!!!!!

For docking procedure, we will need to obtain a pose of target when the robot is getting close enough to it. Then, a docking path will be calculated based on this pose data. We used two algorithms for two different visual target pose detection, one for AR Tag and another for University of Alberta emblem. 

For AR Tag detection, we created a template instead of directly using ar_track_alvar package because we found two problems of the package. One is it cannot detect AR Tags when Turtlebot moves too fast. The other one is it is too sensitive when AR Tag is far and edge-on, which affects navigation a lot. Therefore, after we using AR Tag template, Turtlebot can find AR Tags and do pre-docking at a desired position with side camera. Then the front camera will get a good pose with [ar_track_alvar](http://wiki.ros.org/ar_track_alvar) package and dock precisely.

And for the University of Alberta emblem, we implemented a features matching algorithms (Fast key point detection and ORB descriptor) to match key points between the model emblem and an image frame. Once we got those matched key points, we calculate the pose data by using [solvePnP](http://docs.opencv.org/3.0-beta/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html#solvepnp) function in openCV.

Once those individual components are working, a main controller has been used to receive data from components and make final decisions based on those data like a general in an army. The decisions it makes usually are state translating decisions.

Altogether, our structure of program or system is shown in Graph 4.  

# Results

# Analysis/Discussion

# Conclusions
