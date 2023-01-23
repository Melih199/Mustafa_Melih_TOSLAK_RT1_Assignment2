# Mustafa_Melih_TOSLAK_RT1_Assignment2
RESEARCH TRACK 1  ASSIGNMENT 2
================================

## Goal Of The Assignment

This assignment involves creating a package for a robot simulation in Gazebo and Rviz. 
The aim of this assignment is to create a new ROS package which we will develope 3 nodes: 

1. A node that implements an action client, allowing the user to **set a target (x, y) or to cancel it**. The node
also **publishes the robot position and velocity** as a custom message (x,y, vel_x, vel_y), by relying on the values
published on the topic /odom.
2. A service node that, when called, prints the number of goals reached and cancelled.
3. A node that subscribes to the robot’s position and velocity (using the custom message) and prints the
**distance of the robot from the target and the robot’s average speed**. Use a parameter to set how fast the
node publishes the information.
4. Also create a **launch file** to start the whole simulation. Set the value for the **frequency** with which node (c) publishes
the information.

<p align="center" width="100%">
    <img width="490" height="350" src="https://user-images.githubusercontent.com/58879182/213936088-b599162b-4c8a-4728-b4f6-830d56a3db6e.png">
    <img width="490" height="350" src="https://user-images.githubusercontent.com/58879182/213935894-04b775d8-8a03-4a45-86b4-349905741c48.png">
    
</p>

---------------------------------
## First Node: Action (action_user.py)

The first node of our package creates a publisher "pub" that publishes a custom message "**Posxy_velxy**" on the topic "/posxy_velxy". The custom message contains four fields "**msg_pos_x**", "**msg_pos_y**", "**msg_vel_x**", "**msg_vel_y**" that represent the position and velocity of the robot.

```python
 def publisher(msg):
    global pub
    # get the position information111
    pos = msg.pose.pose.position
    # get the velocity information
    velocity = msg.twist.twist.linear
    # custom message
    posxy_velxy = Posxy_velxy()
    # assign the parameters of the custom message
    posxy_velxy.msg_pos_x = pos.x
    posxy_velxy.msg_pos_y = pos.y
    posxy_velxy.msg_vel_x = velocity.x
    posxy_velxy.msg_vel_y = velocity.y
    # publish the custom message
    pub.publish(posxy_velxy)
```
The node also creates a subscriber "sub_from_Odom" that subscribes to the topic "/odom", which publishes the Odometry message. The callback function "publisher" is called every time a message is received on the topic "/odom". This function extracts the position and velocity data from the Odometry message and creates an instance of the custom message. The function then assigns the position and velocity data to the corresponding fields of the custom message and publishes the message on the topic "/posxy_velxy".

<p align="center" width="100%">
    <img width="800" height="250" src="https://user-images.githubusercontent.com/58879182/213940945-5b4c75b8-79c5-45ce-9602-caa3081905f1.png">
</p>



Finally the "action_client()" funtion creates an action client and waits for the action server "/reaching_goal" to start. It enters a while loop that prompts the user to enter the target position or type "c" to cancel the goal. If the user enters "c", the action client cancels the goal and sets the status_goal to false. If the user inputs a target position, the function converts the inputs from strings to floats, creates a goal with the target position and sends it to the action server(Planning.action). It also sets status_goal to true.
It's a simple implementation of action client, it sends a goal to the action server and waits for the result of the goal, it could be an error, a success, or a cancelation. The user can interact with the client, setting a goal or canceling it.

<p align="center" width="100%">
    <img width="60%" src="https://user-images.githubusercontent.com/58879182/213941409-7911d914-4ef2-48ae-b2bb-a1432ce44d4f.png">
</p>





--------------------------------------------------------------------------------------------------------------------------------------------------
## Second Node: Service (goal_service.py)

The second node creates a ROS service that listens for requests on the "goal_service" topic, and responds with the number of goals reached and cancelled. It also subscribes to the "/reaching_goal/result" topic to receive messages about the status of goals and updates the counters for goals reached and cancelled accordingly. When the service is called, it returns a goal_rcResponse message containing the current values of goal_reached and goal_cancelled.

<p align="center" width="100%">
    <img width="800" height="250" src="https://user-images.githubusercontent.com/58879182/213945125-df4fc75e-a79e-40b6-813d-e3963bbc4f50.png">
</p>

It initializes a ROS node called "goal_service" and creates an instance of the Service class. This creates the service, which listens for requests on the "goal_service" topic, and a subscriber to the "/reaching_goal/result" topic. When a request is received on the "goal_service" topic, the data method is called, which returns a goal_rcResponse message containing the current values of goal_reached and goal_cancelled.

