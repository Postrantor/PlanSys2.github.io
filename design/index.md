---
tip: translate by openai@2023-06-16 08:43:18
title: PlanSys2 design
...

![image](images/plansys2_arch.png){.align-center width="800px"}

PlanSys2 has a modular design. It is basically composed of 4 nodes:

> PlanSys2 具有模块化设计。它基本上由 4 个节点组成:

- **Domain Expert**: Contains the PDDL model information (types, predicates, functions, and actions).
- **Problem Expert**: Contains the current instances, predicates, functions, and goals that compose the model.
- **Planner**: Generates plans (sequence of actions) using the information contained in the Domain and Problem Experts.
- **Executor**: Takes a plan and executes it by activating the _action performers_ (the ROS2 nodes that implement each action).

> - **领域专家**：包含 PDDL 模型信息(类型、谓词、函数和动作)。
> - **问题专家**：包含构成模型的当前实例、谓词、函数和目标。
> - **规划师**：利用领域和问题专家所包含的信息，生成计划(行动序列)。
> - **Executor**：根据计划激活执行动作的 Executor(实现每个动作的 ROS2 节点)来执行计划。

Each of these nodes exposes its functionality using ROS2 services. Even so, in PlanSys2 we have created a client library that can be used in any application and hides the complexity of using ROS2 services.

> 每个节点都使用 ROS2 服务来公开其功能。尽管如此，在 PlanSys2 中，我们创建了一个可以在任何应用程序中使用的客户端库，可以隐藏使用 ROS2 服务的复杂性。

![image](images/plansys2_clients.png){.align-center width="500px"}

# 1. Domain Expert

The objective of the Domain Expert is to read PDDL domains from files and make them available to the rest of the components. It\'s static (by now).

> 目标领域专家的目标是从文件中读取 PDDL 领域并使其对其余组件可用。它目前是静态的。

## Parameters

- `model_file` \[string\]: PDDL model files to load, separates by \":\". These models will be merged. This allows for a modular application in which each component/package contributes with part of the PDDL and the action implementation. See [plansys2_multidomain_example](https://github.com/IntelligentRoboticsLabs/ros2_planning_system_examples/tree/master/plansys2_multidomain_example) for more details.

> -`model_file` \[字符串\]: 要加载的 PDDL 模型，用“：”分隔。这些模型将被合并。这允许模块化的应用，其中每个组件/包都提供部分 PDDL 和动作实现。有关详细信息，请参阅[plansys2_multidomain_example]。

## Client API

```cpp
std::vector<std::string> getTypes()
std::vector<plansys2::Predicate> getPredicates()
std::optional<plansys2::Predicate> getPredicate(const std::string & predicate)
std::vector<plansys2::Function> getFunctions()
std::optional<plansys2::Function> getFunction(const std::string & function)
std::vector<std::string> getActions()
plansys2_msgs::msg::Action::SharedPtr getAction(const std::string & action)
std::vector<std::string> getDurativeActions()
plansys2_msgs::msg::DurativeAction::SharedPtr getDurativeAction(const std::string & action)
std::string getDomain()
```

## Services

- `domain_expert/get_domain` \[plansys2_msgs::srv::GetDomain\]: Get the domain.
- `domain_expert/get_domain_types` \[plansys2_msgs::srv::GetDomainTypes\]: Get the valid types.
- `domain_expert/get_domain_actions` \[plansys2_msgs::srv::GetDomainActions\]: Get the available actions.
- `domain_expert/get_domain_action_details` \[plansys2_msgs::srv::GetDomainActionDetails\]: Get the details of a specific action.
- `domain_expert/get_domain_durative_actions` \[plansys2_msgs::srv::GetDomainDurativeActions\]: Get the available durative actions.
- `domain_expert/get_domain_durative_action_details` \[plansys2_msgs::srv::GetDomainDurativeActionDetails\]: Get the details of a specific durative action.
- `domain_expert/get_domain_predicates` \[plansys2_msgs::srv::GetDomainPredicates\]: Get the valid predicates.
- `domain_expert/get_domain_predicate_details` \[plansys2_msgs::srv::GetDomainPredicateDetails\]: Get the details of a specific predicate.
- `domain_expert/get_domain_functions` \[plansys2_msgs::srv::GetDomainFunctions\]: Get the valid functions.
- `domain_expert/get_domain_function_details` \[plansys2_msgs::srv::GetDomainFunctionDetails\]: Get the details of a specific function.
- `domain_expert/get_domain` \[plansys2_msgs::srv::GetDomain\]: Set the domain as a string.

