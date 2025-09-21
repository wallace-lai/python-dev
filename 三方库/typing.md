# 1. 介绍

`typing`库是Python 3.5引入的标准库，用于支持类型提示。它让开发者能够为变量、函数参数和返回值等标注类型信息，提高代码的可读性和可维护性。类型提示本身不影响运行时行为，但可以通过静态类型检查工具（如mypy）发现潜在的类型错误。

# 2. 案例说明

```py
from typing import (
    TYPE_CHECKING,
    Any,
    Generic,
    Optional,
    TypeVar,
    Union,
)
```

说明：

## 2.1 TYPE_CHECKING- 类型检查专用常量

这是一个在​​运行时恒为False​​，仅在​​静态类型检查器​​（如 mypy或 IDE）工作时才被视为True的特殊常量

用途：常用于将**​​仅用于类型注解​**​的导入放在条件语句中，避免在运行时执行这些导入。这有助于​​解决循环导入问题​​，并减少不必要的运行时开销

示例：

```py
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    # 以下导入只在类型检查时进行，运行时不会执行
    from expensive_module import SomeClass
    from another_module import AnotherClass

def process_item(item: 'SomeClass') -> 'AnotherClass':  # 这里使用字符串形式的类型注解（前向引用）
    ...
```

## 2.2 Any - 动态类型或任意类型

当某个值的类型过于动态或复杂而无法精确指定，或者你想明确表示允许任何类型时，可以使用Any

用途：关闭类型检查​​。标注为Any的类型会与所有其他类型兼容。但过度使用会降低类型提示的好处

示例：

```py
from typing import Any

def flexible_function(data: Any) -> Any:
    # 这个函数接受任何类型的参数，并返回任何类型的值
    return data
```

## 2.3 Optional - 可选类型

用于表示一个值可以是某种类型，也可以是 None。这在函数可能返回 None或参数可选时非常常见。本质上`Optional[T]`等价于`Union[T, None]`。

用途：明确表示可能存在的None值，使类型意图更加清晰

示例：

```py
from typing import Optional

def find_user(username: str) -> Optional[str]:
    # 这个函数可能找到用户并返回其名称，也可能找不到而返回 None
    if user_exists(username):
        return username
    else:
        return None
```

## 2.4 Union - 联合类型

用于表示一个值可以是多种类型中的一种。

用途：当函数参数或返回值有不止一种可能的类型时

示例：

```py
from typing import Union

def square(number: Union[int, float]) -> Union[int, float]:
    # 这个函数接受整数或浮点数，并返回整数或浮点数
    return number ** 2

# Python 3.10+ 中可以写为：
# def square(number: int | float) -> int | float:
#     return number ** 2
```

## 2.5 TypeVar - 类型变量

用于创建​​类型变量​​，它是​​泛型编程​​的基础。

用途：当你希望一个函数或类中的多个值的类型是​​相关联的​​（例如，函数返回值的类型与参数类型相同），但又希望它是​​灵活可变的​​时，使用 TypeVar

示例：

```py
from typing import TypeVar, List

# 创建一个名为 T 的类型变量
T = TypeVar('T')

def get_first_element(items: List[T]) -> T:
    # 这个函数接受一个某种类型的列表，并返回单个该类型的元素
    return items[0]

# 类型检查器会推断出：
first_num: int = get_first_element([1, 2, 3])     # T 是 int
first_str: str = get_first_element(['a', 'b', 'c']) # T 是 str
```

你还可以为 TypeVar指定​​边界​​，限制它可以代表的类型范围：

```py
from typing import TypeVar

# 限制 NumberT 只能是 int 或 float
NumberT = TypeVar('NumberT', int, float)

def double(x: NumberT) -> NumberT:
    return x * 2
```

## 2.6 Generic - 泛型类基类

用于定义​​泛型类​​，泛型类是指其行为可以参数化，基于一个或多个类型参数的类

用途：与 TypeVar结合使用，创建可重用、类型安全的容器或数据结构，这些容器可以处理多种不同的类型，而无需为每种类型重写类。

示例：

```py
from typing import TypeVar, Generic

T = TypeVar('T')

class Box(Generic[T]):
    """一个可以存放任意类型值的盒子"""
    def __init__(self, content: T):
        self.content = content

    def get_content(self) -> T:
        return self.content

# 实例化泛型类
int_box: Box[int] = Box(123)        # 明确指定 Box 的类型参数是 int
str_box: Box[str] = Box("hello")    # 指定类型参数是 str

content_int: int = int_box.get_content()  # 类型检查器知道这是 int
content_str: str = str_box.get_content()  # 类型检查器知道这是 str
```

## 注意事项：

1. 类型提示不影响运行时行为​​：Python 解释器在运行时​​不会强制检查​​这些类型提示。它们主要用于​​静态类型检查器、IDE 和文档​​，以提高代码质量。

2. Python 仍是动态类型语言​​：添加类型提示并不会改变 Python 作为动态类型语言的本质，类型提示是​​可选的​​。

3. ​性能无影响​​：类型提示在运行时会被忽略，因此​​不会对性能产生任何影响​​

4. 使用静态类型检查工具​​：要充分利用类型提示的优势，建议使用mypy等工具对代码进行静态检查