When a message is received on the "/reaching_goal/result" topic, the result_callback method is called. This method examines the status (when robot moving: status = 1, when robot target cancelled: status = 2 and when robot reached the target: status = 3) of the goal, which is contained within the message, and increments the appropriate counter, either goal_cancelled or goal_reached. To check the status "rostopic echo /reaching_goal/status" can be run.

<p align="center" width="100%">
    <img width="32%" src="https://user-images.githubusercontent.com/58879182/213946558-6baa0529-c805-478d-bbfa-5d8cf2a23401.png">
    <img width="32%" src="https://user-images.githubusercontent.com/58879182/213946559-fb0281e6-65eb-4569-b5b9-347fece81313.png">
    <img width="32%" src="https://user-images.githubusercontent.com/58879182/213946570-c6f54c7b-8104-4759-9046-0e861b1b48c9.png">
</p>

----------------------------------------------------------------------------------

## Third Node: Print Distance and Average Velociity (print_dis_avgvel.py)

The third node prints out information about a robot's distace from target and average velocity. The node gets the publish frequency parameter from ROS parameters, which is used to determine how often the information is printed. It also initializes a variable to keep track of the last time the information was printed and creates a subscriber to the '/posxy_velxy' topic, which i  to containining messages of robot's curren x,y positions and x,y velocities.

```python
 def __init__(self):
        # Get the publish frequency parameter
        self.freq = rospy.get_param("frequency")

        # Last time the info was printed
        self.printed = 0

        # Subscriber to the position and velocity topic
        self.sub_pos = rospy.Subscriber("/posxy_velxy", Posxy_velxy, self.posvel_callback)
```
The node first gets the desired position of the robot, and the actual position of the robot from the message received. It then calculates the distance between the desired and actual positions using the math.dist() function. It also gets the actual velocity of the robot from the message and calculates the average speed using the velocity components from the message. Finally, it prints the distance and average speed information using the rospy.loginfo() function, and updates the last printed time variable.

<p align="center" width="100%">
    <img width="60%" src="https://user-images.githubusercontent.com/58879182/213949410-960707c9-6672-490f-96c1-2d3c2618f1cd.png">
</p>


-------------------------------------
## Creating Launch file (assignment2.launch)

The ROS launch file is used to start multiple nodes and set parameters at once. The launch file is written in XML and uses the <launch> tag as the root element.The launch file starts by including another launch file, "sim_w1.launch", which is already located in our package to run Gazebo and Rviz simulators and environment related nodes. Then, it sets two parameters "des_pos_x" and "des_pos_y" with values 0.0 and 1.0 respectively. These parameters used by other nodes to determine the desired position of the robot.

Then we set a parameter "frequency" with a value of 1.0. This parameter used by the node "print_dis_avgvel.py" to determine how often the distance and average velocity information should be printed.

After that, it starts nodes using the <node> tag, these nodes are:

  +  "wall_follower.py"
  +  "go_to_point.py"
  +  "bug_action_service.py"
  +  "action_user.py"
  +  "goal_service.py"
  +  "print_dis_avgvel.py"

Each of these nodes is defined by specifying the package name "assignment_2_2022" where they reside, the type of the file, and the name of the node. The last two nodes are run with the additional parameter output="screen" and launch-prefix="xterm -hold -e" respectively, which causes the output of these nodes to be printed to the screen in a new terminal window.
	

	
```xml
 def __init__(self):
 <?xml version="1.0"?>
<launch>
    <include file="$(find assignment_2_2022)/launch/sim_w1.launch" />
    <param name="des_pos_x" value= "0.0" />
    <param name="des_pos_y" value= "1.0" />
    
    <!--Frequency parameter to set the frequency of the print_dis_avgvel node -->
    <param name="frequency" type="double" value="1.0" />
    
    <node pkg="assignment_2_2022" type="wall_follow_service.py" name="wall_follower" />
    <node pkg="assignment_2_2022" type="go_to_point_service.py" name="go_to_point"  />
    <node pkg="assignment_2_2022" type="bug_as.py" name="bug_action_service" output="screen" />
    <node pkg="assignment_2_2022" type="action_user.py" name="action_user" output="screen" launch-prefix="xterm -hold -e" />
    <node pkg="assignment_2_2022" type="goal_service.py" name="goal_service"  />
    <node pkg="assignment_2_2022" type="print_dis_avgvel.py" name="print_dis_avgvel" output="screen" launch-prefix="xterm -hold -e" />
</launch>
```
This launch file allows to start all the necessary nodes for the application and set the required parameters with a single command, instead of running each node and setting each parameter separately. It also allows to run the nodes in a specific order and with specific settings.
	