## Publishers / Subscriber

None

# 2. Problem Expert

Contains the knowledge of the system: instances, grounded predicates and functions, and goals.

> 包含系统知识：实例、接地谓词和函数以及目标。

## Parameters

- `model_file` \[string\]: PDDL model files to load, separates by \":\". These models will be merged. This allows for a modular application in which each component/package contributes with part of the PDDL and the action implementation. See [plansys2_multidomain_example](https://github.com/IntelligentRoboticsLabs/ros2_planning_system_examples/tree/master/plansys2_multidomain_example) for more details.

> - `model_file` [字符串]: 加载 PDDL 模型文件，用“：”分隔。这些模型将被合并。这允许模块化应用程序，其中每个组件/包贡献部分 PDDL 和动作实现。有关更多详细信息，请参阅[plansys2_multidomain_example]。

## Client API

```cpp
std::vector<plansys2::Instance> getInstances();
bool addInstance(const plansys2::Instance & instance);
bool removeInstance(const plansys2::Instance & instance);
std::optional<plansys2::Instance> getInstance(const std::string & name);

std::vector<plansys2::Predicate> getPredicates();
bool addPredicate(const plansys2::Predicate & predicate);
bool removePredicate(const plansys2::Predicate & predicate);
bool existPredicate(const plansys2::Predicate & predicate);
std::optional<plansys2::Predicate> getPredicate(const std::string & predicate);

std::vector<plansys2::Function> getFunctions();
bool addFunction(const plansys2::Function & function);
bool removeFunction(const plansys2::Function & function);
bool existFunction(const plansys2::Function & function);
bool updateFunction(const plansys2::Function & function);
std::optional<plansys2::Function> getFunction(const std::string & function);

plansys2::Goal getGoal();
bool setGoal(const plansys2::Goal & goal);
bool isGoalSatisfied(const plansys2::Goal & goal);

bool clearGoal();
bool clearKnowledge();

std::string getProblem();
```

## Services

- `problem_expert/add_problem_goal` \[plansys2_msgs::srv::AddProblemGoal\]: Replace the goal.
- `problem_expert/add_problem_instance` \[plansys2_msgs::srv::AffectParam\]: Add an instance.
- `problem_expert/add_problem_predicate` \[plansys2_msgs::srv::AffectNode\]: Add a predicate.
- `problem_expert/add_problem_function` \[plansys2_msgs::srv::AffectNode\]: Add a function.
- `problem_expert/get_problem_goal` \[plansys2_msgs::srv::GetProblemGoal\]: Get the current goal.
- `problem_expert/get_problem_instance` \[plansys2_msgs::srv::GetProblemInstanceDetails\]: Get the details of an instance.
- `problem_expert/get_problem_instances` \[plansys2_msgs::srv::GetProblemInstances\]: Get all the instances.
- `problem_expert/get_problem_predicate =` \[plansys2_msgs::srv::GetNodeDetails\]: Get the details of a predicate.
- `problem_expert/get_problem_predicates` \[plansys2_msgs::srv::GetStates\]: Get all the predicates.
- `problem_expert/get_problem_function =` \[plansys2_msgs::srv::GetNodeDetails\]: Get the details of a function.
- `problem_expert/get_problem_functions` \[plansys2_msgs::srv::GetStates\]: Get all the functions.
- `problem_expert/get_problem` \[plansys2_msgs::srv::GetProblem\]: Get the PDDL problem as a string.
- `problem_expert/remove_problem_goal` \[plansys2_msgs::srv::RemoveProblemGoal\]: Remove the current goal.
- `problem_expert/remove_problem_instance` \[plansys2_msgs::srv::AffectParam\]: Remove an instance.
- `problem_expert/remove_problem_predicate` \[plansys2_msgs::srv::AffectNode\]: Remove a predicate.
- `problem_expert/remove_problem_function` \[plansys2_msgs::srv::AffectNode\]: Remove a function.
- `problem_expert/clear_problem_predicate` \[plansys2_msgs::srv::ClearProblemKnowledge\]: Clears the instances, predicates, and functions.
- `problem_expert/exist_problem_predicate` \[plansys2_msgs::srv::ExistNode\]: Check if a predicate exists.
- `problem_expert/exist_problem_function` \[plansys2_msgs::srv::ExistNode\]: Check if a function exists.
- `problem_expert/update_problem_function` \[plansys2_msgs::srv::AffectNode\]: Update a function value.
- `problem_expert/is_problem_goal_satisfied` \[plansys2_msgs::srv::IsProblemGoalSatisfied\]: Check if a goal is satisfied.

