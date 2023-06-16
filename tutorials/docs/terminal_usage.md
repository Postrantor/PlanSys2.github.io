---
tip: translate by openai@2023-06-16 08:52:06
...
---
title: Terminal Usage
---------------------

- [Overview](#overview)
- [Tutorial Steps](#tutorial-steps)
  - [0- Requisites](#0--requisites)
  - [1- Launch PlanSys2](#1--launch-plansys2)
  - [2- Execute PlanSys2 terminal](#2--execute-plansys2-terminal)
  - [2- Interactive session](#2--interactive-session)

# Overview

The terminal is an application that allows the interaction with PlanSys2 to test or monitor its operation. It is not essential to use PlanSys2 since your application is expected to manage the plans\' knowledge and execution automatically. Still, it is useful and convenient to get to know the terminal in these first tutorials.

> 终端是一个应用程序，可以与 PlanSys2 进行交互，以测试或监控其操作。虽然您的应用程序期望自动管理计划的知识和执行，但不是必须使用 PlanSys2。尽管如此，在这些初级教程中了解终端是有用且方便的。

This tutorial shows how to use the terminal and the main commands and is also a good starting point for working with PlanSys2.

> 这个教程展示了如何使用终端和主要命令，也是开始使用 PlanSys2 的一个很好的起点。

# Tutorial Steps

## 0- Requisites

Follow the same process as in `getting_started`{.interpreted-text role="ref"} for installing PlanSys2.

> 请按照 `getting_started`{.interpreted-text role="ref"}中的步骤安装 PlanSys2。

We will use a test PDDL domain. If you do not know what PDDL is, it is recommended to take a look at one of these links first:

> 我们将使用一个测试 PDDL 域。如果您不知道什么是 PDDL，建议您先查看其中一个链接：

- [PDDL 2.1](https://arxiv.org/pdf/1106.4561.pdf)
- [An Introduction to PDDL](http://www.cs.toronto.edu/~sheila/2542/w09/A1/introtopddl2.pdf)
- [PDDL by Example](http://www.cs.toronto.edu/~sheila/384/w11/Assignments/A3/veloso-PDDL_by_Example.pdf)

Download a simple PDDL domain for this tutorial. Later, you could reproduce the steps of this tutorial with your own domain:

> 下载一个简单的 PDDL 域以用于本教程。稍后，您可以使用自己的域重复本教程的步骤：

```bash
wget -P /tmp https://raw.githubusercontent.com/IntelligentRoboticsLabs/ros2_planning_system_examples/master/plansys2_simple_example/pddl/simple_example.pddl
```

## 1- Launch PlanSys2

PlanSys2 can be launched with two different launchers that implement two different execution forms: distributed or monolithic. In distributed form, each component of PlanSys2 runs in a different process, and to launch it, each component\'s launchers are called in cascade. In monolithic form, all components are released in the same process.

> PlanSys2 可以通过两种不同的启动器以两种不同的执行形式启动：分布式或单体式。在分布式形式下，PlanSys2 的每个组件都在不同的进程中运行，要启动它，需要按顺序调用每个组件的启动器。在单体式形式下，所有组件都在同一个进程中运行。

In the first steps with PlanSys2, it is irrelevant how we execute it to choose any of them. Both launchers have a parameter where the PDDL domain file to use is specified.

> 在使用 PlanSys2 的第一步，我们如何执行它并不重要，可以任选其中之一。两种启动器都有一个参数，用于指定要使用的 PDDL 域文件。

```bash
ros2 launch plansys2_bringup plansys2_bringup_launch_distributed.py model_file:=/tmp/simple_example.pddl
```

If all went well, all components of PlanSys2 will have been launched and awaits requests.

> 如果一切顺利，PlanSys2 的所有组件都将启动并等待请求。

## 2- Execute PlanSys2 terminal

The PlanSys2 terminal is executed by entering the following command in another terminal:

> 在另一个终端中输入以下命令来执行 PlanSys2 终端：

```bash
ros2 run plansys2_terminal plansys2_terminal
```

At that moment, an interactive shell will open in which we can enter commands. We can use the up and down arrows to navigate between the commands already entered or use `Ctrl-R` as in the shell to search for commands. It also has autocompletion with the `Tab` key. You can quit anytime typing \"quit\" or pressing `Ctrl-D`.

> 在那一刻，一个交互式的壳将打开，我们可以在其中输入命令。我们可以使用上下箭头在已输入的命令之间导航，或者像在壳中一样使用 `Ctrl-R` 来搜索命令。它还有使用 `Tab` 键的自动完成功能。您可以随时输入"quit"或按 `Ctrl-D` 退出。

It should be noted that the state of the system is at PlanSys2 components, not in the terminal. You can close and reopen (or even use several terminals), and the system\'s state persists. If you want to reset the system, press `Ctrl-C` in the terminal where you launched PlanSys2, and relaunch it.

> 应当指出，系统的状态在 PlanSys2 组件中，而不是在终端中。您可以关闭和重新打开（甚至可以使用几个终端），系统的状态仍然保持不变。如果您想要重置系统，请在您启动 PlanSys2 的终端中按下 Ctrl-C，然后重新启动它。

```lisp
ROS2 Planning System console. Type "quit" to finish
>
```

## 2- Interactive session

1. The first thing is to check the domain that is being used. In the PlanSys2 terminal window type:

> 第一件事是检查正在使用的域。在 PlanSys2 终端窗口中输入：

```lisp
get domain

( define ( domain simple )
( :requirements :strips :adl :typing :durative-actions :fluents )
( :types
  robot - object
  room - object
)
( :predicates
  ( robot_at ?robot0 - robot ?room1 - room )
  ( connected ?room0 - room ?room1 - room )
  ( battery_full ?robot0 - robot )

  ...
```

2. To see what types of instances the model contains, type:

> 查看模型包含哪些类型的实例，请输入：

```lisp
get model types
Types: 2
      robot
      room
```

3. Use other variations of `get model` to get more information of the domain:

> 使用其他变体的"get model"来获取更多关于域的信息。

```lisp
get model actions
Actions: 0
  move (durative action)
  askcharge (durative action)
  charge (durative action)

get model predicates
Predicates: 5
  robot_at
  connected
  battery_full
  battery_low
  charging_point_at
```

4. It is also possible to get the details of a predicate or an action:

> 也可以获取谓词或动作的细节：

```lisp
get model predicate robot_at
Parameters: 2
 robot - ?robot0
 room - ?room1

get model action move
Type: durative-action
Parameters: 3
 ?0 - robot
 ?1 - room
 ?2 - room
AtStart requirements: (and (connected ?1 ?2)(robot_at ?0 ?1))
OverAll requirements: (and (battery_full ?0))
AtEnd requirements:
AtStart effect: (and (not (robot_at ?0 ?1)))
AtEnd effect: (and (robot_at ?0 ?2))
```

5\. So far, we have seen how to inspect the model, which remains unchanged during the execution of PlanSys2. We could say that it is the static part of the planning ingredients. The other ingredient is the problem, which contains the instances, grounded (not generic as in the domain, but already with instances) predicates, and goals. We will check that it is empty for now.

> 到目前为止，我们已经看到了如何检查模型，在 PlanSys2 的执行期间该模型保持不变。我们可以说，它是规划成分的静态部分。另一个成分是问题，其中包含实例、基础（不是域中的通用实例，而是已经有实例的）谓词和目标。我们将检查它现在是空的。

```lisp
get problem instances
Instances: 0

get problem predicates
Predicates: 0

get problem goal
Goal:
```

6\. First, let\'s add instances. If you analyze the domain, we want a robot to be able to move between rooms. For this, the robot must have a battery, and the rooms must be connected. Therefore, we need rooms and a robot:

> 首先，让我们添加实例。如果你分析域，我们希望机器人能够在房间之间移动。 为此，机器人必须有电池，房间必须相连。 因此，我们需要房间和机器人：

```lisp
set instance leia robot
set instance entrance room
set instance kitchen room
set instance bedroom room
set instance dinning room
set instance bathroom room
set instance chargingroom room
```

If no errros, these instances can be checked by typing:

> 如果没有错误，可以通过输入来检查这些实例：

```lisp
get problem instances
Instances: 7
 leia    robot
 entrance    room
 kitchen room
 bedroom room
 dinning room
 bathroom    room
 chargingroom    room
```

7. To add predicates, we type:

> 加入谓语，我们键入：

```lisp
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
```

Let\'s check it:

```lisp
get problem predicates
Predicates: 13
(connected entrance dinning)
(connected dinning entrance)
(connected dinning kitchen)
(connected kitchen dinning)
(connected dinning bedroom)
(connected bedroom dinning)
(connected bathroom bedroom)
(connected bedroom bathroom)
(connected chargingroom kitchen)
(connected kitchen chargingroom)
(charging_point_at chargingroom)
(battery_low leia)
(robot_at leia entrance)
```

8\. The predicates and instances previously added to the problem are the knowledge used to generate the plan. Also, we need to have an objective of our planning, which is a logic expression to end up being true. It is usually a predicate that we want to add to the knowledge:

> 8. 在问题中添加的谓词和实例是用来生成计划的知识。我们还需要有一个规划的目标，即一个逻辑表达式最终为真。通常是我们想要添加到知识中的谓词。

```lisp
set goal (and(robot_at leia bathroom))
```

9. At this time we can ask that the plan be calculated to obtain this goal:

> 此时我们可以要求计算计划以达到这一目标：

```lisp
get plan
plan:
0    (askcharge leia entrance chargingroom)  5
0.001    (charge leia chargingroom)  5
5.002    (move leia chargingroom kitchen)    5
10.003   (move leia kitchen dinning) 5
15.004   (move leia dinning bedroom) 5
20.005   (move leia bedroom bathroom)    5
```

To create the plan, the first thing to do is generate two files: `` `/tmp/domain.pddl``[ and ]{.title-ref}`/tmp/problem.pddl`\`. You can check that they are there from the last planning. In fact, we can run the planner directly by typing in a shell in another terminal:

> 要创建计划，首先要做的是生成两个文件：``/tmp/domain.pddl`` 和 ``/tmp/problem.pddl``。您可以检查它们是否已经从最后一次计划中出现。实际上，我们可以在另一个终端中输入一个 shell 来直接运行计划程序：

```bash
ros2 run popf popf /tmp/domain.pddl /tmp/problem.pddl
```

10. We can also delete instances, predicates or the goal:

> 我们也可以删除实例、谓词或目标。

```lisp
remove instance leia
remove predicate (connected entrance dinning)
remove goal
```

11\. What we will not be able to do is execute the plan (we would do it with the `run` command) because there is no node running right now that implements the domain actions. We will see that in the next tutorial.

> 11. 我们无法执行计划（我们将使用 `run` 命令执行），因为目前没有运行的节点来实现域动作。我们将在下一个教程中看到。
