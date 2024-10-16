# Active vision project
Grasping novel objects using Heuristic and ML based active vision.
> Franka Emika Panda <br>
> Parallel jaw gripper <br>
> Intel Realsense D435i <br>

## Bash scripts

1. **dataCollector.sh** :
    <br> Begins automatic data collection for specified objects. Arguments must be adjusted inside the .sh file.
    <br> ```rosrun active_vision dataCollector.sh```

    | Argument       | Type | Description |
    | -------------- | ------- | ------ |
    | objectID | List of ints | List of object ID's from objects.yaml to test |
    | nData | int | Number of poses to test. Should generally be left at default value of 100 |

    Sample Workflow:
    User A wants to collect optimal paths for objects #1 and 3. They open dataCollector.sh, set ```objectID=(1 3)``` and 
    then run ```rosrun active_vision dataCollector.sh``` from the terminal. A new window will automatically open and launch
    gazebo, and a second window will automatically open and run the data collection routine on object #1. After nData points have been
    collected, both windows will close, and be reopened for object #3. The data will be saved in the dst directory, under a new
    directory named Data_{number}, where {number} is 1 if no Data_ directories exist, or the highest valued directory +1 if they do. 

2. **policyEvaluator.sh** :
    <br> Begins automatic policy testing on specified objects for specified policies. Arguments must be adjusted inside the .sh file.
    <br> ```rosrun active_vision policyEvaluator.sh```

    | Argument       | Type | Description |
    | -------------- | ------- | ------ |
    | objectID | List of ints | List of object ID's from objects.yaml to test |
    | nData | List of ints | Number of poses to test for each object |

    | Argument       | Values | Description |
    | -------------- | ------- | ------ |
    | simulationMode | SIMULATION (Default) <br> FRANKASIMULATION <br> FRANKA | Gazebo + Floating realsense <br> Gazebo + Franka <br> Franka |
    | Policyxxx | ("Policy" "Unique description" "Param 1 name" "Param 1 value" ...) | List of policy characteristics to be tested. See file for details.

    Sample Workflow:
    User A wants to test policies Brick and QLearn for objects #1 - 4. They open policyEvaluator.sh, set ```objectID=(1 2 3 4)```, uncomment the lines ```Policy1B=("BRICK" "Brick")``` and ```Policy5=("QLEARN" "Q_Learning" "/active_vision/policyTester/HAFstVecGridSize" 5)``` 
    then run ```rosrun active_vision dataCollector.sh``` from the terminal. A new window will automatically open and launch
    gazebo, and a second window will automatically open and run the data collection routine on object #1. After nData points have been
    collected, both windows will close, and be reopened for object #3. The data will be saved in the dst directory, under a new
    directory named Test_{number}, where {number} is 1 if no Data_ directories exist, or the highest valued directory +1 if they do. 

    User B has developed a new policy called BPolicy that they want to test on the same objects as User A. They should first add their policy to trainedPolicyService.py, then add the line ```Policy7=("BPOLICY" "BPolicy" "[parameter1 name]" "[parameter1 value]" ...)``` to policyEvaluator.sh. They then run ```rosrun active_vision dataCollector.sh``` as before.

## Launch files

1. **workspace.launch** :
    <br> Starts the environment for the required simulation mode.
    <br> ```roslaunch active_vision workspace.launch simulationMode:="FRANKASIMULATION"```

    | Argument       | Values | Description |
    | -------------- | ------- | ------ |
    | simulationMode | SIMULATION (Default) <br> FRANKASIMULATION <br> FRANKA | Gazebo + Floating realsense <br> Gazebo + Franka <br> Franka |
    | visual*        | ON (Default) <br> OFF | Gazebo GUI ON <br> Gazebo GUI OFF |

    \* Only applicable for SIMULATION and FRANKASIMULATION

2. **policyTester.launch** :
    <br> Starts the environment, starts the required policy and tests the policy.
    <br> ```roslaunch active_vision policyTester.launch policy:="3DHEURISTIC"```

    | Argument       | Values | Description |
    | -------------- | ------- | ------ |
    | simulationMode | SIMULATION (Default) <br> FRANKASIMULATION <br> FRANKA | Gazebo + Floating realsense <br> Gazebo + Franka <br> Franka | |
    | policy         | NIL (Default) <br> HEURISTIC <br> 3DHEURISTIC <br> PCA_LDA <br> PCA_LDA_LR <br> PCA_LR <br> RANDOM | Manual mode : User enters the direction to move <br> Heuristic Policy <br> Heuristic Policy <br> Trained Policy <br> Trained Policy <br> Trained Policy <br> Random Policy : Takes random steps |

