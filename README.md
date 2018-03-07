mimic_joint_gazebo_tutorial
===========================

This package is a simple example on how to properly use the mimic joint plugin
in Gazebo, while avoiding the [gravity
bug](https://github.com/ros-simulation/gazebo_ros_pkgs/issues/612) in Gazebo7.

Tested on ROS Kinetic, Ubuntu 16.04.

Inspired by: [ROS Answers: How to do mimic joints that work in Gazebo?](https://answers.ros.org/question/283537/how-to-do-mimic-joints-that-work-in-gazebo/?answer=283550#post-id-283550)

Installation
------------

This repo has the following source dependency. Simply clone it into your catkin workspace:

    git clone https://github.com/roboticsgroup/roboticsgroup_gazebo_plugins.git


Running
-------

To see the plugin in action, run:

```bash
roslaunch mimic_joint_gazebo_tutorial gazebo.launch
```

A simple "gripper" will pop up. The right finger (green) is controlled by
ros_control, the left finger (red) is controlled by the mimic joint plugin and
simply follows the right finger if possible.

* switch to the `rqt_joint_trajectory_controller` window
* in the "controller manager ns" dropdown box, select "/controller_manager"
* in the "controller" dropdown box, select "gripper_controller"
* press the red button so it becomes green
* control the gripper with the "right_finger_joint" slider

You should see the left finger follow the right finger.


Reproducing the gravity bug
---------------------------

For fun, you can now pause the simulation, lift the model for about a meter or
so and resume the simulation. You should see the model dropping to the ground
normally.

Now, edit the file `launch/gazebo.launch` and comment out the following line:

```xml
  <rosparam file="$(find mimic_joint_gazebo_tutorial)/config/gazebo/gazebo_controller.yaml" command="load" />
```

Kill the launch file if it's still running and start it again:

```bash
roslaunch mimic_joint_gazebo_tutorial gazebo.launch
```

Repeat the dropping experiment. Your model should now drop very slowly with the
right finger (the one controlled by ros_control) up, as if dangling from a
parachute. Congratulations, you've been hit by the [gravity
bug](https://github.com/ros-simulation/gazebo_ros_pkgs/issues/612).

You will also note that the mimic joint plugin doesn't work any more. If you
move the right finger, the left doesn't follow any more. This is because the
mimic joint plugin expected us to specify PID parameters, and our change to
`gazebo.launch` caused the PID parameters not to be loaded.

If you want, you can edit the following line in the file `urdf/mimic_joint_gazebo_tutorial.urdf.xacro`:

```xml
  <xacro:mimic_joint_plugin_gazebo name_prefix="left_finger_joint"
    parent_joint="right_finger_joint" mimic_joint="left_finger_joint"
    has_pid="true" multiplier="1.0" max_effort="10.0" />
```

... and change `has_pid="true"` to `has_pid="false"`.

Now restart Gazebo and drop the model again. The mimic plugin now works, but is
also affected by the gravity bug. Note that you actually have to delete the
line at the moment, not set the value of hasPID to false.

In case you're wondering what the point is by now: The whole point was to
demonstrate why loading the Gazebo PID parameters is important to work around
the gravity bug.


Alternatives
------------

For ros_control, you can use a VelocityJointInterface or EffortJointInterface
instead of a PositionJointInterface. In that case, you don't need the Gazebo
PID parameters (but you'll need PID parameters for your ros_control velocity or
effort controller instead).

For the mimic joint plugin, loading the Gazebo PID gains is the only option AFAIK.
