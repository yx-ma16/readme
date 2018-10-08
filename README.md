readme
======
This readme file details how to use the optimization function for the neck mechanism and also goes through the idea of the code.The instruction goes into three parts, including background information, main functon instruction and code details. To use the *optimise* function, you just need to read the first two parts of this file. If you want to change the code to achieve a better optimization or implement other functionality, the last part will be helpful to edit the code. </br>

## Background Information
The background information includes model explanation and model simplification. The part of model explanation introduces the actual neck mechanism, while the part of model simplification abstracts the actual mechanism into a geometric model. This simplification facilitates the  process of optimization.</br>

### model explanation
The neck mechanism is a 3-UPU (3-universal-prismatic-universal) parallel manipulator with a passive constraining spherical joint, which consists of a base, a platform, a support pole and three legs. The base is fixed and the platform carries the head of the robot. The support pole and three legs connect the platform to the base. The spherical joint on the support pole guarantees the spherical trajectory of the platform. The prismatic joints in the legs serve as actuator of the whole mechanism.</br>

### model simplification
To carry out optimization, some simplifications are necessary. The actual mechanism is too complex to optimise because there are too many unnecessary factors. So the goal of simplification is to focus on the most significant features of the mechanism. The following is how the simplification goes.

1. The most important features of the mechanism are the 6 U-joints and the Spherical joint on the support pole. Considering that the size of these joints are not important during optimization, we can use 7 points to represent these 7 joints. Based on that, the three points on the base will form a triangle, which can be found as the actual base, and the same to the platform. Then the prismatic joints are the lines between two U-joint points on the base and the platform. The picture of the actual model and abstract model are shown. </br>

2. For the present mechanism, the point of the spherical joint is not exactly on the plane of the platform, but they are very close. According to the result of simulation in Simulink, the distance between that point and the platform plane has little influence on the performance of the mechanism when this distance is small. So we can assume the distance is zero so that one parameter of optimization can be reduced.</br>

3. When the actual mechanism moves around, some attitudes are impossible to achieve because of the constraints of actuator and collision. So the workspace of the mechanism is limited and during the optimization, we can compute the possible workspace under these constraints. First, the actuator (prismatic joint) has a limited stroke length, so the length of the actuators must always stay in its working range. With the coordinates of the joint points, we can calculate the distance of the actuators easily, so this constraint is not difficult to consider. Second, if the support pole and any of the three legs are too close, they will crash into each other, so the collision is another constraint. The detection of collision is complex since the shape of the leg is not very regular (I mean, not like a cylinder). But we can still set a critical distance (this parameter is *delta*) for collision detection. If the distances between each leg and the support pole is longer than *delta*, it is safe; if not, we claim the collision happens. As the simulation in Simulink shows, this is a valid method to detect collision.</br>

4. For the actual mechanism, the rotation range of U-joint is also limited so this is another constraint.(deviation angle must be lower than *alfa0*). But we can change the rotation range by change U-joints. So in the optimization function, instead of regarding the rotation range another constraint, the U-joint rotation range is calculated as an output.</br>

## Main Function Instruction
This part introduces how to use the optimization function (neck_optimise), including variable definition and optimization method . This function has a lot limitations, if you want to edit the code, the next part (Code details) as well as the comments in the function will help you understand the code.</br>

### Variable Definition
The optimization function is defined by "function \[l0, theta0, angles, jangles] = optimise(r,R,H,delta)". </br>

Input:</br>
name &emsp; data categary  &emsp;&emsp;&emsp;&emsp; explaination </br>
&emsp;r&ensp; &emsp;&emsp;1\*1 double &emsp; the circumradius of the triangle on the platform</br>
&emsp;R &emsp;&emsp; 1\*1 double &emsp; the circumradius of the triangle on the base</br>
&emsp;H &emsp;&emsp; 1\*1 double &emsp; the height of the support pole</br>
&ensp;delta&ensp;&emsp; 1\*1 double &emsp; the critical value of collision</br>
Output:</br>
&emsp;l0 &emsp;&emsp; 1\*1 double &emsp; the natural length of the acuator </br>
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;"natural" means the mechanism is at its natural position (pitch, roll are both 0 and yaw is theta0)</br>
theta0 &emsp; 1\*1 double &emsp; the natural angle of yaw </br>
angles &emsp; 2\*3 double &emsp; the rotation range of roll,pitch,yaw when the other two angles are both 0 (the rotation order is roll,pitch,yaw)</br>
anguvel&emsp;2\*3 double &emsp; the max angular speed of roll,pitch,yaw</br>
jangles &emsp;1\*1 double &emsp; the angle range of the U-joint required by the workspace</br>

