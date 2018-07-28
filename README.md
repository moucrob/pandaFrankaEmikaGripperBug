# pandaFrankaEmikaGripperBug
If you could debug me it'd be great thanks in advance

## Wiring
I'm sorry but I assume if you want to reproduce this you already have setted up everything using the
[franka emika FCI](https://frankaemika.github.io/docs/overview.html).

What happened to me is that while patching the existing linux kernel in order for him to become real time efficient, I "broke" 
my display (NVidia GPU was not anymore able to do the job) and our engineer managed to reestablish it using Nouveau graphical
driver and all these stuff. So in the end RViz was not launchable on the workstation and we had to connect a laptop, in order to
see RViz on the laptop, and then use MoveIt on the same device and send orders to the workstation orders are further sent to the
panda controller.

Anyway here is the wiring : one should :
- Get a router
- Plug internet source to the router
- plug an ethernet cable between the router and the workstation (for internet)
- plug an ehternet cable between the workstation and the controller (with parameters of the FCI mentionned above, so that the
network remains only local)
- plug an ethernet cable between the router and the laptop (mainly for sending orders from the lap to the wstation)
- the big black cable provided is plugged between the controller and the robot arm X1 port (no ethernet is plugged near the
emergency button plug on the arm)

## Turning ON the robot :
- turn on the controller (wait until the yellow leds stop to struggle and become permanent)
- pull the emergency button to the top (above a yellow line)
- unlock the brackets by clicking on the bottom-right of the web interface (accessible by the fixed robot IP you chose, see FCI above)
- put the repository here above named catkin_ws into your ~/home
- in the worksation shell, `gedit ~/.bashrc &`, then (I assume you sourced that catkin_ws' devel/...) add at the end :
```
export ROS_IP=`ifconfig eth1|awk '/inet addr/{print substr($2, 6)}'`
export ROS_MASTER_URI=http://$ROS_IP:11311 
```
- in the laptop one, after putted the panda_ws repo in its ~/home, add at its end :
```
source ~/panda_ws/devel/setup.bash
export ROS_IP=`hostname -I | cut -d' ' -f1`
export ROS_MASTER_URI=http://<the ip of the workstation associated with the internet source>:11311
```
- then open a new terminal in the workstation, and, assuming the local ip of the controller is 172.16.0.2,
  - tab 1 : `source ~/catkin_ws/devel/setup.bash && roslaunch franka_control franka_control.launch robot_ip:=172.16.0.2 load_gripper:=true`
  - tab 2 : `source ~/catkin_ws/devel/setup.bash && roslaunch franka_gripper franka_gripper.launch robot_ip:=172.16.0.2`
  - tab 3 : `source ~/catkin_ws/devel/setup.bash && roslaunch panda_moveit_config panda_moveit.launch controller:=position load_gripper:=true` (there is
  also effort controller but it doesn't change anything apparetnly here)
  - tab 4 : `source ~/catkin_ws/devel/setup.bash && roslaunch franka_example_controllers move_to_start.launch robot_ip:=172.16.0.2` but this
  shell is only to use in case where the robot is stucked because some error. For example ctrl+C the gripper node (shell 2), push down
  the emergency button, close the bracket, reopen the button, reopen the bracket, launch this command in tab4 (it will shut
  off the 3 previous tabs, and relaunch them one after all
- then open a new terminal in the laptop, and :
  - tab 1 : `source ~/panda_ws/devel/setup.bash && roslaunch panda_moveit_config moveit_rviz.launch`
  - tab 2 : `source ~/panda_ws/devel/setup.bash && roslaunch moveit_tutorials move_group_tet.launch` (main program, please be aware that the << cliq next >>
  are due to be done within RViz on the bottom left, no return entry in the bash will work. Also you might to need to manually add,
  by clicking on the top left of RViz, the RvizToolsVisualGui pannel to be able to click on next)
  - tab 3 : prepare your `cd ~/panda_ws && catkin build` and run it after each change into a c++ snippet
  
## See my problem (and fix it please if you can)
After the second commandline in the laptop you should see this on RViz

![ezgif com-optimize](https://user-images.githubusercontent.com/33000071/43357596-17842d32-92bf-11e8-90d3-9f477722f5c9.gif)

So what does not move is the real robot, even though I change the gripper width in move_group_test.cpp. Actually the real movement
goes down toward the target and continue to close the gripper further than the desired conf, and then the movement stops, whereas on the simulation
we can see the following step : the post_grasp retreat. Also I tried to
change the epsilon in the .yaml config file about the gripper but does not seem to fix the problem.
And what moves on the gif is the simulated robot, who indeed does what I programm...

# Could you know why I can't manage to not ending up with a completely closed gripper? Thank you in advance
