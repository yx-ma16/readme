readme
======
This readme file details how to use the optimise function and also goes through the idea of the code.The instrution goes into three parts, including background information, main funciton instruction and code details. To use the optimise function, you just need to read the first two parts of this file. If you want to change the code to achieve a better optimization or implement other functionality, the last part will be helpful to edit the code.

# Background Information
The background information includes model explanation and model simplification. The part of model explanation introduces the actual neck mechanism, while the part of model simplification abstracts the actual mechanism into a geometric model. This simplification facilitates the  process of optimization.
## model explanation
The neck mechanism is a 3-UPU (3-universal-prismatic-universal) parallel manipulator with a passive constraining spherical joint, which consists of a base, a platform, a support pole and three legs. The base is fixed and the platform carries the head of the robot. The support pole and three legs connect the platform to the base. The spherical joint on the support pole guarantees the spherical trajectory of the platform. The prismatic joints in the legs serve as actuator of the whole mechanism.


## model simplification
To carry out optimization, some simplifications are necessary. The actual mechanism is too complex to optimise because there are too many unnecessary factors. So the goal of simplification is to focus on the most significant features of the mechanism. The following is how the simplification goes.

1. The most important features of the mechanism are the 6 U-joints and the Spherical joint on the support pole. Considering that the size of these joints are not important during optimization, we can use 7 points to represent these 7 joints. Based on that, the three points on the base will form a triangle, which can be found as the actual base, and the same to the platform. 

2. 

3. 

# Main Function Instruction




# Code details



