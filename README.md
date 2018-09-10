# Map My World Robot Project Starter Code

The objective of this project is to create a 2D occupancy grid and 3D octomap from a provided simulated environment. In addition,, a simulated environment will be created to map as well.

<!--more-->

[//]: # (Image References)

[image1]: ./misc_images/bot_slam.jpg "Custom Bot"
[image2]: ./misc_images/frames.jpg "Robot TF Frames"
[image3]: ./misc_images/kitchen_scene.jpg "Kitchen World Scene"
[image4]: ./misc_images/custom_scene.jpg "Custom World Scene"
[image5]: ./misc_images/kitchen_slam.jpg "SLAM Mapped"
[image6]: ./misc_images/kitchen_rtabmap.jpg "Rtabmap"
[image7]: ./misc_images/custom_slam.jpg "Custom SLAM"
[image8]: ./misc_images/custom_rtabmap.jpg "Rtabmap"

---


#### How to run the program with your own code

Go to Desktop and open a terminal.

For the execution of your own code, we head to the Project Workspace. For this setup, catkin_ws is the name of the active ROS workspace. If your workspace name is different, change the commands accordingly.

If you do not have an active ROS workspace, you can create one. You can launch it by running the following commands first.
```bash
  cd /home/workspace
  mkdir -p catkin_ws/src
  cd /home/workspace/catkin_ws
  catkin_make
  source devel/setup.bas
```

Clone the required repositories to the `/home/workspace/catkin_ws/src` folder
```bash
cd /home/workspace/catkin_ws/src
git clone https://github.com/Abhaycl/RoboND-Map-My-World-Robot-2P3.git
```

Now install missing dependencies using rosdep install:
```bash
rosdep install -i slam_bot
```

Build the project:
```bash
cd /home/workspace/catkin_ws
catkin_make
```

Download the proper kitchen dining room `*.world` file that includes collision models required to map properly:
```bash
curl -L https://s3-us-west-1.amazonaws.com/udacity-robotics/Term+2+Resources/P3+Resources/models.tar.gz | tar zx -C ~/.gazebo/
```

Open three terminals to the `/home/workspace/catkin_ws/src/slam_bot/launch` folder and run each command in one terminal:
```bash
roslaunch slam_bot world.launch world_name:=kitchen_dining.world
roslaunch slam_bot teleop.launch
roslaunch slam_bot mapping.launch
```
Dependencias del sistema que se encuentran con las convenciones de CMake

---

The summary of the files and folders within repo is provided in the table below:

| File/Folder               | Definition                                                                                                   |
| :------------------------ | :----------------------------------------------------------------------------------------------------------- |
| config/*                  | Folder that contains all the parameters and some values defined for you to help you get started rviz.        |
| db/*                      | Folder that contains all the databases generated from the maps.                                              |
| launch/*                  | Folder that contains all the launch files in ROS that allow us to execute more than one node simultaneously. |
| meshes/*                  | Folder that contains all the parameterization of the sensors.                                                |
| scrips/*                  | Folder that contains all the scripts for the execution of the different necessary tools.                     |
| urdf/*                    | Folder that contains all the robot's URDF description.                                                       |
| worlds/*                  | Folder that contains all the Gazebo worlds.                                                                  |
| misc_images/*             | Folder containing the images of the project.                                                                 |
|                           |                                                                                                              |
| CMakeLists.txt            | Contains the System dependencies that are found with CMake's conventions.                                    |
| frames.gv                 | Contains the project documentation.                                                                          |
| frames.pdf                | Contains the project documentation.                                                                          |
| package.xml               | Contains the slam_bot package.                                                                               |
| README.md                 | Contains the project documentation.                                                                          |
| README.pdf                | Contains the project documentation in PDF format.                                                            |

---

**Steps to complete the project:**  

1. You will develop your own package to interface with the rtabmap_ros package.
2. You will build upon your localization project to make the necessary changes to interface the robot with RTAB-Map. An example of this is the addition of an RGB-D camera.
3. You will ensure that all files are in the appropriate places, all links are properly connected, naming is properly setup and topics are correctly mapped. Furthermore you will need to generate the appropriate launch files to launch the robot and map its surrounding environment.
4. When your robot is launched you will teleop around the room to generate a proper map of the environment. 
5. You then will build your own simulated environment and apply the code you used to map the supplied environment on this environment.
6. Finally you will create a report and organize the required files for submission. 


## [Rubric Points](https://review.udacity.com/#!/rubrics/1441/view)
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
## Abstract

This project aims to successfully perform SLAM leverage GraphSLAM to generate 2D occupancy grids and 3D octomaps of two simulated environments in Gazebo. A two-wheel robot equipped with RGB-D and 2D Lidar sensors (configured as a camera and a Hokuyo lidar, respectively) was simulated to traverse two Gazebo world environments, one provided in this practice, and another that was custom built. The robot maps the environment using the "Real Time Appearance Based Mapping" (RTAB-Map) GraphSLAM approach. After the robot traversed both environments, maps were evaluated on accuracy.


## Introduction

For a robot to be fully autonomous it must adapt and understand its environment as it changes and be able to recover when it obtains noisy data from sensors. Simultaneous Localization and Mapping (SLAM) provides a robot with the capability to dynamically map its environment as it relates to it. When implemented correctly, SLAM solves the localization problem to map the environment, and maps the environment to solve the localization problem. This is all done as an iterative cycle.

In this project, two 3D world environments are to be simulated, tested, and mapped using SLAM. The environment provided, and set to be the benchmark, is called kitchen dining. Another environment shall be designed and developed in Gazebo and saved in world format; this one is called my world. The two environments shall be mapped in both 2D and 3D by using Gazebo and Rviz simulation frameworks.


## Background / Formulation

Given that a map is needed for localization, but at the same time the robots location and orientation is needed for mapping, this has often been considered the chicken or the egg problem. SLAM is a challenging problem, as it has to quickly solve two related problems simultaneously and back-to-back. There are many approaches to perform SLAM such as:

* Extended Kalman Filter SLAM (EKF)
* Sparse Extended Information Filter (SEIF)
* Extended Information Form (EIF)
* FastSLAM
* GraphSLAM

The two most commonly used approaches in robotics are FastSLAM and GraphSLAM. Both of these, solve the SLAM problem well, although in different ways.​

### FastSLAM

The FastSLAM algorithm can solve the full SLAM problem with known correspondences. Using a custom particle filter approach along with a low dimensional Extended Kalman Filter, FastSLAM is able to estimate the trajectory of the robot which will allow it to map the environment concurrently.


### GraphSLAM

GraphSLAM is a SLAM algorithm that solves the Full SLAM Problem. This algorithm recovers the entire path and map, which allows it to consider dependencies between current and previous poses. GraphSLAM has a few advantages over FastSLAM. GraphSLAM improves upon the need of onboard processing capability, while still improving accuracy over FastSLAM. Since GraphSLAM is able to retain information from past locations, it proves an advantage over FastSLAM which uses less information and has a finite number of particles.


### RTAB-Mapping

The Real Time Appearance Based Mapping algorithm is a GraphSLAM approach and will be used in this project to perform SLAM. This algorithm uses data collected from sensors to localize the robot and map the environment. In RTAB-Map a process called loop closure is used to allow the robot to determine if the location has been observed before. While the robot continues to traverse through its environment the map continues to grow. For other Appearance-Based methods, the robot continues to compare new images to past images to identify whether it has been at that location before. This, over time, increases the number of images for comparison, causing the loop closure process to take longer, proportionally increasing complexity along the way. However, RTAB-Map is optimized for large-scale and long-term SLAM, allowing loop closure to be processed fast enough to be known in real-time without dramatically affecting performance.

There are numerous challenges for SLAM algorithms to address. As with any robotics application, large amounts of noise may introduce irreparable error into the algorithm. Specifically for SLAM applications, algorithms will tend to have difficulty with large environments that have multiple areas that appear similar. These produce incorrect consolidations of the created map. Proper tuning of algorithm parameters will mitigate this issue.Additionally, some algorithms may reach memory management issues. This is more prevalent in larger environments. The required memory to accurately map the environment may become costly for the algorithm to maintain. This issue can be addressed architecturally by selecting the most suited SLAM method for the robot and environment. Environments with unique landmarks and smaller spaces are mapped more easily.


## Scene and robot configurations

### Robot Model

The robot used for this project It's new build to solve the localization problem in a previous project, which has a cylindrical shape, with two cylindrical wheels, and to spherical casters. This robot has been repurposed to perform SLAM by changing its regular RGB camera for an RGB-D sensor camera, which allows it to detect the depth of its environment. This, along with its existing 2D laser rangefinder (Hokuyo) sensor, allows the robot ROS package slambot to leverage the rtabmap-ros package to perform SLAM.

During development of the mapping project, it became apparent that the RGB-D camera frame required rotation and translation to properly re-orient the measured point cloud axes for mapping purposes. To accomplish this, a specific transform node was created in the ”robot description.launch” file.

![alt text][image1]
###### Custom Robot

Since manual mapping is to be performed the slam-bot package communicates with a teleop package that allows the user to move and steer the robot.

The backend configuration of the robot can be better appreciated in the Transform tree.

![alt text][image2]
###### Robot TF Frames


### Scene Configuration

In order for the robot to perform SLAM, it must have a World to perform it in. For this reason, two environments will be used to test and simulate a robot performing SLAM and to generate 2D and 3D maps from them.

#### Kitchen and Dining Scene

The Kitchen and Dining Scene world was provided and will be used as a benchmark for experimentation purposes in the image below.

![alt text][image3]
###### Kitchen and Dining Scene

#### My Custom World Scene

The my custom world scene was designed in Gazebo to be used as the experimental environment to deploy a robot for performing SLAM. This scene was developed with the RTAB-Map Appearance Based feature in mind. Therefore, this scene is dramatically feature-rich to help the robot easily identify where it is based on appearance, and map the environment more accurately. This scene includes a tables, boxes, closet and even floor colors that allow the robot to detect more features.

![alt text][image4]
###### Custom World Scene


## Results

### Kitchen and Dining Scene 

Upon launching the Kitchen and Dining scene simulation, and launching the mapping node, the slam-bot started mapping the environment in its immediate vicinity.

After traversing and navigating around the entire world, the slam-bot was able to generate an accurate map of the world that was provided.

As can be seen from the rtabmap database, several loop closures were observed, and bot a 2D occupancy grid map and a 3D octomap were generated for the provided environment.

![alt text][image5]
###### Kitchen slam mapping

![alt text][image6]
###### Kitchen Rtabmap


### Custom World Scene

Upon launching the my custom world scene simulation along with the mapping node the slam-bot started mapping the immediate environment.

After navigating throughout the entire environment, the slam-bot was able to generate an accurate map of the world that was developed.

As can be seen from the rtabmap database, several loop closures were observed, and bot a 2D occupancy grid map and a 3D octomap were generated for the custom developed environment.

![alt text][image7]
###### Custom slam mapping

![alt text][image8]
###### Custom Rtabmap


## Discussion

Since it was a manual navigation, traversing through the two environment was rather tedious. Furthermore, making a close approaches to obstacles threw off the mapping process at times, and was difficult for the robot to recover afterwards. Although the Kitchen and Dining scene seemed to have less noticeable features that the my custom world scene, it was less complicated and faster to map that the latter. In order to map the my custom world scene, navigating around the objects that were placed to serve as both obstacles and features had to be done cautiously. Otherwise, if the robot got too close to objects, it would delocalize the robot, throwing off the mapping process. Whereas not getting close enough, would not allow the robot to map the entire my custom world scene accurately. One thing that was observed was that the laser range-finder had a lower range than necessary for the designed environment. While the Kitchen and Dining scene was more constrained, with solid objects and obstacles, the range-finder had solid surfaces to bounce off to detect distance accurately. However, the my custom world scene has more open space which seemed to have made the mapping slightly more complicated for the robot. Nonetheless, the robot was able to successfully map both environments accurately, solving the SLAM problem in the process.

The performance of the robot’s mapping for each environment have a few similarities. Both were capable of clearly mapping 2D map boundaries, except for the unbounded section of the Kitchen Dining area. The robot was able to generate 3D maps of each world that were very recognizable of the actual environment. However, both struggled with 2D and 3D mapping the tables due to the height and angle of the camera and laser sensors.

The Kitchen Dining Area resulted in full, apparent walls while the custom world had sections missing. This is partially due to failing to scan the entire area methodically.


## Future Work

Possible future work includes developing a 4-wheeled robot equipped with the Jetson TX2 and a kinect RGBD camera sensor to map an entire real-life apartment. Possibly this would lead to building a custom Roomba robot to vacuum or somehow clean an apartment. With some improvements, this approach can be implemented in a drone for real estate aerial mapping; among other solutions that can be deployed for aerial or ground robotics.

Mapping is a very valuable feature in many robotics applications, both autonomous and controlled. This can be used in surveillance drones to generate maps of areas of interest as they scan the area. The map can be used for any sort of strategic planning for the area. An autonomous robot can use mapping to learn how to navigate around an environment to complete its goal. This would be useful for a home assistance robot that may also implement pick-nplace perception to grab items the owner may need. These are only two valuable examples where mapping would be invaluable.

A ROS package was generated and successfully performed 2D and 3D mapping using RTAB-Map. The robot was able to generate quality maps of the kitchen dining room and average maps of the custom world. However, there is still much room for improvement in this SLAM implementation. Further work is required to optimize the SLAM package to be more robust and reliable. Also, the robot may be expanded to perform more autonomous behavior. Newer configurations should be tested to attempt to address shortcomings noted in the Discussion section.