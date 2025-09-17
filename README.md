Here is a detailed and professional `README.md` for the **autonomous docking** ROS package, based on your original content. It includes clear structure, additional installation commands, and formatting improvements — all without emojis.

---

# Autonomous Docking

ROS packages for autonomous docking of differential-drive robots using fiducial markers.

This solution, `autodock`, implements a reliable and accurate **state machine-based autonomous docking system** for robots with differential drive. It operates **on top of the ROS navigation stack** and **requires a camera** for detecting fiducial markers placed on the docking station. The system has been thoroughly tested in both simulation and real-world environments.

---

## Package Overview

* **`autodock_core`**: Core scripts and logic for the docking state machine.
* **`autodock_examples`**: Example scripts and test cases for autodocking.
* **`autodock_sim`**: Simulation environment for testing autodocking in Gazebo.

---

## Requirements

* Ubuntu 20.04
* ROS Noetic Ninjemys ([Install Instructions](http://wiki.ros.org/noetic/Installation/Ubuntu))
* A camera-equipped robot (for fiducial detection)
* Catkin workspace (`catkin_ws`)

---

## Dependencies

Install the required ROS packages and external dependencies:

```bash
# Create and initialize catkin workspace
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src

# Clone this repository and required dependencies
git clone https://github.com/YOUR_USERNAME/autodock.git
git clone https://github.com/UbiquityRobotics/fiducials.git

# (Optional) Turtlebot3 simulation support
sudo apt update
sudo apt install ros-noetic-turtlebot3-gazebo ros-noetic-turtlebot3-navigation ros-noetic-turtlebot3-msgs

# Install any remaining dependencies
cd ~/catkin_ws
rosdep update
rosdep install --from-paths src --ignore-src -r -y

# Build the workspace
catkin_make

# Source the workspace
source devel/setup.bash
```

---

## Running in Simulation with MockRobot

This launches a Gazebo simulation with a mock differential drive robot and a docking station with fiducial markers.

```bash
roslaunch autodock_sim dock_sim.launch
```

Once launched, send a docking action request:

```bash
rostopic pub /autodock_action/goal autodock_core/AutoDockingActionGoal "{}" --once
```

**Expected Behavior**:
The robot will initiate an autonomous docking sequence. Upon successful docking, it will send a `std_srvs/Trigger` request to the `MockCharger`, which verifies the docking state and sends a success response back.

---

## Useful Commands

```bash
# Manual teleoperation of the robot
rosrun teleop_twist_keyboard teleop_twist_keyboard.py

# Cancel active docking action
rostopic pub /autodock_action/cancel actionlib_msgs/GoalID "{}" --once

# Pause docking (useful during testing)
rostopic pub /pause_dock std_msgs/Bool "data: true" --once
```

---

## Smoke Test Script

Run multiple randomized docking tests in simulation:

```bash
# Run 10 iterations
rosrun autodock_examples dock_sim_test.py -c 10
```

---

## Running with TurtleBot3 (Front-Dock)

This setup demonstrates autodocking using a TurtleBot3 robot equipped with a front-facing camera.

```bash
# Launch TurtleBot3 with front-docking simulation
roslaunch autodock_sim tb3_dock_sim.launch

# Send docking goal
rostopic pub /autodock_action/goal autodock_core/AutoDockingActionGoal "{}" --once
```

---

## TurtleBot3 Navigation + Autodocking Integration

Integrate `autodock` with the full TurtleBot3 navigation stack.

### Step 1: Ensure Dependencies

```bash
sudo apt install ros-noetic-turtlebot3-gazebo ros-noetic-turtlebot3-navigation ros-noetic-turtlebot3-description
```

> You can also install all relevant packages:

```bash
sudo apt install ros-noetic-turtlebot3*
```

Make sure the `TURTLEBOT3_MODEL` environment variable is set:

```bash
export TURTLEBOT3_MODEL=burger
```

---

### Step 2: Launch World + Autodock Server

```bash
roslaunch autodock_sim tb3_nav_dock_sim.launch
```

---

### Step 3: Launch Navigation Stack with Map

Use an existing map or create one following the TurtleBot3 SLAM tutorial.

```bash
roslaunch turtlebot3_navigation turtlebot3_navigation.launch \
  map_file:=$HOME/catkin_ws/src/autodock/autodock_sim/maps/map.yaml \
  open_rviz:=false
```

---

### Step 4: Dock the Robot

Make sure the robot is localized and facing the charging station:

```bash
rostopic pub /autodock_action/goal autodock_core/AutoDockingActionGoal "{}" --once
```

---

## Obstacle Handling

The docking process includes a basic obstacle avoidance mechanism via the `obstacle_observer` node. If an obstacle is detected during critical docking states (`predock`, `steer_dock`, or `parallel_correction`), the system will **pause** the docking process. Remove the obstacle to resume.

---

## Notes

* This package **does not modify** your navigation stack and can be integrated with existing move\_base pipelines.
* The docking markers should be **visible and uniquely identifiable** using fiducial IDs.
* For best results, calibrate the camera and ensure stable marker detection.

---

## Troubleshooting

* If the docking action doesn't start, verify that:

  * Camera is publishing images.
  * Fiducials are detected.
  * Robot is properly localized.
  * The `/autodock_action/goal` topic is publishing correctly.

* If `catkin_make` fails:

  * Make sure all dependencies are correctly installed.
  * Run `rosdep install --from-paths src --ignore-src -r -y` again.

---

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

---

## Acknowledgments

* Ubiquity Robotics for their fiducial marker detection package.
* ROBOTIS for the TurtleBot3 simulation tools.

---

Let me know if you'd like this in a `README.md` file format or converted into documentation (e.g. Sphinx, Doxygen).