------------------------------------
## Installation

First of all before running the program it is required to install the xterm libray. Open a terminal window and run the following command to install the xterm package, this library helps us to print outputs of the nodes in a new terminal window :

```command
	sudo apt-get install xterm -y
```
Next, navigate to your ROS workspace 'src' folder and clone this repository using the following command:
	
```command
	git clone <link of the repository>
```
Once the repository has been cloned, navigate to the work space drectory and run the following command to build the package:

```command
	catkin_make
```


After the package has been built successfully, finally, we can launch the simulation
	
---------------------------------

## How To Run The Simulation
The launch file for the assignment can be found in the "launch" folder within the "assignment_2_2022" directory. To start the simulation, use the following command: 

```command
	roslaunch assignment_2_2022 assignment2.launch
```
Upon successful launch, four screens should appear: one for inputting target coordinates (action_user.py), one for displaying the distance and average velocity of the robot (print_dis_avgvel.py), and two for the Gazebo and Rviz visualization environments.
	
<p align="center" width="100%">
    <img width="24%" height="200" src="https://user-images.githubusercontent.com/58879182/213941409-7911d914-4ef2-48ae-b2bb-a1432ce44d4f.png">
    <img width="24%" height="200" src="https://user-images.githubusercontent.com/58879182/213949410-960707c9-6672-490f-96c1-2d3c2618f1cd.png">
    <img width="24%" height="200" src="https://user-images.githubusercontent.com/58879182/213936088-b599162b-4c8a-4728-b4f6-830d56a3db6e.png">
    <img width="24%" height="200" src="https://user-images.githubusercontent.com/58879182/213935894-04b775d8-8a03-4a45-86b4-349905741c48.png">	
</p>

To run the service node (goal_service.py) that, when called, prints the number of goals reached and cancelled use the following command:

```command
	rosservice call /gaol_service
```
or just us the rqt tool to call the service. rqt is a tool in ROS (Robot Operating System) that provides a simple and intuitive GUI (graphical user interface) for debugging and analyzing various aspects of the system. It allows users to view various data streams, such as topics, services, and parameters, as well as perform tasks such as plotting, logging, and debugging. rqt also provides a plugin system which allows developers to create custom plugins for specific tasks. It helps to debug, visualize and inspect the ROS system, also to monitor and control the ROS nodes and topics.

```command
	rqt
```
<p align="center" width="100%">
    <img width="60%" src="https://user-images.githubusercontent.com/58879182/214059310-a25e8d3d-29fd-4a1f-927f-9e372578cba3.png">
</p>

<p align="center" width="100%">
    <img width="60%" src="https://user-images.githubusercontent.com/58879182/214059310-a25e8d3d-29fd-4a1f-927f-9e372578cba3.png">
</p>
	
---------------------------------

## Troubleshooting

When running `python run.py <file>`, you may be presented with an error: `ImportError: No module named 'robot'`. This may be due to a conflict between sr.tools and sr.robot. To resolve, symlink simulator/sr/robot to the location of sr.tools.

On Ubuntu, this can be accomplished by:
* Find the location of srtools: `pip show sr.tools`
* Get the location. In my case this was `/usr/local/lib/python2.7/dist-packages`
* Create symlink: `ln -s path/to/simulator/sr/robot /usr/local/lib/python2.7/dist-packages/sr/`

-----------------------------------

## Not Autonomous Solution

In this solution the path of the robot already given with two array: 

* silver_tokens_offset_number = [3,2,1,0,5,4]
* golden_tokens_offset_number = [11,10,9,8,7,6]

First of all robot will search and find the silver token which offset number is equal to the first element of the **silver_tokens_offset_number** array, then it will grab it. When graping is done, robot will search for golden token token which offset number is equal to the first element of the **golden_tokens_offset_number** array and it will release the silver token near to golden token and the process will continue with the other elements of the both arrays, respectively until all the elements of arrays are processed.

Note: The issue of this solution is you have to find the offset numbers of tokens and write in the arrays **manually**. 