### Optimal Method
#### Optimal goal
The optimization goals include required angle ranges (workspace goal) and required angular velocities of roll, pitch and yaw (speed goal). The workspace of the mechanism means the size of space the mechanism can achieve. Indeed, if we regard (roll, pitch, yaw) as coordinates for different positions of the mechanism, we can work out the critical surface. (Each point on the surface represents a critical position, in which the mechanism just breaks one of the constraints.). However, the surface is irregular and it is hard to evaluate the workspace with this surface. So I use the maximum angles of roll, pitch and yaw that the mechanism can achieve when the other two angles are 0 (the parameter *angles*) to represent the size of workspace.</br>

Related definition statements:</br>

angles_req=\[roll+,pitch+,yaw+;roll-,pitch-,yaw-]; &emsp;&emsp; defines required angle ranges of roll, pitch and yaw when the other two angles are 0.</br>
anguvel_req=\[roll_speed,pitch_speed,yaw_speed]; &emsp; defines required angular velocities. </br>

Given the size of the mechanism (*r*,*R*,*H*), the optimization function will find the best natural place (*theta0*,*l0*) and the best minimum actuator length to make the mechanism get as close as possible to these goals.</br>

#### Optimal constraint
As mentioned in the previous section, there are two constraints that we need to consider, the stroke length of the actuator and mechanical collision.</br> 

#### Method
The optimization process is change the minimum actuator length to change the configuration of the mechanism. As to each configuration, find the best natural place (*theta0_n,l0_n*) and work out performance parameters (*angles_n,vm,jangles*).
For each configuration, I defined an optimal function (*target_func*) to evaluate how close the workspace size of different configurations is to the optimal goal (*angles_req*).</br>

To define *target_func*, we first need to define the difference (*d_angles*) between *angles_n* and *angles_req*: </br>

$d\_angles=\left|angles\_req\right|-\left|angles\_n\right|$ </br>       
d\_angles is 2\*3 double
If the ranges of roll, pitch and yaw corresponding to *angles* are in the ranges corresponding to *angles_req*, the difference of that part is 0; if not, the difference of that part is the absolute value of that part. </br>
Then the *target_func* is defined as the sum of all elements in d_angles.</br>
</br>
The optimization function (*optimise*) first generates a series of configurations with a fixed step size and then it traverses all the configurations to find a best configuration which has the minimum value of *target_func*. During the traversal, the optimization function also works out the required actuator speed and the maximum deviation angles of U-joints. 

## Code Details

This part only explains the function named "*required_speed*", because the other codes are easy to understand with the comments in these codes.</br>

*required_speed* calculates the required speed for the actuator to achieve the required angular speed of the mechanism over the whole workspace. The idea is traversing all possible positions in the workspace with a fixed step-length (1 degree in the function, you can take a smaller step-length, but remember running time will be longer) and then computing the differences of the lengths of three actuators between two neighbouring positions. "Two neighbouring positions" means that the coordinates (roll,pitch,yaw) of one position is only different from those of the other position in one angle among roll, pitch and yaw for one step-length, while the other two angles are the same. Since the required angular velocity (*anguvel_req*) are known, the time between two neighbouring positions can be worked out and we can further compute the velocities of three actuators between two positions. Finally we take the maximum velocity of three actuators as the required speed.</br>

There are two points to note to understand the code of this function. First, there are three angles (roll, pitch, yaw), whose angular speeds need to be satisfied. So three times of traversal over the whole workspace is needed and the order of configuration search traversal in the function is yaw, roll and finally pitch.  Second, the search of yaw, pitch and roll all begins at (0,0,0), but there are two borders for each coordinate( take yaw as an example, yaw+ and yaw- are two borders). So the direction of search need to switch when the search meets the first border. In other words, if the original search direction is positive, switch it to negative when the search reaches yaw+ (pitch+ or roll+). 

This function can be put after the process of optimization because it has no influence on the workspace optimization. But right now this function is put right after the configuration workspace-calculation function (*configuration2*) in the process of optimization.(you can improve the running time by running this function after the workspace optimization) This is because I want to calculate the required velocity for different configurations. The result shows that the change of the velocity is very limited for different configurations. So That is to say the optimization for the workspace is roughly equal to the optimization for both workspace and speed of the mechanism.</br>
