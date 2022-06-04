# Route Planning Project

## Overview

In this project, I created a route planner that plots a path between two points on a map. We use real map data from the OpenStreetMap project (i.e. open-source user-generated maps), and the A* search algorithm to find the path between two points. 

I had to extend the starter code (provided by the 2D graphics library, IO2D) to search for, and display a path between two points on the map. 

<img src="map.png" width="600" height="450" />

## The data

OpenStreetMap data is in the form of an XML file (used to represent structured information).

The file contains the following:

1. **Nodes**: a node is a single point on the map. It has an **id**, a **latitude**, and **longitude**. 
2. **Ways**: a way is a group of nodes. This group of nodes represent a feature on the map (for example, a road, a park boundary). 
    
    Each way has **at least** **one** **tag** that provides some information about the way). 
    
    Each way also belongs to **at least one relation** (explained next).
    
3. **Relations**: could be interpreted as a collection of ways. A relation could list out the ways which form a highway, cycle route, or bus route. 

In this project, I worked primarily with nodes. The code that was provided is used to determine which nodes are neighbours by using the ways, and relations that those nodes belong to. 

The code for the parsing of the OpenStreeMap data, and the data structures which are used to store the data has been provided. 

## Project code overview

In **main()**:

1. We read-in the data by invoking the **ReadFile() function.**
2. With the data, a **RouteModel object** is created. 
    1. This object holds the OpenStreetMap data in a convenient format, and provides some **methods** for using the data. 
    2. The **RouteModel class** also has a **sub-class** called **Node** that is used frequently. 
    3. The **RouteModel class** provides the **FindClosestNode() method.** 
        1. When the user provides a starting point or an ending point, this method will find the **closest Node** in the stored data to the coordinates provided (i.e. to say that the coordinates provided by the user may not represent a Node - the closest Node for the starting coordinates, and the closest Node for the ending coordinates need to be found).
    4. The **Node class** has the following **members**:
        1. Each **Node object** is created with an **empty vector of neighbour Nodes** (which may be populated by calling the **FindNeighbors() method** of the **Node class,** as explained in the section that follows).
        2. Furthermore, a **Node object** has a pointer to the **parent Node**. This is a **null pointer**. **However**, for the **neighbours of the current Node** (which may be populated by calling the **FindNeighbors() method** as explained in the section that follows), the **parent Node** is to be set as the current Node. 
            
            Keeping track of the parent of each Node, allows the path to be reconstructed once we reach the end. 
            
        3. An initial **g value** of zero. **g value** is set for the neighbours of the current Node. It is calculated as the current Node’s g value (which is zero), plus the g value of the neighbour Node (calculated as distance from current Node to the neighbour Node). 
            
            Nodes have different distances between them, and we want to record a greater cost to travel longer distances between nodes. 
            
            This will help us to choose the shortest next step when presented with several options. 
            
            The distance is calculated using the **Distance() method** of the **Node class,** explained in the following section. 
            
        4. An initial **h value** of zero. **h value is** set for each neighbour of the current Node. It is the distance from **neighbour Node** to the **end Node** calculated as distance from neighbour Node to the end Node, using the **Distance() method** of the **Node class** explained in the following section.
        5. **visited**. This is a boolean variable to represent whether this node has been explored.
    5. The **Node class** provides **two methods:**
        1. **FindNeighbors():** This method can be used to populate the **empty vector of neighbour Nodes of the current Node**, with valid neighbours (i.e. has not yet been **visited**).
        2. **Distance():** This method provides the distance between two Nodes. This is used to determine the cost of traveling to the next Node.  
