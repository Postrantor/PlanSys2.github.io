---
tip: translate by openai@2023-06-16 08:49:53
...
---
title: The first planning package
---------------------------------

- [Overview](#overview)
- [Tutorial Steps](#tutorial-steps)
  - [0- Requisites](#0--requisites)
  - [1- Running the example](#1--running-the-example)
  - [2- Package structure](#2--package-structure)
  - [3- Actions implementation](#3--actions-implementation)
  - [4- Launcher](#4--launcher)

# Overview

It is common for PlanSys2 to have a package that contains all the elements to plan and execute in a domain:

> PlanSys2 通常有一个包，其中包含所有在一个领域中进行计划和执行所需的元素。

- A PDDL model.
- A node that implements each action, probably in different executables.
- A launcher that executes the actions and launches PlanSys2 with the appropriate domain.
- Possibly (not in this tutorial) a node that initiates knowledge and manages when to execute a plan and its goals.

> 可能（不在本教程中）存在一个节点，它可以发起知识，并管理何时执行计划及其目标。

In this tutorial we are going to run again and to analyze the package of the simple example shown in: ref: [getting_started]{.title-ref}.

> 在本教程中，我们将再次运行并分析参考[getting_started]{.title-ref}中展示的简单示例的包。

# Tutorial Steps

## 0- Requisites

Clone in your workspace and build the [examples](https://github.com/IntelligentRoboticsLabs/ros2_planning_system_examples)

> 克隆到你的工作空间并且构建[示例](https://github.com/IntelligentRoboticsLabs/ros2_planning_system_examples)

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
  ros2 launch plansys2_simple_example plansys2_simple_example_launch.py
  ```

  This launch PlanSys2 this [simple PDDL domain](https://github.com/IntelligentRoboticsLabs/ros2_planning_system_examples/blob/master/plansys2_simple_example/pddl/simple_example.pddl), and some processes that implements a fake version of the domain\'s actions.

> 这个发射 PlanSys2 这个[简单 PDDL 域](https://github.com/IntelligentRoboticsLabs/ros2_planning_system_examples/blob/master/plansys2_simple_example/pddl/simple_example.pddl)，以及一些实现域动作的虚拟版本的进程。

- Open a new terminal and run the terminal:

  ```bash
  ros2 run plansys2_terminal plansys2_terminal
  ```
- In the terminal shell, copy and paste the next commands to init the knowledge of the system and set the goal:

> 在终端 shell 中，复制并粘贴下面的命令来初始化系统的知识并设定目标：

```lisp
set instance leia robot
set instance entrance room
set instance kitchen room
set instance bedroom room
set instance dinning room
set instance bathroom room
set instance chargingroom room

set predicate (connected entrance dinning)
set predicate (connected dinning entrance)

set predicate (connected dinning kitchen)
set predicate (connected kitchen dinning)

set predicate (connected dinning bedroom)
set predicate (connected bedroom dinning)

set predicate (connected bathroom bedroom)
set predicate (connected bedroom bathroom)

set predicate (connected chargingroom kitchen)
set predicate (connected kitchen chargingroom)

set predicate (charging_point_at chargingroom)
set predicate (battery_low leia)
set predicate (robot_at leia entrance)

set goal (and(robot_at leia bathroom))
```

- In the terminal shell, run the plan:

  ```lisp
  run
  ```

In the PlanSys2 terminal you\'ll be able to see the plan. In both terminal, you\'ll see the current actions executed and its level of completion. As soon as the plan execution finished, the terminal will be available again.

> 在 PlanSys2 终端中，您可以查看计划。在两个终端中，您可以看到正在执行的当前操作及其完成程度。一旦计划执行完成，终端将再次可用。

## 2- Package structure

Go to `<your_workspace>/ros2_planning_system_examples/plansys2_simple_example`. This is the ROS2 package that contains the example of this tutorial. The structure is the usual of a ROS2 package, with a `package.xml` and a `CMakeLists.txt` as usual. Besides, we have the next directories:

> 转到 <your_workspace>/ros2_planning_system_examples/plansys2_simple_example。这是包含本教程示例的 ROS2 软件包。结构与通常的 ROS2 软件包相同，具有通常的 `package.xml` 和 `CMakeLists.txt`。此外，我们还有以下目录：

- **pddl** The directory with the PDDL file that contains the domain. It is the same of the ref: [terminal_usage]{.title-ref}.

> 目录中包含 PDDL 文件，其中包含域信息。它与参考文献[terminal_usage]{.title-ref}相同。

- **launch** It contains the launcher of this example:
- **src** It contains the implementation of the three actions of the domain.

## 3- Actions implementation

The actions in the domain are _move_, _charge_ and _ask_charge_. It will contain a \"fake\" implementation. Each node that implements an action is called _action performer_. Each action will take some seconds to execute, only incrementing a `progress` value and displaying it in the terminal. Each action in PlanSys2 is a [managed node](https://design.ros2.org/articles/node_lifecycle.html) , also known as _lifecycle node_. When active, it will iterativelly call a function to perform the job. Let\'s analyze the code of _move_ action in `src/move_action_node.cpp`:

> 在域中的行动是_move_、_charge_和_ask_charge_。它将包含一个“虚拟”实现。实现操作的每个节点称为_action performer_。每个操作将花费一些秒来执行，只增加一个 `progress` 值，并在终端中显示它。PlanSys2 中的每个操作都是一个[管理节点](https://design.ros2.org/articles/node_lifecycle.html)，也称为_lifecycle node_。当处于活动状态时，它将迭代调用一个函数来执行工作。让我们分析一下 `src/move_action_node.cpp` 中的_move_操作的代码：

```{.c++ linenos=""}
class MoveAction : public plansys2::ActionExecutorClient
{
public:
  MoveAction()
  : plansys2::ActionExecutorClient("move", 250ms)
  {
    progress_ = 0.0;
  }

private:
  void do_work()
  {
    if (progress_ < 1.0) {
      progress_ += 0.02;
      send_feedback(progress_, "Move running");
    } else {
      finish(true, 1.0, "Move completed");

      progress_ = 0.0;
      std::cout << std::endl;
    }

    std::cout << "\r\e[K" << std::flush;
    std::cout << "Moving ... [" << std::min(100.0, progress_ * 100.0) << "%]  " <<
      std::flush;
  }

  float progress_;
};

int main(int argc, char ** argv)
{
  rclcpp::init(argc, argv);
  auto node = std::make_shared<MoveAction>();

  node->set_parameter(rclcpp::Parameter("action", "move"));
  node->trigger_transition(lifecycle_msgs::msg::Transition::TRANSITION_CONFIGURE);

  rclcpp::spin(node->get_node_base_interface());

  rclcpp::shutdown();

  return 0;
}
```

- `MoveAction` is a `plansys2::ActionExecutorClient` (defined [here](https://github.com/IntelligentRoboticsLabs/ros2_planning_system/blob/master/plansys2_executor/include/plansys2_executor/ActionExecutorClient.hpp)), that inherit from `rclcpp_cascade_lifecycle::CascadeLifecycleNode`. This is, basically, a `rclcpp_lifecycle::LifecycleNode`, but with an additional property: it can activate in cascade another `rclcpp_cascade_lifecycle::CascadeLifecycleNode` when it is active. It\'s useful when an action requires to activate a node that process a sensor information. It will only be active while the action that requires its output is active. See [this repo](https://github.com/fmrico/cascade_lifecycle) for more info.

> MoveAction 是一个 plansys2::ActionExecutorClient（定义在此处：[https://github.com/IntelligentRoboticsLabs/ros2_planning_system/blob/master/plansys2_executor/include/plansys2_executor/ActionExecutorClient.hpp](https://github.com/IntelligentRoboticsLabs/ros2_planning_system/blob/master/plansys2_executor/include/plansys2_executor/ActionExecutorClient.hpp)），它继承自 rclcpp_cascade_lifecycle::CascadeLifecycleNode。这基本上是一个 rclcpp_lifecycle::LifecycleNode，但具有额外的属性：它可以在激活时级联激活另一个 rclcpp_cascade_lifecycle::CascadeLifecycleNode。当需要一个动作来激活一个处理传感器信息的节点时，这很有用。它只在需要其输出的动作处于活动状态时才处于活动状态。有关更多信息，请参阅此存储库：[https://github.com/fmrico/cascade_lifecycle](https://github.com/fmrico/cascade_lifecycle)。

```c++
: plansys2::ActionExecutorClient("move", 250ms)
```

This indicates that each 250ms (4Hz) the method `do_work()` will be called.

> 这表明每 250 毫秒（4Hz）就会调用方法 `do_work()`。

- `plansys2::ActionExecutorClient` has an API, with these relevant protected methods for the actions:

> plansys2::ActionExecutorClient 具有一个 API，具有与动作相关的受保护的方法：

```c++
const std::vector<std::string> & get_arguments();
void send_feedback(float completion, const std::string & status = "");
void finish(bool success, float completion, const std::string & status = "");
```

`get_arguments()` returns the list of arguments of an action. For example, when executing `(move leia chargingroom kitchen)`, it will return this vector of strings: `{"leia", "chargingroom", "kitchen"}`

> `get_arguments()` 返回一个动作的参数列表。例如，当执行 `（move leia chargingroom kitchen）` 时，它将返回以下字符串向量：`{"leia"，"chargingroom"，"kitchen"}`

This code is sending feedback of its completion. When finished, `finish` method indicates that the action has finished, and send back if it succesfully finished, the completion value in \[0-1\] and optional info. Then, the node will pass to inactive state.

> 这段代码正在发送完成反馈。完成后，`finish` 方法表明操作已完成，并发回成功完成的值[0-1]以及可选的信息。然后，节点将进入非活动状态。

```c++
if (progress_ < 1.0) {
  progress_ += 0.02;
  send_feedback(progress_, "Move running");
} else {
  finish(true, 1.0, "Move completed");

  progress_ = 0.0;
  std::cout << std::endl;
}
```

- The action node, once created, must pass to inactive state to be ready to execute.

  ```c++
  auto node = std::make_shared<MoveAction>();

  node->set_parameter(rclcpp::Parameter("action", "move"));
  node->trigger_transition(lifecycle_msgs::msg::Transition::TRANSITION_CONFIGURE);

  rclcpp::spin(node->get_node_base_interface());
  ```

The parameter `action` sets what action implements the `node` object.

> 参数 `action` 设置 `node` 对象实施的操作。

## 4- Launcher

The launcher must include the PlanSys2 launcher, selecting the domain, and it runs the executables that implements the PDDL actions:

> 发射器必须包括 PlanSys2 发射器，选择域，并运行实现 PDDL 动作的可执行文件：

```{.python linenos=""}
def generate_launch_description():
    example_dir = get_package_share_directory('plansys2_simple_example')
    namespace = LaunchConfiguration('namespace')

    declare_namespace_cmd = DeclareLaunchArgument(
        'namespace',
        default_value='',
        description='Namespace')

    plansys2_cmd = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(os.path.join(
            get_package_share_directory('plansys2_bringup'),
            'launch',
            'plansys2_bringup_launch_monolithic.py')),
        launch_arguments={
          'model_file': example_dir + '/pddl/simple_example.pddl',
          'namespace': namespace
          }.items())

    move_cmd = Node(
        package='plansys2_simple_example',
        executable='move_action_node',
        name='move_action_node',
        namespace=namespace,
        output='screen',
        parameters=[])

    charge_cmd = Node(
        package='plansys2_simple_example',
        executable='charge_action_node',
        name='charge_action_node',
        namespace=namespace,
        output='screen',
        parameters=[])

    ask_charge_cmd = Node(
        package='plansys2_simple_example',
        executable='ask_charge_action_node',
        name='ask_charge_action_node',
        namespace=namespace,
        output='screen',
        parameters=[])   # Create the launch description and populate
    ld = LaunchDescription()

    ld.add_action(declare_namespace_cmd)

    # Declare the launch options
    ld.add_action(plansys2_cmd)

    ld.add_action(move_cmd)
    ld.add_action(charge_cmd)
    ld.add_action(ask_charge_cmd)

    return ld
```