## Publishers / Subscriber

- `problem_expert/update_notify` \[std_msgs::msg::Empty\] {Publisher: rclcpp::QoS(100)}: A message is published on this topic when any element of the problem changes.
- `problem_expert/knowledge` \[plansys2_msgs::msg::Knowledge\] {Publisher: rclcpp::QoS(100)}: A message is published on this topic when any element of the problem changes.

> 在这个主题上发布消息时，当问题的任何元素发生变化时，`problem_expert/update_notify` \[std_msgs::msg::Empty\] {Publisher: rclcpp::QoS(100)} 将发布消息。
> 当问题的任何元素发生变化时，将在此主题上发布消息(发布者：rclcpp::QoS(100))：problem_expert/knowledge[plansys2_msgs::msg::Knowledge]。

# 3. Planner

This component calculates the plan to obtain a goal.

> 这个组件计算达到目标的计划。

1. A plan may be requested by providing a domain acquired from the Domain Expert and a problem acquired from the Problem expert.

> 可以通过从领域专家获取的域和从问题专家获取的问题来请求一个计划。

2. The domain is stored in `/tmp/<node namespace>/domain.pddl`. This allows for several PlanSys2 instances in the same machine, which is useful for simulating multiple robots in the same machine.

> 存储域位于`/tmp/<节点命名空间>/domain.pddl`。这样可以在同一台机器上运行多个 PlanSys2 实例，这对于在同一台机器上模拟多个机器人非常有用。

3. The problem is stored in `/tmp/<node namespace>/problem.pddl`.

> 问题存储在`/tmp/<节点命名空间>/problem.pddl`中。

4. Run the PDDL Solver, storing the output in `tmp/<node namespace>/plan.pddl`.

> 运行 PDDL 求解器，将输出存储在'tmp/<节点命名空间>/plan.pddl'中。

5. Parse `tmp/<node namespace>/plan.pddl` to get the sequence of actions as a vector of string.

> 解析`tmp/<节点命名空间>/plan.pddl`，以获取动作序列作为字符串向量。 6. Return the result.

