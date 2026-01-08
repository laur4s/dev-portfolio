# IMPL Assignment Drawing: Drawing a Circle

This assignment is about drawing a circle or an arbitrary .svg file on a whiteboard. A Universal Robots UR10e robot with a 3D printed Marker holder as a tool was used.

![setup.jpeg](setup.jpeg)

It is possible to select a custom .svg file, change its size by setting the size and specify the location on the drawing board where the image should be drawn.

Furthermore we assured that the image cannot be drawn outside the drawing board and the robot keeps a safety distance to the border of the drawing board which can also be set manually.

### Getting started:  
1. Robot configuration  
    - Specify the tool center point on the robot
    - Generate a plane corresponding to the drawing board on the robot with the 3 point method (The origin is in the top left corner, the x-axis is pointing down, y-axis to the right, and z-axis out of the drawing plane)

2. RoboDK environment
    - Install RoboDK (version >= 5.9.4)
    - load the example "Drawing with ABB IRB 4600-20/2.50"
    - change the ABB robot to the UR10e robot
    - add the marker tool to the robot (SpringMarkerToolAssy.stl)
    - save the TCP and the plane (from step 1) in the RoboDK station
    - replace the python file with the draw.py in this repo


3. Code settings
    - In the setting section of the python file specify the wanted variables (all units in mm)
    - When given a path to the **IMAGE_FILE**, this svg is used. Leave this empty ("") to prompt a file explorer
    - Define **BOARD_WIDTH** and  **BOARD_HEIGHT** according to the real board
    - The variable **SIZE** defines the radius of the circle and max(width, height)/2 of an arbitrary .svg on the real world whiteboard
    - **SAVE_BOARDER** is the safety distance between the board border and the drawing
    - Set **CENTER_IMAGE** to True for drawing the image at the center of the drawing frame. By setting it to False the position is customly defined by **IMAGE_POSITION_WIDTH** and **IMAGE_POSITION_HEIGHT** (frame is defined in step 1). A separate check ensures that the complete image lies within the drawing area.


4. Running
    - In RoboDK create a robot program of the python file by right-clicking on the python file in the RoboDK file-tree. This is only possible if we include the following code line after initializing and specifying the robot:
    ```RDK.setRunMode(robolink.RUNMODE_MAKE_ROBOTPROG)```
    By setting **GENERATE_ROBOT_PROGRAM** to True in the code settings this is done automatically.
    - Now a .script and .urp file are created
    - Import the .urp on the robot and run the program  


An example of the robot drawing can be seen in [this video](https://youtu.be/z7hvwAJxzuY) and [in this video](https://www.youtube.com/watch?v=bLJ4T0XL7Vg).
Adjusting the pentip decreast overall drawing quality as can be seen in these two videos.  [this video](https://www.youtube.com/watch?v=UC3RvJ7HVww) and  [this video](https://www.youtube.com/watch?v=6Q-JkP-9h6Y)

   
### Happy drawing!  


------
Authors:  
Laura Schiller (03797770),  
Raimund Lau (03806149),  
Maximilian Reif (03671189)
