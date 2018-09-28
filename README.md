readme
======
This readme file details how to use the optimization function for the neck mechanism and also goes through the idea of the code.The instrution goes into three parts, including background information, main funciton instruction and code details. To use the optimise function, you just need to read the first two parts of this file. If you want to change the code to achieve a better optimization or implement other functionality, the last part will be helpful to edit the code.

## Background Information
The background information includes model explanation and model simplification. The part of model explanation introduces the actual neck mechanism, while the part of model simplification abstracts the actual mechanism into a geometric model. This simplification facilitates the  process of optimization.
### model explanation
The neck mechanism is a 3-UPU (3-universal-prismatic-universal) parallel manipulator with a passive constraining spherical joint, which consists of a base, a platform, a support pole and three legs. The base is fixed and the platform carries the head of the robot. The support pole and three legs connect the platform to the base. The spherical joint on the support pole guarantees the spherical trajectory of the platform. The prismatic joints in the legs serve as actuator of the whole mechanism.

### model simplification
To carry out optimization, some simplifications are necessary. The actual mechanism is too complex to optimise because there are too many unnecessary factors. So the goal of simplification is to focus on the most significant features of the mechanism. The following is how the simplification goes.

1. The most important features of the mechanism are the 6 U-joints and the Spherical joint on the support pole. Considering that the size of these joints are not important during optimization, we can use 7 points to represent these 7 joints. Based on that, the three points on the base will form a triangle, which can be found as the actual base, and the same to the platform. Then the prismatic joints are the lines between two U-joint points on the base and the platform. The picture of the actual model and abstact model are shown. 

2. For the present mechanism, the point of the spherical joint is not exactly on the plane of the platform, but they are very close. According to the result of simulation in Simulink, the distance between that point and the platform plane has little influence on the performace of the mechcanism when this distance is small. So we can assume the distance is zero so that one parameter of optimization can be reduced.

3. When the actual mechanism moves around, some attitudes are impossible to achieve because of the constraints of actuator and collision. So the workspace of the mechanism is limited and during the optimization, we can compute the possible workspace under these constraints. First, the actuator (prismatic joint) has a limited stroke length, so the length of the actuators must always stay in its working range. With the coordinates of the joint points, we can calculate the distance of the actuators easily, so this constaint is not difficult to consider. Second, if the support pole and ang of the three legs are too close, they will crash into each other, so the collision is another constraint. The detection of collision is complex since the shape of the leg is not very regular. But we can still set a critical distance (this parameter is *delta*) for collision detection. If the distances between each leg and the support pole is longer than *delta*, it is safe; if not, we claim the collision happens. As the simulation in Simulink shows, this is a valid method to detect collision.

4. For the atual mechanism, the rotation range of U-joint is also limited so this is another constraint.(deviation angle must be lower than *alfa0*). But we can change the rotation range by change U-joints. So in the optimization function, instead of regarding the rotation range another constraint, the U-joint rotation range is calculated as an output.

## Main Function Instruction
This part introduces how to use the optimization function (neck_optimise), including variable defination, optimization method and some notices. This function has a lot limitations, if you want to edit the code, the next part (Code details) as well as the comments in the function will help you understand the code.

### Variable Defination
The optimization function is defined by "function \[l0, theta0, angles, jangles] = optimise(r,R,H,delta)"

Input:</br>
name &emsp; data categary  &emsp;&emsp;&emsp;&emsp; explaination </br>
&emsp;r&ensp;&emsp;&emsp; 1\*1 double &emsp; the circumradius of the triangle on the platform</br>
&emsp;R &emsp;&emsp; 1\*1 double &emsp; the circumradius of the triangle on the base</br>
&emsp;H &emsp;&emsp; 1\*1 double &emsp; the hight of the support pole</br>
&ensp;delta &emsp; 1\*1 double &emsp; the critical value of collision</br>
Output:</br>
&emsp;l0 &emsp;&emsp; 1\*1 double &emsp; the natural length of the acuator </br>
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;"natural" means the mechanism is at its natural position (pitch, roll are both 0 and yaw is theta0)</br>
theta0 &emsp; 1\*1 double &emsp; the natural angle of yaw </br>
angles &emsp; 2\*3 double &emsp; the rotation range of roll,pitch,yaw (the rotation order is roll,pitch,yaw)</br>
anguvel &emsp;2\*3 double &emsp; the max angular speed of roll,pitch,yaw</br>
jangles &emsp;1\*1 double &emsp; the angle range of the U-joint required by the workspace</br>

### Optimization Method
#### Optimization goel
The optimization goels include required angle ranges and required angular velocities of roll, pitch and yaw.
Related defination satements:</br>

angles_req=\[roll+,pitch+,yaw+;roll-,pich-,yaw-]; &emsp;       defines required angle ranges<\br>
anguvel_req=\[roll_speed,pitch_speed,yaw_speed]; &emsp;&ensp;  defines required angular velocities<\br>

Given the size of the mechanism (*r*,*R*,*H*), the optimizaition function is to find a best natural place (*theta0*,*l0*) to approach these goels.<\br>
#### Optimization constraint
As mentioned in the previous section, there are two constraints that we need to consider, the stroke length of the actuator and mechanical collision.

#### Method
The optimization process is change the minimum length of the actuator to change the configuration of the mechanism. As to each configuration, look for the best natural place and  find the best parameter
In the function, I defined an optimal function (*Target_func*) to evaluate the workspace of different configurations.

### Notices

## Code details