Each PDDL solver in PlanSys2 is a plugin. By default PlanSys2 uses [POPF](https://github.com/IntelligentRoboticsLabs/ros2_planning_system/tree/master/plansys2_popf_plan_solver), although other PDDL solvers can be used easily. Currently, the [Temporal Fast Downward](https://github.com/IntelligentRoboticsLabs/plansys2_tfd_plan_solver) is also available.

> 每个 PDDL 求解器在 PlanSys2 中都是一个插件。默认情况下，PlanSys2 使用[POPF]，尽管可以很容易地使用其他 PDDL 求解器。目前，还可以使用[时间快速下降]。

![image](images/plansys2_planner.png){.align-center width="300px"}

## Parameters

- `plan_solver_plugins` \[vector\<string\>\]: List of PDDL solver plugins. Currently, only the first plugin specified will be used. If not set, POPF will be used by default. Check [this config](https://github.com/IntelligentRoboticsLabs/ros2_planning_system/blob/master/plansys2_bringup/params/plansys2_params.yaml) as an example on how to use it.

> `- plan_solver_plugins` \[vector\<string\>\]: PDDL 求解器插件列表。 目前，仅使用指定的第一个插件。 如果未设置，则默认情况下将使用 POPF。 请查看[此配置]以了解如何使用它。

## Client API

```cpp
boost::optional<plansys2_msgs::msg::Plan> getPlan(const std::string & domain, const std::string & problem)
```

## Services

- `planner/get_plan` \[plansys2_msgs::srv::GetPlan\]: Get a plan that will satisfy the provided domain and problem.

## Publishers / Subscriber

None

# 4. Executor

This component is responsible for executing a provided plan. It is, by far, the most complex component since the execution involves activating the action performers. This task is carried out with the following characteristics:

> 这个组件负责执行提供的计划。它是目前最复杂的组件，因为执行涉及激活动作 Executor。这项任务是以下特征进行的：

- It optimizes its execution, parallelizing the actions when possible.
- It checks if the requirements are met at runtime.
- It allows more than one action performer for each action, supporting multirobot execution.

![image](images/plansys2_arch2.png){.align-center width="600px"}

## Parameters

- `action_timeouts.actions` \[vector\<string\>\]: List of actions with enabled duration timeout capability. When the duration timeout capability is enabled for a given action, the action will halt after exceeding the action duration by more than a specified percentage. Duration timeouts are not enabled by default. To enable duration timeouts, the user must provide a custom action execution XML behavior tree template that includes the CheckTimeout BT node. Additionally, this parameter must specify the actions for which duration timeouts are enabled. Finally, the duration overrun percentage must be specified for each action.
- `action_timeouts.<action_name>.duration_overrun_percentage` \[double\]: When action duration timeouts are enabled (see explanation above), the duration overrun percentage specifies the amount of time an action is allowed to overrun its duration before halting. The overrun time is defined as a percentage of the action duration specified by the domain.
- `default_action_bt_xml_filename` \[string\]: Filepath to a user provided custom action execution XML behavior tree template. The user can use this template to specify a different XML behavior tree template than the one provided in the plansy2_executor package. Currently the only available BT node not used by the default behavior tree template is the CheckTimeout node.
- `enable_dotgraph_legend` \[bool\]: Enable legend with planning graph in DOT graph plan viewer.
- `print_graph` \[bool\]: Print planning graph to terminal.
- `enable_groot_monitoring` \[bool\]: Enable visualizing the plan\'s behavior tree inside [Groot](https://github.com/BehaviorTree/Groot).
- `publisher_port` \[unsigned int\]: ZeroMQ publisher port for [Groot](https://github.com/BehaviorTree/Groot).
- `server_port` \[unsigned int\]: ZeroMQ server port for [Groot](https://github.com/BehaviorTree/Groot).
- `max_msgs_per_second` \[unsigned int\]: Maximum number of ZeroMQ messages sent per second to [Groot](https://github.com/BehaviorTree/Groot).

> - `action_timeouts.actions` \[vector\<string\>\]: 具有可用持续时间超时功能的动作列表。 当给定动作启用持续时间超时功能时，动作将在超出动作持续时间超过指定百分比后停止。 默认情况下不启用持续时间超时。 要启用持续时间超时，用户必须提供包含 CheckTimeout BT 节点的自定义动作执行 XML 行为树模板。 此外，此参数必须指定启用持续时间超时的操作。 最后，必须为每个操作指定持续时间超越百分比。
> - `action_timeouts.<action_name>.duration_overrun_percentage` \[double\]: 当启用操作持续时间超时(请参阅上面的解释)时，持续时间超额百分比指定在停止之前允许操作超过其持续时间的时间量。超额时间由域指定的操作持续时间的百分比定义。
> - `default_action_bt_xml_filename` \[string\]：用户提供的自定义动作执行 XML 行为树模板的文件路径。用户可以使用此模板指定不同于 plansy2_executor 包中提供的 XML 行为树模板的模板。目前，默认行为树模板未使用的唯一可用 BT 节点是 CheckTimeout 节点。
> - `enable_dotgraph_legend` \[bool\]：在点图计划查看器中使用计划图的传奇。
> - `print_graph` \[bool\]：打印计划图到终端。
> - 启用 Groot 监控[bool]：在[Groot]中可视化计划的行为树。
> - `publisher_port` \[unsigned int\]: 用于[Groot]的 ZeroMQ 发布者端口。
> - `server_port` \[unsigned int\]: [Groot] 的 ZeroMQ 服务器端口。
> - `max_msgs_per_second` \[unsigned int\]: 每秒发送给[Groot]的 ZeroMQ 消息的最大数量。

## Client API

```cpp
bool start_plan_execution(const plansys2_msgs::msg::Plan & plan);
bool execute_and_check_plan();
void cancel_plan_execution();
std::vector<plansys2_msgs::msg::Tree> getOrderedSubGoals();
std::optional<plansys2_msgs::msg::Plan> getPlan();

ExecutePlan::Feedback getFeedBack() {return feedback_;}
std::optional<ExecutePlan::Result> getResult();
```

## Actions

- `execute_plan` \[plansys2_msgs::action::ExecutePlan\]: Execute the provided plan.

## Publishers / Subscriber

- `dot_graph` \[std_msgs::msg::String\] {Publisher: rclcpp::QoS(1)}: Publishes the planning DOT graph.
- `/action_execution_info` \[plansys2_msgs::msg::ActionExecutionInfo\] {Publisher: rclcpp::QoS(100)}: Publishes the action execution information. Note that the action execution information is also provided to the ExecutePlan action client via the feedback and result channels.

> 发布规划点图图形的`dot_graph` [std_msgs::msg::String] {发布者: rclcpp::QoS(1)}：
> 发布动作执行信息。请注意，动作执行信息也通过反馈和结果通道提供给 ExecutePlan 动作客户端。

# Behavior Tree builder

Once a plan is obtained, the Executor converts it to a Behavior Tree to execute it. Each action becomes the following subtree:

> 一旦获得计划，Executor 将其转换为行为树来执行。每个动作都成为以下子树：

![image](images/action_bt.png){.align-center width="400px"}

The first step is building a planning graph that encodes the action dependencies that define the execution order. This is made by pairing the effects of an action with a requirement of a posterior action. We take as reference the time of the calculated plan:

> 第一步是建立一个规划图，用来编码定义执行顺序的行动依赖关系。这是通过将行动的效果与后续行动的要求相匹配来完成的。我们以计算计划的时间为参考。

![image](images/action_deps.png){.align-center width="250px"}
![image](images/plan_graph.png){.align-center width="200px"}

Once created the graph, we identify the execution flows:

> 一旦创建图表，我们就可以识别执行流程：

![image](images/graph_flows.png){.align-center width="800px"}

From the red flow, for example, we get:

> 从红色流动，例如，我们得到：

![image](images/red_flow.png){.align-center width="400px"}

Each flow is executed in parallel. There is no problem if flows overlap because the BT that executes an action is implemented following a Singleton-like approach.

> **每个流都是并行执行的**。如果流重叠也没有问题，因为执行操作的 BT 是遵循单例模式实现的。

# Action delivery protocol

In the first implementations of PlanSys2, the delivery of actions was done using ROS2 actions. This approach has currently been discarded as it is not flexible enough. Instead, a bidding-based delivery protocol has been developed in the `ActionExecutor` and `ActionExecutorClient` classes that uses the `plansys2_msgs::msg::ActionExecution` message.

> 在 PlanSys2 的最初实现中，动作的传递是使用 ROS2 动作完成的。由于不够灵活，这种方法目前已被抛弃。取而代之的是，在`ActionExecutor`和`ActionExecutorClient`类中开发了一种基于竞标的传递协议，使用`plansys2_msgs::msg::ActionExecution`消息。

When the Executor must execute an action, it requests which action performer can execute it. Those who can reply to this request. The Executor confirms one of them (the first to answer), rejecting the rest. If none are found, repeat the request every second until you give up, aborting the execution of the plan. This protocol uses the topic `/action_hub`, where you can monitor the execution of the system.

> 当 Executor 必须执行一项动作时，它会请求哪个动作 Executor 可以执行它。那些可以回应这个请求的人。Executor 确认其中一个(第一个回答的)，拒绝其他人。如果没有找到，每秒重复请求，直到放弃，中止计划的执行。该协议使用主题`/action_hub`，您可以在其中监视系统的执行情况。

![image](images/protocol.png){.align-center width="500px"}

All the action performers inherit from `ActionExecutorClient`, that is a ROS2 Node with the next information:

> 所有行动 Executor 继承自`ActionExecutorClient`，这是一个具有以下信息的 ROS2 节点：

## Parameters

- `~/action` \[string\]: The action managed. This action performer discard any request non equal to this parameter.
- `~/specialized_arguments` \[vector\<string\>\]: If this parameter is not void, it only replies to action request that contains in any of the arguments any of these values.

> - 管理的动作。此动作 Executor 会丢弃任何与此参数不等的请求。
> - 如果此参数不为空，它只会回复包含这些值中任何一个参数的动作请求。

::: note
::: title
Note
:::

In a multirobot application, for example, we add to the actions a parameter with the robot that should do the action. In each robot we can execute the same action performer, and using `~/specialized_arguments` we can select which will be executed.

> 在多机器人应用中，例如，我们为动作添加了一个参数，用于指定应该执行该动作的机器人。我们可以在每个机器人上执行相同的动作 Executor，并使用`~/specialized_arguments`来选择要执行哪个。

:::

## Publishers / Subscriber

- `/actions_hub` \[plansys2_msgs::msg::ActionExecution\] {Publisher: rclcpp::QoS(100).reliable()}: Receive messages from the Action Hub.
- `/actions_hub` \[plansys2_msgs::msg::ActionExecution\] {Subscriber: rclcpp::QoS(100).reliable()}: Publish messages to the Action Hub.

> 从动作中心接收消息：`/actions_hub` [plansys2_msgs::msg::ActionExecution] {发布者：rclcpp::QoS(100)。可靠()}。
> 将消息发布到动作中心，使用 rclcpp::QoS(100)。可靠的订阅者。