## Active vision algorithm scripts

1. **3DheuristicPolicyService.cpp** :
    <br> Creates a service that accepts active_vision::heuristicPolicySRV requests containing object and unexplored
    pointclouds, and returns an integer corresponding to the next best direction selected by the 3D Heurisitc.
    <br> ```rosrun active_vision 3DheuristicPolicyService```
    
    | Direction Number | Value |
    | ---------------- | ----- |
    | 1 | North |
    | 2 | North East |
    | 3 | East |
    | 4 | South East |
    | 5 | South |
    | 6 | South West |
    | 7 | West |
    | 8 | North West |

2. **heuristicPolicyService.cpp** :
    <br> Creates a service that accepts active_vision::heuristicPolicySRV requests containing object and unexplored
    pointclouds, and returns an integer corresponding to the next best direction selected by the 2D Heurisitc.
    <br> ```rosrun active_vision heuristicPolicyService```
    
    | Direction Number | Value |
    | ---------------- | ----- |
    | 1 | North |
    | 2 | North East |
    | 3 | East |
    | 4 | South East |
    | 5 | South |
    | 6 | South West |
    | 7 | West |
    | 8 | North West |

3. **trainedPolicyService.py** :
    <br> Creates a service that accepts active_vision::heuristicPolicySRV requests containing object and unexplored
    pointclouds, and returns an integer corresponding to the next best direction selected by the model.
    <br> ```rosrun active_vision trainedPolicyService.py```

    Ensure that the parameters in parameters.yaml file are set correctly before running the script. Following are the parameters used:
    | Parameter      | Type   | Value | Description |
    | -------------- | ------ | ----- | ----------- |
    | policy         | string | 'PCA_LDA' | Policy to be used for testing <br> Same options as mentioned in policyTester.launch policy argument|
    | directory      | string | '/dataCollected/testData/' | Directory to which the PCD, CSV will be saved |
    | csvStVecDir   | string | '/misc/State_Vector/' | Directory where the state vector has been stored |
    | csvStVec     | string | 'obj_2_3_g5.csv' | Name of csv inside csvStVecDir |
    | HAFstVecGridSize | int | 5 | Dimension of the HAF state vector used |
    | PCAcomponents  | float | 0.85 | Parameter for any policy which uses PCA <br> Use a value from 0-1 to set components based on the explained_variance_ratio cumulative sum |
    
    | Direction Number | Value |
    | ---------------- | ----- |
    | 1 | North |
    | 2 | North East |
    | 3 | East |
    | 4 | South East |
    | 5 | South |
    | 6 | South West |
    | 7 | West |
    | 8 | North West |

4. **trainingPolicy.py** :
    <br> TODO

5. **QLearning3.py** :
    <br> TODO 

## Data Collection and Policy testing scripts

1. **environmentTesting.cpp** :
    <br> Code to test the individual functions of environment.cpp file. Used to test any changes to the environment.cpp file.

    | S.No. | Functions available |
    | :---: | ------------------- |
    | 01 | Spawn and delete objects and its configurations on the table |
    | 02 | Load and view the gripper model |
    | 03 | Move the realsense/franka to a custom position |
    | 04 | Continuously move the realsense in a viewsphere with centre on the table |
    | 05 | Read and view the data from realsense |
    | 06 | Read and fuse the data from different viewpoints |
    | 07 | Extract the table and object from the environment |
    | 08 | Generate the initial unexplored pointcloud based on the object |
    | 09 | Update the unexplored pointcloud based on different viewpoints |
    | 10 | Grasp synthesis based on different viewpoints. |
    | 11 | Store and rollback configurations |
    | 12 | Surface patch testing |
    | 13 | Gripper open close testing |
    | 14 | Testing object pickup |
    | 15 | Testing moveit collision add/remove |
    | 16 | Testing moveit constraint |
    | 17 | Rotation and shaking test |

    *Note: Some of these functions only work in simulations*

    Example usage
    <br> ```roslaunch active_vision workspace.launch simulationMode:="SIMULATION"```
    <br> ```rosrun active_vision environmentTesting```

