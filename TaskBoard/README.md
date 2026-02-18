# IMPL Task-Board Manipulation

This assignment is about developing a program for a Franka FR3 robot equipped with an Intel Realsense D435i camera to solve the electronic task board trial protocol. 

### Picture of the task board:  

![board setup](./TaskBoard/task-board.jpeg)

### Challenge description:  
**Board detection:** First detect the task board inside the camera frame.   
**Speed test:** The robot has to press the blue button, then release it and as fast as possible press the red button.  
**Light test:** After release of the red button, one arbitrary button (either red or blue) is illuminated and the robot has to press it.

<br></br>

### Getting started: 
1. Code source:
    - This project uses [ROS2 jazzy](https://docs.ros.org/en/jazzy/index.html). It is only tested with jazzy, using humble might cause some errors that need to be fixed individually
    - We cloned the repository from the [Franka Github](https://github.com/frankarobotics/franka_ros2) with the commit **#7ef0ab0**. Also make sure to have libfranka installed.
    - This project is only working in **simulation** and was not tested on the real robot yet.

    Here the whole setup and also the starting configuration of the robot can be seen:  
    ![board general setup](./TaskBoard/setup-task-board.jpeg)


2. Preparation
    - Setup the environment by sourcing following file: ```source /opt/ros/jazzy/setup.bash```
    - Navigate into the workspace folder where your franka_ros2 is installed
    - Clone this repository into a new folder inside your src where also the franka_ros2 folder is located
    - Build the workspace (outside your src folder): ```colcon build --symlink-install``` (later only ```colcon build --packages-select franka_bringup taskboard_exercise``` is sufficient)
    - In each terminal:
        - Source the setup.bash from the install folder: ```source install/setup.bash``` 
        - Change the RMW implementation and use FastRTPS intead of CycloneDDS: ```export RMW_IMPLEMENTATION=rmw_fastrtps_cpp```    


3. Task execution:
    - Terminal 1:
        Launch the gazebo simulation and the moveit script:
        ```ros2 launch taskboard_exercise complete_gazebo_moveit_control.launch.py```

    - Terminal 2:
        Launch the vision components that mimic a real camera and button/board detection  
        as well as the position control action:
        ```ros2 launch taskboard_exercise complete_mock_taskboard_exercise.launch.py```

    - **Some verification tests:**  
        - In rviz add the topic camera/color/image_raw/Image for the "mock-live" rgb_image of the taskboard
        and the detection_debug_image/Image topic to see the detected buttons and taskboard  
        - Open ```rqt_graph``` in another terminal and verify whether all topics and nodes exist and are connected properly   
        - Open ```ros2 run rqt_tf_tree rqt_tf_tree``` and verify whether there is an existing (and connected) transformation from the base frame of the robot to the camera_link (otherwise make naming adjustments in the coordinate_extraction_ros and complete_mock_taskboard_exercise)

    - Terminal 3:
        Start the service call in order to move the robot in the simulation: 
        ```ros2 service call /toggle_taskboard_logic std_srvs/srv/SetBool "{data: true}"```


### Completed Tasks
**Board Detection**
The camera is able to detect the red and blue button of the taskboard and publishes the coordinates on two different topics. In rviz the detections as well as the taskboard bounding box can be visualized: 

![board detections](./TaskBoard/taskboard_with_buttons.jpeg)

**Speed Test**
Also the robot controller part is already existing and implemented with moveit. It can be called via a service which then starts the movement of the robot in simulation. Therefore we can ensure that all dependencies and topics are loaded/subscribed/published correctly before even moving the robot.

However, this part still needs to be tested on the actual robot. Probably adjustments need to be made because the robot is controlled via effort and the PID values are so far only tuned for the robot in simulation.

**Light Test**
The logic for the light test is also already implemented. The system detects glowing buttons and non-glowing buttons. For the glowing buttons, we used the color values from the team Atlabotics, because the taskboard in the lab is not starting. So changes to these values may be needed for our board or environment. To detect which button glows we developed a robust implemetation:
There are two variables (red and blue). For every detection of buttons they are changed like in the table below. After that the variable with the higher value is used for the "glowing status". If they are the same, the robot waits and checks the lights again. This makes it possible to detect the status of the buttons in the light test more robust.

| Found button     | changes for red | changes for blue |
|------------------|-----------------|------------------|
| Red              | -1              | /                |
| Not Red          | +1              | /                |
| Red Glowing      | +1              | /                |
| Not Red Glowing  | -1              | /                |
| Blue             | /               | -1               |
| Not Blue         | /               | +1               |
| Blue Glowing     | /               | +1               |
| Not Blue Glowing | /               | -1               |

Examples:
- The red button is glowing and the blue button is not glowing.
    - The red variable is +1, the blue variable -1.
    - Robot detects red button glowing.

- The red button is glowing and the blue button is not glowing, but not detected at all.
    -  The red variable is +1, the blue variable is 0, because "Not Blue" and "Not Blue Glowing".
    - Robot detects red button glowing.

- Both buttons are not glowing and detected.
    - Both variables have the value -2, because e.g. "Red" and "Not Glowing Red"
    - Both values are same -> robot waits


Here you see a video of the robot motion in simulation:
<video src="./TaskBoard/taskboard-exercise.mp4" width="600" controls>
  Your browser does not support the video tag.
</video>

![board exercise](./TaskBoard/taskboard-exercise.mp4)

[![Watch the video](./TaskBoard/task-board.jpeg)](./TaskBoard/taskboard-exercise.mp4)

<br></br>

### Issues that still need to be addressed in the future
Improvements in simulation:  
- **Inlcude gripper control**: The gripper of the robot in simulation is currently not controlled and therefore shakes a little bit and also opens and closes randomly during the movement.  


Testing on real hardware:
- **Robot motion discontinuity error:** The real robot exceeded some acceleration limits in real time, therefore an even smoother function than just linear interpolation between the start and target position has to be used. 
    We therefore focused on implementing the task in simulation as time was running out and we no longer had access to the hardware.
    - **Note:** This error might be fixed now due to the usage of *moveit* to control the robot. Eventually we have to add an impedance control part that the real robot is not fully stiff and to prevent crashes into the taskboard. As mentioned previouly also the PID values for controlling the robot need to be tuned for the real robot and it might be that the values arrive at the robot with too low of a frequency.
- **Movement towards the buttons:** For pressing the buttons fast and safe it might be helpful to include two different kinds of movements. First, the robot moves fast to a point a few centimeters above the button and then move slower to the button and press it. This enables a faster movement for long movements (starting point to the taskboard) and a safer and more reliable movement for the region nerarby the board.
- **Light Test Robustness:**  We can only verify whether the Glow detection logic is robust or needs to be changed when the taskboard is working.


<br></br>

------
Authors:  
Laura