3. After **main()** creates the **RouteModel object, a RoutePlanner object** is created using the created **RouteModel object**, and **user-inputed starting and ending coordinates**. 
    1. This class provides methods to conduct A* search - **AStarSearch(), NextNode(), ConstructFinalPath(), CalculateHValue(), AddNeighbors().**
    2. This is the class where most of my code is written in. 
    3. The **RoutePlanner class** has the following **members**:
        1. **Starting node** - The node closest to the starting coordinates provided by the user. This would be found by using the **FindClosestNode() method** from the **RouteModel class.**
        2. **Ending node** - The node closest to the ending coordinates provided by the user. This would be found by using the **FindClosestNode() method** from the **RouteModel class.**
        3. **A list of open Nodes.** These are the Nodes that have been populated by the **AddNeighbors() method** of the **RoutePlanner class.** 
    4. **NextNode() method** sorts the **list of open Nodes** and provides the **next Node** to explore. We sort it by selecting the node with the **lowest g + h values.** The selected Node is then removed from **the list of open Nodes.** 
    5. **ConstructFinalPath() method** creates a **vector of Nodes** in the path from the beginning Node, to the end Node. This vector can then be used to render the final path. 
    6. **CalculateHValue() method** is used to set the **h value** of the **Node object** (i.e. the distance from Node to **end Node**)**.** 
    7. **AddNeighbors() method** has several steps.
        1. **AddNeighbors() method** will start by calling **FindNeighbors() method** from the **Node class** (the sub-class in **RouteModel class),** on the **current Node.** 
            
            **Each Node object** is created with an **empty vector of neighbour Nodes**, as stated in the section on the **Node class (**within the **RouteModel class** section).
            
            **FindNeighbors() method** from the **Node class** (the sub-class in **RouteModel class)** will populate the empty vector of neighbour Nodes of the current Node, with valid neighbours.
            
        2. Then, for **each neighbour Node of the current Node**, the **parent of that Node** is set to **current Node**, as stated in the section on the **Node class (**within the **RouteModel class** section). 
            
            Keeping track of the parent of each Node, allows the path to be reconstructed once we reach the end. 
            
        3. After setting the **parent Node,** for each neighbour Node, the **g-value** of the neighbour Node is set to the current g-value plus the distance from the current Node to the neighbour Node. 
            
            Nodes have different distances between them, and we want to record a greater cost to travel longer distances between nodes. This will help us to choose the shortest next step when presented with several options. 
            
        4. Next, we also set the **h-value** for **each neighbour Node** using the **CalculateHValue() method.**
        5. Finally, the **neighbour Nodes** are added to the **open list of Nodes** of the **RoutePlanner object.**
        6. These **neighbour Nodes** are then marked as **visited (**i.e. we have taken them into account)**.**
4. **main()**, then, calls the **AStarSearch() method** within the **RoutePlanner class.**
    1. **AStarSearch() method** starts a **while-loop** that continues as long as the **list of open Nodes** of the **RoutePlanner object** is **non-empty.**
    2. Inside the **while-loop,** the **NextNode() method** from the **RoutePlanner class** is called. As explained above, this is to sort the list of open Nodes and provide the next Node to be explored. The selected Node is then removed from the **list of open Nodes**.
    3. In the **while-loop,** we then check whether the Node provided by the **NextNode() method** from the **RoutePlanner class** is the goal. 
    4. If the next Node happens to be the goal, the **ConstructFinalPath() method** from the **RoutePlanner class** is invoked. 
    5. If the next node is not the goal, the **AddNeigbors() method** from the **RoutePlanner class** is called. 
5. **main()** will then create a **Render object** and render the map with the results from A* search after the **AStarSearch() method** within the **RoutePlanner class** has finished executing,

## Cloning

When cloning this project, be sure to use the `--recurse-submodules` flag. Using HTTPS:
```
git clone https://github.com/udacity/CppND-Route-Planning-Project.git --recurse-submodules
```
or with SSH:
```
git clone git@github.com:udacity/CppND-Route-Planning-Project.git --recurse-submodules
```

## Dependencies for Running Locally
* cmake >= 3.11.3
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1 (Linux, Mac), 3.81 (Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 7.4.0
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same instructions as make - [install Xcode command line tools](https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* IO2D
  * Installation instructions for all operating systems can be found [here](https://github.com/cpp-io2d/P0267_RefImpl/blob/master/BUILDING.md)
  * This library must be built in a place where CMake `find_package` will be able to find it

## Compiling and Running

### Compiling
To compile the project, first, create a `build` directory and change to that directory:
```
mkdir build && cd build
```
From within the `build` directory, then run `cmake` and `make` as follows:
```
cmake ..
make
```
### Running
The executable will be placed in the `build` directory. From within `build`, you can run the project as follows:
```
./OSM_A_star_search
```
Or to specify a map file:
```
./OSM_A_star_search -f ../<your_osm_file.osm>
```

## Testing

The testing executable is also placed in the `build` directory. From within `build`, you can run the unit tests as follows:
```
./test
```

