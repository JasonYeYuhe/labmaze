# LabMaze (DeepMind Lab 3D 迷宫生成器)

<p align="center">
  <a href="README.md">English</a> · <b>简体中文</b>
</p>

**LabMaze** 是 [DeepMind Lab](https://github.com/deepmind/lab) 所采用的迷宫生成器的独立开源版本。它是一个基于回溯法（backtracking）的迷宫生成器，支持配置简化路径以及生成房间。

LabMaze 的主要初衷是为 DeepMind Lab 的迷宫生成器提供 Python 绑定，以便生成的迷宫布局能够通过其他第三方库创建导航任务。该软件包还包含了艺术资源资产，用于以与 DeepMind Lab 相同的风格对生成的迷宫环境进行纹理渲染。

该迷宫生成算法及其代码最初由 DeepMind 的 Charlie Beattie 设计并编写。

## 论文引用 (Citation)

如果您的研究工作使用了 LabMaze，建议引用 [DeepMind Lab 论文](https://arxiv.org/abs/1612.03801) 并附上本 GitHub 仓库的链接：

```bibtex
@article{beattie2016deepmind,
  title={DeepMind Lab},
  author={Beattie, Charles and Leibo, Joel Z and Teplyashin, Denis and Ward, Tom and Marcus, Marcus and Sadik, Abbas and Antonoglou, Ioannis and King, Helen and Beattie, Simon and Legg, Shane and Petersen, Stig and Hester, Todd and Green, Jesse and Horgan, Daniel and Laborde, Arthur and Degrave, Jonas and Dupont, Eugene and Bignell, Nick and Ramalho, Tiago and O'Donoghue, Tom and Muldal, Alistair and Hassabis, Demis},
  journal={arXiv preprint arXiv:1612.03801},
  year={2016}
}
```

## 从 PyPI 安装 (Installation from PyPI)

LabMaze 包含首先需要编译为 Python 扩展的 C++ 代码。针对运行在 x64 Linux、macOS 和 Windows 上的 Python 3.6、3.7、3.8 和 3.9，我们提供了[预编译分发包（bdist）](https://packaging.python.org/glossary/#term-built-distribution)，可以直接通过以下命令安装：

```sh
pip install labmaze
```

我们同时向 PyPI 上传了[源码分发包（sdist）](https://packaging.python.org/glossary/#term-source-distribution-or-sdist)，这使得 LabMaze 也能够在其他平台上通过 `pip install labmaze` 进行编译安装。然而，您需要首先[安装 Bazel](https://docs.bazel.build/versions/master/install.html)，以及链接页面中所记录的相应平台构建工具链。

## 从 GitHub 仓库安装 (Installation from GitHub repository)

或者，您也可以使用 `pip` 直接从我们的 GitHub 仓库安装。您需要首先[安装 Bazel](https://docs.bazel.build/versions/master/install.html) 以及链接页面中记录的特定平台构建工具链，然后运行：

```sh
pip install git+https://github.com/deepmind/labmaze.git
```

## 使用示例：随机迷宫 (Example usage: random mazes)

生成一个随机迷宫：

```python
import labmaze
maze = labmaze.RandomMaze(height=11, width=13, random_seed=42)
print(maze.entity_layer)

*************
*   *   *****
*   *   *****
*           *
*** *** *** *
*           *
*   *****   *
*   *****   *
*   *****   *
*   *****   *
*************
```

默认情况下，迷宫墙壁由 `'*'` 符号表示。`RandomMaze` 对象可以重新随机生成：

```python
maze.regenerate()
print(maze.entity_layer)

*************
*******     *
*******     *
*     *     *
* *** ***** *
*   *       *
*   *** *** *
*       *   *
*   *   *   *
*   *       *
*************
```

我们还可以生成出生点（spawn locations）和目标物品位置（goal object locations），例如用于导航任务。当调用 `regenerate()` 方法时，这些位置将与迷宫布局本身一同被重新随机化生成：

```python
maze = labmaze.RandomMaze(height=11, width=13, random_seed=42,
                          spawns_per_room=1, objects_per_room=1)
print(maze.entity_layer)

*************
*P  *P  *****
*   * G *****
* G         *
*** *** *** *
* P      PG *
*   *****   *
*   *****   *
*   *****   *
*G  *****   *
*************
```

默认情况下，玩家出生位置由 `'P'` 符号表示，目标物品位置由 `'G'` 符号表示。在上面的示例中，迷宫中的每个“房间”都生成了一个玩家出生位置。使用上述生成布局的一种方式是：随机挑选四个出生点之一作为玩家起点，并在所有四个目标位置放置目标物品。

`RandomMaze` 对象还包含“变体层（variation layers）”，用于指定迷宫不同区域应如何进行纹理贴图渲染：

```python
.............
.BBB.DDD.....
.BBB.DDD.....
.BBB.DDD.....
.............
.CCC.....AAA.
.CCC.....AAA.
.CCC.....AAA.
.CCC.....AAA.
.CCC.....AAA.
.............
```

`'.'` 符号代表“默认”纹理风格，而每个英文字母符号代表不同的纹理风格。用户可以自行决定如何将建议的布局转换为不同的实际视觉纹理。

## 使用示例：固定布局迷宫 (Example usage: fixed-layout mazes)

LabMaze 还支持通过字符串直接指定的固定布局迷宫：

```python
MAZE_LAYOUT = """
*********
*********
*********
***   ***
***   ***
***   ***
*********
"""[1:]
maze = labmaze.FixedMazeWithRandomGoals(entity_layer=MAZE_LAYOUT)
print(maze.entity_layer)

*********
*********
*********
***   ***
***   ***
***   ***
*********
```

出生点与目标位置也可以在固定布局迷宫中直接指定或随机生成。例如，如果需要一个固定的出生点和一个随机生成的目标：

```python
MAZE_LAYOUT = """
*********
***   ***
***   ***
*** P ***
***   ***
***   ***
*********
"""[1:]
maze = labmaze.FixedMazeWithRandomGoals(entity_layer=MAZE_LAYOUT,
                                        num_spawns=1, num_objects=1)
print(maze.entity_layer)

*********
***   ***
***   ***
*** P ***
***   ***
***G  ***
*********

maze.regenerate()
print(maze.entity_layer)

*********
***   ***
***   ***
*** PG***
***   ***
***   ***
*********
```

更一般地，固定布局字符串中存在的任何出生点或目标符号都会被迷宫生成器原样保留。如果 `num_objects` 大于字符串中已存在的目标符号数量，则会自动随机生成额外的目标位置。`num_spawns` 同理。

## 免责声明 (Disclaimer)

本仓库不是 Google 的官方正式产品（This is not an official Google product）。

---

> **翻译说明（Translation Disclaimer）**
> 本文档由社区志愿者（[@JasonYeYuhe](https://github.com/JasonYeYuhe)）人工翻译并维护，旨在降低中文开发者的阅读门槛。最后同步日期：2026年10月23日。若译文与英文原版存在任何出入或滞后，请以 [英文原版 (README.md)](README.md) 为准。欢迎通过 PR 协助改进译文！
