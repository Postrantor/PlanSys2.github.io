---
tip: translate by openai@2023-06-16 08:47:32
...
---
title: Using a planning controller
----------------------------------

```{=html}
<h1 align="center">
  <div>
    <div style="position: relative; padding-bottom: 0%; overflow: hidden; max-width: 100%; height: auto;">
      <iframe width="450" height="300" src="https://www.youtube.com/embed/fAEGySqefwo" frameborder="1" allowfullscreen></iframe>
    </div>
  </div>
</h1>
```

- [Overview](#overview)
- [Tutorial Steps](#tutorial-steps)

# Overview

In the previous examples, we have started the knowledge of PlanSys2, and we have calculated and executed plans interactively using the terminal. Obviously, when PlanSys2 is embedded in an application, interactive execution is not convenient.

> 在前面的例子中，我们已经开始了 PlanSys2 的知识，并且我们已经使用终端交互式地计算和执行计划。显然，当 PlanSys2 被嵌入到应用程序中时，交互式执行不太方便。

A _planning controller_ is a program that: \* Initiates, consults, and updates knowledge. \* Sets goals. \* Makes requests to execute the plan.

> 一个计划控制器是一种程序，它：\* 启动、咨询和更新知识。\* 设定目标。\* 发出执行计划的请求。

This program controls the robot\'s mission. It is usually implemented as a finite state machine and decides what the next goal to be achieved is.

> 这个程序控制机器人的任务。它通常被实现为一个有限状态机，决定下一个要实现的目标是什么。

In this tutorial, we will integrate PlanSys2 and Nav2 to make a robot patrol its environment. There are 5 waypoints: `wp_control`, `wp_1`, `wp_2`, `wp_3`, and `wp_4`. Each one has different coordinates.

> 在本教程中，我们将集成 PlanSys2 和 Nav2，使机器人巡逻其环境。有 5 个航点：`wp_control`、`wp_1`、`wp_2`、`wp_3` 和 `wp_4`。每个航点都有不同的坐标。

- `wp_control` is the junction to which all other waypoints are connected. A patrol always goes through wp_control, since the waypoints are not connected to each other.

> wp_control 是所有其他航点连接的枢纽。巡逻队总是经过 wp_control，因为航点之间没有连接。

- Patrolling a waypoint consists of moving to the waypoint and, once there, rotating for a few seconds to perceive the environment.

> 巡逻一个航点包括移动到航点，一旦到达，旋转几秒钟以感知周围环境。

- During a patrol, all waypoints will be patrolled. Once patrolled, the patrol will begin again.

In the tutorial\'s first steps, we will use a test component that simulates the navigation process, and at the end of the tutorial, we will launch the real Nav2 and a simulator.

> 在教程的第一步中，我们将使用一个测试组件来模拟导航过程，在教程结束时，我们将启动真正的 Nav2 和模拟器。

# Tutorial Steps

## 0- Requisites

If you haven\'t done yet, clone in your workspace and build the [examples](https://github.com/IntelligentRoboticsLabs/ros2_planning_system_examples)

> 如果你还没有做，请在你的工作区克隆并构建[示例](https://github.com/IntelligentRoboticsLabs/ros2_planning_system_examples)。

```bash
cd <your_workspace>
git clone -b <ros2-distro>-devel https://github.com/IntelligentRoboticsLabs/ros2_planning_system_examples.git src
colcon build --symlink-install
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
```

## 1- Running the example

- Open a new terminal and run with a node that simulates de action of navigation. So, the execution of Nav2 will not take place in the first steps of this tutorial:

> 在新的终端中打开，并使用节点模拟导航操作。因此，Nav2 的执行不会在本教程的第一步中进行。

```bash
ros2 launch plansys2_patrol_navigation_example patrol_example_fakesim_launch.py
```

- Open a new terminal and run:

  ```bash
  ros2 run plansys2_patrol_navigation_example patrolling_controller_node
  ```

This will start the mission. You will see in the first terminal how the plans are calculated and how the actions are executed. In the last terminal you will see the execution of the actions too.

> 这将开始任务。你将在第一个终端看到计划如何被计算，以及行动如何被执行。在最后一个终端，你也将看到行动的执行。

## 2- Patrol action performer

The action `patrol` (`ros2_planning_system_examples/plansys2_patrol_navigation_example/src/patrol_action_node.cpp`) contains the actions that makes the robot spin.

> 行动 `patrol`（`ros2_planning_system_examples/plansys2_patrol_navigation_example/src/patrol_action_node.cpp`）包含使机器人旋转的动作。

```{.c++ linenos=""}
class Patrol : public plansys2::ActionExecutorClient
{
public:
  Patrol()
  : plansys2::ActionExecutorClient("patrol", 1s)
  {
  }

  rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface::CallbackReturn
  on_activate(const rclcpp_lifecycle::State & previous_state)
  {
    progress_ = 0.0;

    cmd_vel_pub_ = this->create_publisher<geometry_msgs::msg::Twist>("/cmd_vel", 10);
    cmd_vel_pub_->on_activate();

    return ActionExecutorClient::on_activate(previous_state);
  }

  rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface::CallbackReturn
  on_deactivate(const rclcpp_lifecycle::State & previous_state)
  {
    cmd_vel_pub_->on_deactivate();

    return ActionExecutorClient::on_deactivate(previous_state);
  }

private:
  void do_work()
  {
    if (progress_ < 1.0) {
      progress_ += 0.1;

      send_feedback(progress_, "Patrol running");

      geometry_msgs::msg::Twist cmd;
      cmd.linear.x = 0.0;
      cmd.linear.y = 0.0;
      cmd.linear.z = 0.0;
      cmd.angular.x = 0.0;
      cmd.angular.y = 0.0;
      cmd.angular.z = 0.5;

      cmd_vel_pub_->publish(cmd);
    } else {
      geometry_msgs::msg::Twist cmd;
      cmd.linear.x = 0.0;
      cmd.linear.y = 0.0;
      cmd.linear.z = 0.0;
      cmd.angular.x = 0.0;
      cmd.angular.y = 0.0;
      cmd.angular.z = 0.0;

      cmd_vel_pub_->publish(cmd);

      finish(true, 1.0, "Patrol completed");
    }
  }

  float progress_;

  rclcpp_lifecycle::LifecyclePublisher<geometry_msgs::msg::Twist>::SharedPtr cmd_vel_pub_;
};

int main(int argc, char ** argv)
{
  rclcpp::init(argc, argv);
  auto node = std::make_shared<Patrol>();

  node->set_parameter(rclcpp::Parameter("action", "patrol"));
  node->trigger_transition(lifecycle_msgs::msg::Transition::TRANSITION_CONFIGURE);

  rclcpp::spin(node->get_node_base_interface());

  rclcpp::shutdown();

  return 0;
}
```

This action runs with a frequency of 1Hz when activated. In each step, it increases its progress by 10% (line 32), sending speed commands to the robot through the topic `/cmd_vel` that make it spin (lines 36-44). When it completes the action, it stops the robot (lines 47-54). In each cycle it sends a feedback (line 34) or declares that the action has already finished (line 56).

> 这个动作在激活时以 1Hz 的频率运行。在每一步中，它会通过主题'/cmd_vel'向机器人发送速度指令（第 36-44 行），使其旋转，并将进度提高 10％（第 32 行）。当完成该动作时，它会停止机器人（第 47-54 行）。在每个周期中，它会发送反馈（第 34 行）或声明该动作已经完成（第 56 行）。

## 3- Move action performer

The action `move` (`ros2_planning_system_examples/plansys2_patrol_navigation_example/src/move_action_node.cpp`) contains the actions that sends navigation requests to Nav2 using the ROS2 action `navigate_to_pose`. This example is quite complex if you are not familiar with [ROS2 actions](https://index.ros.org/doc/ros2/Tutorials/Actions/Writing-a-Cpp-Action-Server-Client/), so we will not enter in details here, only selected pieces of code.

> 行动 `move`（`ros2_planning_system_examples/plansys2_patrol_navigation_example/src/move_action_node.cpp`）包含了使用 ROS2 动作 `navigate_to_pose` 向 Nav2 发送导航请求的行动。如果您不熟悉[ROS2 动作]（[https://index.ros.org/doc/ros2/Tutorials/Actions/Writing-a-Cpp-Action-Server-Client/](https://index.ros.org/doc/ros2/Tutorials/Actions/Writing-a-Cpp-Action-Server-Client/)），这个示例就会相当复杂，所以我们不会在这里进行详细介绍，只选择一些代码片段。

```{.c++ linenos=""}
MoveAction()
: plansys2::ActionExecutorClient("move", 500ms)
{
  geometry_msgs::msg::PoseStamped wp;
  wp.header.frame_id = "/map";
  wp.pose.position.x = 0.0;
  wp.pose.position.y = -2.0;
  wp.pose.position.z = 0.0;
  wp.pose.orientation.x = 0.0;
  wp.pose.orientation.y = 0.0;
  wp.pose.orientation.z = 0.0;
  wp.pose.orientation.w = 1.0;
  waypoints_["wp1"] = wp;

  wp.pose.position.x = 1.8;
  wp.pose.position.y = 0.0;
  waypoints_["wp2"] = wp;

  wp.pose.position.x = 0.0;
  wp.pose.position.y = 2.0;
  waypoints_["wp3"] = wp;

  wp.pose.position.x = -0.5;
  wp.pose.position.y = -0.5;
  waypoints_["wp4"] = wp;

  wp.pose.position.x = -2.0;
  wp.pose.position.y = -0.4;
  waypoints_["wp_control"] = wp;

  using namespace std::placeholders;
  pos_sub_ = create_subscription<geometry_msgs::msg::PoseWithCovarianceStamped>(
    "/amcl_pose",
    10,
    std::bind(&MoveAction::current_pos_callback, this, _1));
}

...
private:
  std::map<std::string, geometry_msgs::msg::PoseStamped> waypoints_;
```

The coordinates of each waypoint are initialized and inserted in `waypoints_`.

> 每个航点的坐标都被初始化并插入到 `waypoints_` 中。

```{.c++ linenos=""}
rclcpp_lifecycle::node_interfaces::LifecycleNodeInterface::CallbackReturn
on_activate(const rclcpp_lifecycle::State & previous_state)
{
  ...

  auto wp_to_navigate = get_arguments()[2];
  goal_pos_ = waypoints_[wp_to_navigate];
  navigation_goal_.pose = goal_pos_;

  ...

  send_goal_options.feedback_callback = [this](
    NavigationGoalHandle::SharedPtr,
    NavigationFeedback feedback) {
      send_feedback(
        std::min(1.0, std::max(0.0, 1.0 - (feedback->distance_remaining / dist_to_move))),
        "Move running");
    };

  send_goal_options.result_callback = [this](auto) {
      finish(true, 1.0, "Move completed");
    };

 ...
}
```

- If we execute the action `(move leia wp_control wp_1)`, the third argument (`get_arguments()[2]`) is the waypoint to navigate, `wp_1`. We use this id to get the coordinate from `waypoints_` for sending it in the Nav2 action.

> 如果我们执行动作 `(move leia wp_control wp_1)`，第三个参数（`get_arguments()[2]`）是要导航的航点，`wp_1`。我们使用这个 ID 从 `waypoints_` 中获取坐标，然后发送给 Nav2 动作。

- When receiving feedback from the Nav2 ROS2 action, we send feedback about the execution of the `move` action.

> 当收到来自 Nav2 ROS2 动作的反馈时，我们会发送有关执行“move”动作的反馈。

- When Nav2 considers the navigation complete, we call `finish` to finalize the execution of `move` action.

> 当 Nav2 认为导航完成时，我们调用 `finish` 来完成 `move` 动作的执行。

## 3- Mission controller

The mission controller (`ros2_planning_system_examples/plansys2_patrol_navigation_example/src/patrolling_controller_node.cpp`) initializes the knowledge in PlanSys2 and implements a FSM to run patrolling plans. It uses four PlanSys2 clients to interact with PlanSys2:

> 控制器（`ros2_planning_system_examples/plansys2_patrol_navigation_example/src/patrolling_controller_node.cpp`）初始化 PlanSys2 中的知识，并实现一个 FSM 来运行巡逻计划。它使用四个 PlanSys2 客户端与 PlanSys2 交互：

```c++
void init()
  {
    domain_expert_ = std::make_shared<plansys2::DomainExpertClient>(shared_from_this());
    planner_client_ = std::make_shared<plansys2::PlannerClient>(shared_from_this());
    problem_expert_ = std::make_shared<plansys2::ProblemExpertClient>(shared_from_this());
    executor_client_ = std::make_shared<plansys2::ExecutorClient>(shared_from_this());
    init_knowledge();
  }

private:
  std::shared_ptr<plansys2::DomainExpertClient> domain_expert_;
  std::shared_ptr<plansys2::PlannerClient> planner_client_;
  std::shared_ptr<plansys2::ProblemExpertClient> problem_expert_;
  std::shared_ptr<plansys2::ExecutorClient> executor_client_;
```

Using these clients, we can add instances and predicates to PlanSys2:

> 使用这些客户端，我们可以向 PlanSys2 添加实例和谓词。

```c++
void init_knowledge()
{
  problem_expert_->addInstance(plansys2::Instance{"r2d2", "robot"});
  problem_expert_->addInstance(plansys2::Instance{"wp_control", "waypoint"});
  problem_expert_->addInstance(plansys2::Instance{"wp1", "waypoint"});
  problem_expert_->addInstance(plansys2::Instance{"wp2", "waypoint"});
  problem_expert_->addInstance(plansys2::Instance{"wp3", "waypoint"});
  problem_expert_->addInstance(plansys2::Instance{"wp4", "waypoint"});

  problem_expert_->addPredicate(plansys2::Predicate("(robot_at r2d2 wp_control)"));
  problem_expert_->addPredicate(plansys2::Predicate("(connected wp_control wp1)"));
  problem_expert_->addPredicate(plansys2::Predicate("(connected wp1 wp_control)"));
  problem_expert_->addPredicate(plansys2::Predicate("(connected wp_control wp2)"));
  problem_expert_->addPredicate(plansys2::Predicate("(connected wp2 wp_control)"));
  problem_expert_->addPredicate(plansys2::Predicate("(connected wp_control wp3)"));
  problem_expert_->addPredicate(plansys2::Predicate("(connected wp3 wp_control)"));
  problem_expert_->addPredicate(plansys2::Predicate("(connected wp_control wp4)"));
  problem_expert_->addPredicate(plansys2::Predicate("(connected wp4 wp_control)"));
}
```

In the `step` method (called at 5Hz) there is the implementation of the FSM. Each switch case contains: \* The code to execute in the state

> 在每秒 5 次调用的 `step` 方法中，实现了有限状态机，每个 `switch case` 包含：* 在该状态下要执行的代码

```c++
case PATROL_WP1:
  {
    auto feedback = executor_client_->getFeedBack();

    for (const auto & action_feedback : feedback.action_execution_status) {
      std::cout << "[" << action_feedback.action << " " <<
        action_feedback.completion * 100.0 << "%]";
    }
```

- The condition to transitate to another state and the code executed before the start of the new state. The important part here is how we set the new goal for the new state (using `setGoal`), how we compute a new plan, and how we call the executor to execute the plan to achieve it (using the non-blocking call `executePlan`):

> 迁移到另一个状态的条件以及新状态开始前执行的代码。这里重要的是我们如何为新状态设定新的目标（使用 `setGoal`），如何计算新的计划，以及如何调用执行器来执行计划以实现它（使用非阻塞调用 `executePlan`）：

```c++
if (executor_client_->getResult().value().success) {
  std::cout << "Plan successful finished " << std::endl;

  // Cleanning up
  problem_expert_->removePredicate(plansys2::Predicate("(patrolled wp1)"));

  // Set the goal for next state
  problem_expert_->setGoal(plansys2::Goal("(and(patrolled wp2))"));

  // Compute the plan
  auto domain = domain_expert_->getDomain();
  auto problem = problem_expert_->getProblem();
  auto plan = planner_client_->getPlan(domain, problem);

  // Execute the plan
  if (executor_client_->executePlan(plan.value())) {
    state_ = PATROL_WP2;
  }
} else {
  ...
}
```

## 4- Running the example with Nav2

- Make sure that Nav2 is [correctly installed and it is working](https://navigation.ros.org/getting_started/index.html).

> 确保 Nav2 已正确安装并且正常工作（参见：[https://navigation.ros.org/getting_started/index.html](https://navigation.ros.org/getting_started/index.html)）。

- Open a new terminal and run with a node that simulates de action of navigation. So, the execution of Nav2 will not take place in the first steps of this tutorial:

> 打开一个新的终端，并使用节点模拟导航的动作。因此，Nav2 的执行不会在本教程的第一步中进行。

```bash
ros2 launch plansys2_patrol_navigation_example patrol_example_launch.py
```

- Open a new terminal and run:

  ```bash
  ros2 run plansys2_patrol_navigation_example patrolling_controller_node
  ```
