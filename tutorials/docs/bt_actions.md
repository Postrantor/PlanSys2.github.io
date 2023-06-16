---
tip: translate by openai@2023-06-16 08:45:48
title: Implement actions as Behavior Trees
...

[](https://www.youtube.com/embed/_oCcIq-TN_0)

- [Overview](#overview)
- [Tutorial Steps](#tutorial-steps)

# Overview

[Behavior Trees](<https://en.wikipedia.org/wiki/Behavior_tree_(artificial_intelligence,_robotics_and_control)>) are a great way to implement actions. Instead of implementing an action, we specify an xml file with the structure of a behavior Tree and implement the tree nodes. This allows you to easily modify the implementation of an action, and reuse the nodes that are part of it in different behavior trees.

> 行为树是实现行动的一个很好的方式。我们不是实现一个行动，而是使用具有行为树结构的 XML 文件来指定并实现树节点。这样可以轻松地修改行动的实现，并在不同的行为树中重复使用其中的节点。

In PlanSys2 we will use the [BehaviorTree.CPP](https://www.behaviortree.dev/) package from [Davide Faconti](https://github.com/facontidavide). Read the tutorials there to learn more control nodes and so more sohisticated actions.

> 在 PlanSys2 中，我们将使用来自 [Davide Faconti] 的 [BehaviorTree.CPP] 包。阅读那里的教程以了解更多的控制节点以及更多的复杂动作。

In this tutorial we are going to see an example of a car assembly factory. The robots in this factory must transport the parts of a car from the areas where they are stored to the assembly area, in order to assemble a car.

> 在本教程中，我们将看到一个汽车装配厂的例子。这个工厂里的机器人必须把汽车的零件从存储区域运送到装配区域，以便组装汽车。

![image](images/bt_actions/carfactory.png){.align-center width="600px"}

You can find the PDDL domain of this tutorial [in this file](https://github.com/IntelligentRoboticsLabs/ros2_planning_system_examples/blob/master/plansys2_bt_example/pddl/bt_example.pddl).

> 你可以在这个文件中找到本教程的 PDDL 领域：[bt_example.pddl]。

# Tutorial Steps

## 0- Requisites

If you haven\'t done yet, clone in your workspace and build the [examples](https://github.com/IntelligentRoboticsLabs/ros2_planning_system_examples)

```bash
cd <your_workspace>
git clone -b <ros2-distro>-devel https://github.com/IntelligentRoboticsLabs/ros2_planning_system_examples.git src
colcon build --symlink-install
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
```

## 1- Running the example

- Open a new terminal and run :

  ```bash
  ros2 launch plansys2_bt_example plansys2_bt_example_launch.py
  ```

- You have two options: you can use the real Nav2 package with a simulation or using a Nav2 fake node:

> 你有两个选择：你可以使用真正的 Nav2 包与模拟，或者使用 Nav2 假节点。

```bash
ros2 launch nav2_bringup tb3_simulation_launch.py  # For real Nav2 and Simulation
ros2 run plansys2_bt_example nav2_sim_node  # For fake Nav2 node
```

- In the terminal shell, copy and paste the next commands to init the knowledge of the system and set the goal:

> 在终端壳中，复制并粘贴下面的命令来初始化系统知识并设定目标：

```lisp
set instance r2d2 robot

set instance wheels_zone zone
set instance steering_wheels_zone zone
set instance body_car_zone zone
set instance assembly_zone zone
set instance recharge_zone zone

set instance wheel_1 piece
set instance body_car_1 piece
set instance steering_wheel_1 piece

set instance wheel_2 piece
set instance body_car_2 piece
set instance steering_wheel_2 piece

set instance wheel_3 piece
set instance body_car_3 piece
set instance steering_wheel_3 piece

set instance car_1 car
set instance car_2 car
set instance car_3 car

set predicate (robot_available r2d2)
set predicate (robot_at r2d2 wheels_zone)
set predicate (is_assembly_zone assembly_zone)
set predicate (is_recharge_zone recharge_zone)

set predicate (piece_at wheel_1 wheels_zone)
set predicate (piece_at body_car_1 body_car_zone)
set predicate (piece_at steering_wheel_1 steering_wheels_zone)

set predicate (piece_is_wheel wheel_1)
set predicate (piece_is_body_car body_car_1)
set predicate (piece_is_steering_wheel steering_wheel_1)
set predicate (piece_not_used wheel_1)
set predicate (piece_not_used body_car_1)
set predicate (piece_not_used steering_wheel_1)

set goal (and(car_assembled car_1))
run
```

## 2- Using Behavior Trees

![image](images/bt_actions/demoDiag2.png){.align-center width="600px"}

In the PDDL domain we have three actions: assemble, move and transport. We will implement the assemble action without using BTs, as we have done in the previous tutorials. We will implement the other two PDDL actions, `Move` and `Transport`, using BTs. For this, each action will have a BT encoded in an XML file that. This XML file contains the control structures (sequences, fallbacks, \...) and the nodes that carry out the action. In this tutorial, 4 BT nodes have been implemented (`Move`, `ApproachObject`, `OpenGripper`, and `CloseGripper`) that can be included in the BTs of the PDDL actions.

> 在 PDDL 领域中，我们有三个动作：组装、移动和运输。我们将不使用 BT 来实现组装动作，就像我们在之前的教程中所做的那样。我们将使用 BT 来实现另外两个 PDDL 动作`Move`和`Transport`。为此，每个动作将有一个以 XML 文件编码的 BT。该 XML 文件包含控制结构(序列、回退等)和执行动作的节点。在本教程中，实现了 4 个 BT 节点(`Move`、`ApproachObject`、`OpenGripper`和`CloseGripper`)，可以包含在 PDDL 动作的 BT 中。

In PlanSys2 there is a generic executable that any BT can run. This executable is `bt_action_node` from the package `plansys2_bt_actions`. To use it, we should add it in the launch file, specifying as parametersthe XML file that contains the BT and the PDDL action name:

> 在 PlanSys2 中有一个通用的可执行文件，任何行为树(BT)都可以运行它。这个可执行文件是来自包 `plansys2_bt_actions` 的 `bt_action_node`。要使用它，我们应该在启动文件中添加它，指定包含 BT 和 PDDL 动作名称的 XML 文件作为参数：

```python
move_cmd = Node(
    package='plansys2_bt_actions',
    executable='bt_action_node',
    name='move',
    namespace=namespace,
    output='screen',
    parameters=[
      example_dir + '/config/params.yaml',
      {
        'action_name': 'move',
        'bt_xml_file': example_dir + '/behavior_trees_xml/move.xml'
      }
    ])
```

The `params.yaml` contains the BT nodes that will be used in the BT. As we implement each BT as plugins, it is necessary to specify each custom node used. We can also include any parameter needed by the specific BT node. In this case, the coordinates of each room is specified to the BT node `move` using parameters:

> `params.yaml` 包含将在 BT 中使用的 BT 节点。由于我们将每个 BT 实现为插件，因此有必要指定每个自定义节点。我们还可以包括特定 BT 节点所需的任何参数。在这种情况下，使用参数将每个房间的坐标指定给 BT 节点'move'：

```yaml
move:
  ros__parameters:
    plugins:
      - plansys2_move_bt_node
    waypoints:
      ["wheels_zone", "steering_wheels_zone", "body_car_zone", "assembly_zone"]
    waypoint_coords:
      wheels_zone: [0.0, -2.0, 0.0]
      steering_wheels_zone: [1.8, 0.0, 0.0]
      body_car_zone: [0.0, 2.0, 0.0]
      assembly_zone: [-2.0, -0.4, 0.0]
```

### 2.1 PDDL Action move

This is the BT for the **move** action. The sequence is not necessary, but it is mantained in the tutorial to be coherent with the available code. It is composed only by one BT node `Move` (do not confuse with PDDL action `move`).

> 这是**移动**行动的行为树(BT)。序列不是必需的，但是为了与可用代码保持一致，在教程中保留了它。它仅由一个 BT 节点 `Move`(不要与 PDDL 动作 `move` 混淆)组成。

```xml
<root main_tree_to_execute = "MainTree" >
    <BehaviorTree ID="MainTree">
       <Sequence name="root_sequence">
           <Move    name="move" goal="${arg2}"/>
       </Sequence>
    </BehaviorTree>
</root>
```

In PlanSys2, the arguments of the PDDL action are accesible in the XML throught values in the blackboard whise identififiers are `arg0`, `arg1`, `arg2`, and so on. If executing the PDDL action `(move r2d2 corridor kitchen)`, `arg2` is the destiny room `kitchen`.

> 在 PlanSys2 中，PDDL 操作的参数可以通过 XML 中标识符为 `arg0`，`arg1`，`arg2` 等的黑板上的值来访问。如果执行 PDDL 操作 `(move r2d2 corridor kitchen)`，`arg2` 就是目的地房间 `kitchen`。

### 2.2 PDDL Action transport

This is the BT for the **transport** action. It is implemented as a sequence of BT nodes, including reuse the BT node `Move`.

> 这是用于**运输**操作的行为树(BT)。它实现为一系列 BT 节点，其中包括重用 BT 节点“Move”。

```xml
<root main_tree_to_execute = "MainTree" >
    <BehaviorTree ID="MainTree">
       <Sequence name="root_sequence">
           <OpenGripper    name="open_gripper"/>
           <ApproachObject name="approach_object"/>
           <CloseGripper   name="close_gripper"/>
           <Move    name="move" goal="${arg3}"/>
           <OpenGripper    name="open_gripper"/>
       </Sequence>
    </BehaviorTree>
</root>
```

### 2.3 BT Nodes

We implemented 4 BT nodes for this tutorial. `ApproachObject`, `OpenGripper`, and `CloseGripper` have a similar implementation, showing only a message in the terminal when executing:

> 我们为本教程实施了 4 个 BT 节点。 `ApproachObject`、`OpenGripper` 和 `CloseGripper` 的实现类似，只在执行时在终端显示一条消息。

```cpp
class ApproachObject : public BT::ActionNodeBase
{
public:
  explicit ApproachObject(
    const std::string & xml_tag_name,
    const BT::NodeConfiguration & conf);

  void halt();
  BT::NodeStatus tick();

  static BT::PortsList providedPorts()
  {
    return BT::PortsList({});
  }

private:
  int counter_;
};
```

```cpp
ApproachObject::ApproachObject(
  const std::string & xml_tag_name,
  const BT::NodeConfiguration & conf)
: BT::ActionNodeBase(xml_tag_name, conf), counter_(0)
{
}

void
ApproachObject::halt()
{
  std::cout << "ApproachObject halt" << std::endl;
}

BT::NodeStatus
ApproachObject::tick()
{
  std::cout << "ApproachObject tick " << counter_ << std::endl;

  if (counter_++ < 5) {
    return BT::NodeStatus::RUNNING;
  } else {
    counter_ = 0;
    return BT::NodeStatus::SUCCESS;
  }
}

}  // namespace plansys2_bt_example

#include "behaviortree_cpp_v3/bt_factory.h"
BT_REGISTER_NODES(factory)
{
  factory.registerNodeType<plansys2_bt_example::ApproachObject>("ApproachObject");
}
```

For implementing the BT node `Move`, we make it to inherit from `BtActionNode<>` to simplify the implementation when the node calls a ROS2 action. In this case, `nav2_msgs::action::NavigateToPose`.

> 为了实现 BT 节点“Move”，我们将它从“BtActionNode<>”继承，以简化节点调用 ROS2 动作时的实现。在这种情况下，“nav2_msgs :: action :: NavigateToPose”。

```cpp
class Move : public plansys2::BtActionNode<nav2_msgs::action::NavigateToPose>
{
public:
  explicit Move(
    const std::string & xml_tag_name,
    const std::string & action_name,
    const BT::NodeConfiguration & conf);

  BT::NodeStatus on_tick() override;
  BT::NodeStatus on_success() override;

  static BT::PortsList providedPorts()
  {
    return {
      BT::InputPort<std::string>("goal")
    };
  }

private:
  int goal_reached_;
  std::map<std::string, geometry_msgs::msg::Pose2D> waypoints_;
};
```

`BtActionNode<>` hides all the complexity, and only it is necessary to implement the method `on_tick`, called each time the BT node is ticked. In this BT node, we get the destination id from the input `goal` parameter. Remember the line in the XML `<Move    name="move" goal="${arg3}"/>`.

> BTActionNode<> 隐藏了所有的复杂性，只需要实现方法 `on_tick`，每次 BT 节点被 tick 时调用。在这个 BT 节点中，我们从输入参数 `goal` 中获取目标 ID。记住 XML 中的行 `<Move    name="move" goal="${arg3}"/>`。

```cpp
BT::NodeStatus
Move::on_tick()
{
  if (status() == BT::NodeStatus::IDLE) {
    rclcpp::Node::SharedPtr node;
    config().blackboard->get("node", node);

    std::string goal;
    getInput<std::string>("goal", goal);

    geometry_msgs::msg::Pose2D pose2nav;
    if (waypoints_.find(goal) != waypoints_.end()) {
      pose2nav = waypoints_[goal];
    } else {
      std::cerr << "No coordinate for waypoint [" << goal << "]" << std::endl;
    }

    geometry_msgs::msg::PoseStamped goal_pos;

    goal_pos.header.frame_id = "map";
    goal_pos.header.stamp = node->now();
    goal_pos.pose.position.x = pose2nav.x;
    goal_pos.pose.position.y = pose2nav.y;
    goal_pos.pose.position.z = 0;
    goal_pos.pose.orientation = tf2::toMsg(tf2::Quaternion({0.0, 0.0, 1.0}, pose2nav.theta));

    goal_.pose = goal_pos;
  }

  return BT::NodeStatus::RUNNING;
}
```

There are also more method that we can implement if needed (`on_success`, `on_aborted`, `on_cancelled`, and so on).

> 也有更多的方法可以实施，如果需要的话(`on_success`、`on_aborted`、`on_cancelled` 等)。