2. **dataCollector.cpp** :
    <br> Code to generate data sets and store point clouds for training purpose. Data is stored to a csv file. Also used for the BFS search.
    <br> ```rosrun active_vision dataCollector <mode>```
    | Argument       | Values | Description |
    | -------------- | ------ | ----------- |
    | mode | 0 (Default)  <br> 1 | Data collection <br>  BFS search|

    Ensure that the parameters in parameters.yaml file are set correctly before running the script. Following are the parameters used:
    | Parameter      | Type   | Value | Description |
    | -------------- | ------ | ----- | ----------- |
    | relative_path  | bool   | True  | Whether the path used is relative to the package path |
    | directory      | string | '/dataCollected/trainingData/' | Directory to which the PCD, CSV will be saved |
    | objID          | int    | 2 | Object for which data has to be collected. Refer ojects.yaml |
    | csvName        | string | 'default.csv' | CSV file to which the data has to be saved. Setting to 'default.csv' uses timestamp as the csv name |
    | nData          | int    | 25 | Number of dataPoints to be recorded for each object pose |
    | homePosesCSV   | string | 'dataCollectionTreeD2.csv' | CSV file in active_vision/misc/ folder to be used to get home poses |
    | maxRdmSteps    | int    | 4 | # Max number of random steps to be taken before restarting |
    | nRdmSearch     | int    | 3 | Number times a random search is initiated before going to the next direction |

    Example usage
    <br> ```roslaunch active_vision workspace.launch simulationMode:="SIMULATION" visual:="OFF"```
    <br> ```rosrun active_vision dataCollector```

3. **policyTester.cpp** :
    <br> Code to test a required policy with different poses of a object
    <br> ```rosrun active_vision policyTester <RunMode>```

    | Argument | Values | Description |
    | -------- | ------ | ----------- |
    | RunMode  | 0  <br> 1 <br> 2 | Manual mode <br> Heuristic policy <br> Trained policy |

    Similar to dataCollector ensure that parameters for this are set correctly. Following are the parameters used:
    | Parameter      | Type   | Value | Description |
    | -------------- | ------ | ----- | ----------- |
    | policy         | string | 'PCA_LDA' | Policy to be used for testing <br> Same options as mentioned in policyTester.launch policy argument|
    | relative_path  | bool   | True  | Whether the path used is relative to the package path |           |
    | directory      | string | '/dataCollected/testData/' | Directory to which the PCD, CSV will be saved |
    | objID          | int    | 2 | Object for which data has to be collected. Refer ojects.yaml |
    | csvName        | string | 'default.csv' | CSV file to which the data has to be saved. Setting to 'default.csv' uses timestamp as the csv name |
    | csvStVecDir*   | string | '/misc/State_Vector/' | Directory where the state vector has been stored |
    | csvStVec*      | string | 'obj_2_3_g5.csv' | Name of csv inside csvStVecDir |
    | HAFstVecGridSize* | int | 5 | Dimension of the HAF state vector used |
    | PCAcomponents*  | float | 0.85 | Parameter for any policy which uses PCA <br> Use a value from 0-1 to set components based on the explained_variance_ratio cumulative sum |
    | maxSteps       | int    | 5 | Max number of steps to be taken before stopping |
    | nDataPoints    | int    | 10 | Number of data points for each pose to test (Max 200) |
    | yawAnglesCSVDir| string | '/misc/yawValues/' | Directory for yawAnglesCSV file |
    | yawAnglesCSV   | string | Seed1.csv | csv file to choose the yaw angles from |
    | uniformYawStepSize | int| 45 | Step size for yaw angles starting from 0 which will be checked apart from random values from the yawAnglesCSV |
    | heuristicDiagonalPref** | bool | false | If the 3D Heuristic should prefer the diagonals or not |
    | recordPtCldEachStep   | bool | true  | True if record pointcloud for each step |

    \* Only for trained policies
    ** Only for 3DHeuristic policy

    Example usage
    <br> ```roslaunch active_vision workspace.launch simulationMode:="SIMULATION" visual:="OFF"```
    <br> ```rosrun active_vision 3DheuristicPolicyService```
    <br> ```rosrun active_vision policyTester 1```
    <br> policyTester.launch runs these commands automatically

3. **genStateVec.cpp** :
    <br> Code to generate the state vector from the saved point clouds.
    <br> ``` rosrun active_vision genStateVec <Directory> <CSV filename> <Type> <Max Steps> ```

    | Argument | Type | Value | Description |
    | -------- | ---- | ----- | ----------- |
    | Directory | string | | Directory path where PCD and CSV files are there |
    | CSV filename | string | | CSV file generated by dataCollector |
    | Type | int | 1 | 1 : Height accumulated features (HAF) based |
    | Max Steps | int | 5 | Data with steps greater than this will be ignored |

