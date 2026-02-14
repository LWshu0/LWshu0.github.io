---
title: Python 包管理
copyright: true
date: 2023-09-18 22:56:09
updated: 2025-01-26 00:03:00
tags:
    - Python
categories:
    - Python
#excerpt:
#index_img:
#banner_img:
---

## 项目包管理方式

### 常见问题

当自己写的项目层级复杂时, 由于不正确的包管理方式(或者其他外界因素)导致:
- 报错 `No module named xxx`
- `import xxx` 时找不到已经安装了的包
- `from xxx import xxx` 和 `import xxx` 使用混乱

该章节来介绍一种合适的包管理方式, 从而避免与此相关的报错.

### 相关解决方法

#### 禁用 Pylance

vscode 中的一个 python 插件 **Pylance** 是微软官方编写的插件, 但是使用后 vscode 可能会找不到已经在 python 环境中安装的软件包. 我是在使用一个由 `pip install -e .` 方式安装的第三方包时出现我不期望的行为:
> 在 import 软件包时 vscode 给出警告, 提示找不到包, 那么也就没有代码补全等功能. 但是运行 python 文件并不会报错.

这很明显是 vscode 的问题, 排查后是 **Pylance** 这个插件的导致的, 所以禁用它即可.

#### 使用 `__init__.py`

在每一个属于包的文件夹中添加一个 `__init__.py` (可以为空文件), 这个文件是 python 识别包用的, 包含这个文件的文件夹将被 python 视为一个包. python 的相对导入 (如 `from . import my_module`) 依赖于 python 的包结构. 

例如下列的目录结构:
```
my_project/
│
├── main.py
├── utils/
│   ├── __init__.py
│   ├── helper.py
│   └── math_utils.py
├── models/
│   ├── __init__.py
│   ├── sub_models/
│   │   ├── __init__.py
│   │   ├── sub_model_a.py
│   │   └── sub_model_b.py
│   ├── model_a.py
│   └── model_b.py
└── tests/
    ├── __init__.py
    └── test_utils.py
```

其中, `utils`, `models`, `sub_models`, `tests` 都是包.

#### 绝对导入与相对导入

> **顶层模块**: 当前运行的 python 文件所在的文件夹, 该文件夹即使添加了 `__init__.py` 也不会被 python 当作一个包, 例如上例的 `my_project`.
> 
> **主脚本**: 在项目根目录下的 python 脚本.

- **绝对导入** 是从顶层模块中开始寻找包. 例如在 main.py 中导入 helper.py 中的一个 `help_func` 函数时可以 `from utils.helper import help_func`; 又比如在 main.py 中导入 sub_model_a.py 中的一个 `SubModelA` 对象时可以 `from models.sub_models.sub_model_a import SubModelA`. 

    绝对导入一般适用与主脚本, 因为主脚本只能从项目根目录开始找包, 无法使用相对寻址去找包. 

- **相对导入** 是从当前 python 文件开始按照相对路径寻找包. 例如在 helper.py 中导入 math_utils.py 中的一个 `math_func` 函数时可以 `from .math_utils import math_func`; 又如 sub_model_a.py 中导入 math_utils.py 中的一个 `math_func` 函数时可以 `from ...utils.math_utils import math_func`. 在相对导入时, `.` 表示当前包, `..` 表示父包, `...` 表示父包的父包, 以此类推.

    相对导入一般适用于包内的脚本, 当一个软件包可能直接被复制到另一个软件包中时, 此时项目的根目录一定发生了变化, 这会导致原来使用绝对导入的包找不到; 而使用相对导入就不会出现这个问题.

**总结: 主脚本中只能使用绝对导入; 软件包中尽量使用相对导入.**

#### 不要直接运行包内脚本

如果直接运行一个包内的脚本 (例如 python utils/helper.py), Python 会将这个脚本所在文件夹视为顶级模块而不是包的一部分, 因此也就无法识别父包. 如果 helper.py 中采用了相对导入, 就会导致报错 `No module named xxx`.

如果一定要运行, 添加 `-m` 参数, 比如在 my_project 目录下执行 `python -m utils.helper` 即可运行 helper.py 而不会报错.

#### 注意导入包顺序

- 避免循环导入.
- 在导入一个包之前会首先执行包的 `__init__.py` 中的内容, 所以可以在 `__init__.py` 中初始化一些可供包中所有文件使用的全局变量.
- 在导入嵌套的包时, 会从顶层到底层以此执行其 `__init__.py`.
    ```py
    # models/__init__.py
    print("models init")

    # models/sub_models/__init__.py
    print("sub_models init")

    # main.py
    import models.sub_models

    # 执行 main.py 输出
    models init
    sub_models init
    ```

## 动态导入

依托 python 内置库 **importlib** 可以实现 python 包的动态导入. 配合 **argparse** 可以通过设置不同的命令行参数来使一份脚本更具通用性.

### 导入包

```py
import path.to.module as module

# 等价于

try:
    module_path = "path.to.module"
    module = importlib.import_module(module_path)
except ImportError:
    print(f"无法导入模块: {module_path}\n")

```

### 获取包中对象

假设已经导入包 `import path.to.module as module`

```py
from moudule import my_obj as obj

# 等价于

try:
    obj_name = "my_obj"
    obj = getattr(module, obj_name)
except AttributeError:
    print(f"模块 \n{module} 中没有找到对象: {obj_name}")

# 等价于

try:
    obj = module.my_obj
except AttributeError:
    print(f"模块 \n{module} 中没有找到对象: my_obj")
```