![Not_Autonomous_Path](https://user-images.githubusercontent.com/58879182/202065266-b522196a-5574-4ed6-9bd9-49be95bafaf7.png)



-----------------------------------

## Semi Autonomous Solution

The Semi autonomous solution can be thought of as upgrading Non autonomous solution to become more autonomous. The issue of manually entering offset numbers to the arrays found the solution here. Now robot capable of:

* Finding closest silver token and grabing it.
* Finding closest golden token and releasing silver token near to golden token.
* Not using already taken silver and golden tokens.
* Ending task when all **12** tokens are used.

**Note:** Although the robot's autonomous level increases with this solution, the user must specify the total token number **manually** in the code.

```python
 if len(Taken_tokens)==12:                       

    		print("Well done assignment completed successfully")

    		exit()
```
------------------------------------------

## Full Autonomous Solution

In this solution "manually entering" issues from Not autonomous and semi autonomous solutions are solved. After using all tokens our robot turning 360 degree to check if there is any silver token left if not ending program. Now our robot capable of:

* Finding closest silver token and grabing it.
* Finding closest golden token and releasing silver token near to golden token.
* Not using already taken silver and golden tokens.
* Ending task when there is no silver token left.

```python

  if offset in Taken_tokens:                    

		turn(+40, 0.1)                       
  		print("Looking for not already taken Silver Token")
		complete = complete + 1  
		
  		if complete == 24:    
				print("I turned 360 degree and couldn`t see any not taken  silver Token")
				print("MISSION COMPLETE")
				exit()        # And the System because there is no any silver token left
 ```
**Note:** When integer variable coplete is equal to 24 this mean our Robot turned 360 degree. Turn(40,0.1) function is turning our robot 15 degree (15*24=360).

### Flowchart Of The Full Autonomous Solution

![Full_autonomous_Flowchart_basic](https://user-images.githubusercontent.com/58879182/202076049-4c5f786d-598a-4f5f-8d76-94f5d5713e7a.png)

---------------------------------------

### Functionalities And Informations About Robot API

The API for controlling a simulated robot is designed to be as similar as possible to the [SR API][sr-api].

### Motors ###

The simulated robot has two motors configured for skid steering, connected to a two-output [Motor Board](https://studentrobotics.org/docs/kit/motor_board). The left motor is connected to output `0` and the right motor to output `1`.

The Motor Board API is identical to [that of the SR API](https://studentrobotics.org/docs/programming/sr/motors/), except that motor boards cannot be addressed by serial number. So, to turn on the spot at one quarter of full power, one might write the following:

```python
R.motors[0].m0.power = 25
R.motors[0].m1.power = -25

```
### The Grabber ###

The robot is equipped with a grabber, capable of picking up a token which is in front of the robot and within 0.4 metres of the robot's centre. To pick up a token, call the `R.grab` method:

```python
success = R.grab()
```
The `R.grab` function returns `True` if a token was successfully picked up, or `False` otherwise. If the robot is already holding a token, it will throw an `AlreadyHoldingSomethingException`.

To drop the token, call the `R.release` method.

Cable-tie flails are not implemented.

### Vision ###

To help the robot find tokens and navigate, each token has markers stuck to it, as does each wall. The `R.see` method returns a list of all the markers the robot can see, as `Marker` objects. The robot can only see markers which it is facing towards.

Each `Marker` object has the following attributes:

* `info`: a `MarkerInfo` object describing the marker itself. Has the following attributes:

  * `code`: the numeric code of the marker.

  * `marker_type`: the type of object the marker is attached to (either `MARKER_TOKEN_GOLD`, `MARKER_TOKEN_SILVER` or `MARKER_ARENA`).

  * `offset`: offset of the numeric code of the marker from the lowest numbered marker of its type. For example, token number 3 has the code 43, but offset 3.

  * `size`: the size that the marker would be in the real game, for compatibility with the SR API.

* `centre`: the location of the marker in polar coordinates, as a `PolarCoord` object. Has the following attributes:

  * `length`: the distance from the centre of the robot to the object (in metres).

  * `rot_y`: rotation about the Y axis in degrees.

* `dist`: an alias for `centre.length`

* `res`: the value of the `res` parameter of `R.see`, for compatibility with the SR API.

* `rot_y`: an alias for `centre.rot_y`

* `timestamp`: the time at which the marker was seen (when `R.see` was called).

For example, the following code lists all of the markers the robot can see:

```python
markers = R.see()
print "I can see", len(markers), "markers:"
for m in markers:
    if m.info.marker_type in (MARKER_TOKEN_GOLD, MARKER_TOKEN_SILVER):
        print " - Token {0} is {1} metres away".format( m.info.offset, m.dist )
    elif m.info.marker_type == MARKER_ARENA:
        print " - Arena marker {0} is {1} metres away".format( m.info.offset, m.dist )
```
[sr-api]: https://studentrobotics.org/docs/programming/sr/

## Conclusion

As a result our robot fully capable of finding silver and golden tokens and successfully completing the assigned task no matter how many tokens. The robot can be improved by the following methods:

* Increasing the robot's field of view
* Avoiding other tokens

**Note:** Now our robot fully autonomous but if the number of the silver and golden tokens are not equal and the number of the silver tokens is more than golden tokens number our program can be crash.