## Visualization scripts

1. **visDataRec.cpp** :
    <br> Simplified visualization the point clouds saved by dataCollector / policyTester.
    <br> ``` rosrun active_vision visDataRec <Directory> <CSV filename> <Visual> ```
    | Argument | Type | Value | Description |
    | -------- | ---- | ----- | ----------- |
    | Directory | string | | Directory path where PCD and CSV files are there |
    | CSV filename | string | | CSV file generated by dataCollector / policyTester |
    | Visual | int | 1 <br>  2 | result.pcd only <br> obj.pcd , unexp.pcd , result.pcd |

2. **visDataRecV2.cpp** :
    <br> Detailed visualization the point clouds saved by policyTester. *recordPtCldEachStep* argument needs to be *true* while running policyTester.
    <br> ``` rosrun active_vision visDataRecV2 <Directory> <CSV filename> ```
    | Argument | Type | Value | Description |
    | -------- | ---- | ----- | ----------- |
    | Directory | string | | Directory path where PCD and CSV files are there |
    | CSV filename | string | | CSV file generated by dataCollector / policyTester |

3. **visDataCollectionPaths.cpp** :
    <br> Visualization the paths used by dataCollector to generate home poses.
    <br> ``` rosrun active_vision visDataCollectionPaths <CSV filename> ```
    | Argument | Type | Value | Description |
    | -------- | ---- | ----- | ----------- |
    | CSV filename | string | | homePosesCSV parameter used in dataCollector |

4. **visAllPaths.cpp** :
    <br> Visualizes all paths taken in a given .csv file. Requires the .csv file be stored
    in the same directory as the pointclouds that produced it.
    <br> ```rosrun active_vision visAllPaths /home/diyogon/Documents/WPI/Berk_Lab/mer_lab/ros_ws/src/projects/active_vision/dataCollected/storage/Data_22/ dataRec.csv 1```
    | Argument | Type | Value | Description |
    | -------- | ---- | ----- | ----------- |
    | Directory | string | | Directory where csv and pcd files are located |
    | CSV filename | string | | homePosesCSV parameter used in dataCollector |
    | Visual | int | 1 <br>  2 | result.pcd only <br> obj.pcd , unexp.pcd , result.pcd |

5. **visViewsphere.cpp** :
    <br> Code to visualize the viewsphere and navigate through it along the 8 directions.
    <br> ``` rosrun active_vision visViewsphere ```

6. **visBFS.py** :
    <br> TODO

7. **visWorkSpace.py** :
    <br> TODO

## Useful tools

1. **toolDataHandling.cpp** :
    <br> All functions related to data handling are defined here.
    * Loading and saving PCD & CSV
    * Get current time in yyyy_mm_dd_hhmmss format
    * Reading object details from objects.yaml

2. **toolStateVector.cpp** :
    <br> Tool which takes a object PCD, unexplored PCD, realsense pose and converts return the HAF State vector
    | Function | Description |
    | -------- | ----------- |
    | setInput | Set the two pointclouds and realsense pose |
    | setGridDim | Set the grid dimension |
    | setMaintainScale | Set if the x, y scale should be same or not |
    | calculate | Calculate the state vector based on the input |
    | print | print the state vector |
    | getStateVec | Return the state vector |
    | saveToCSV | Save the state vector to CSV |

3. **toolViewPointCalc.cpp** :
    <br> Tool to calculate things related to the realsense movement.
    * Convert between cartesian and viewsphere co-ordinate system
    * Check if the pose is valid
    * Distance between two viewsphere co-ordinates
    * Calculate new realsense position given its current position, direction to move and step size

4. **toolVisualization.cpp** :
    <br> Tool to create the PCL visualizer and customize it
    * Setup the visualizer with 1/2/3/4/6/8/9 viewports
    * Add viewsphere to a viewport
    * Add XYZRGB or XYZRGBNormal point cloud to a viewport
    * Set camera viewpoint based on a viewpshere co-ordinate
    * Add keyboard controls to jump between data points
    * Add mouse control to print a point data based on Shift + mouse left click

## Helpful tips

1. Viewing the arguments for a roslaunch file
    <br> ``` roslaunch --ros-args <pkg-name> <launch-file-name> ```

## FAQs / Known issues
    * Visibility constraint on moveit causing bad_alloc errors after ROS Noetic upgrade. Currently disabled.
