# 1. 介绍

`typing`库是Python 3.5引入的标准库，用于支持类型提示。它让开发者能够为变量、函数参数和返回值等标注类型信息，提高代码的可读性和可维护性。类型提示本身不影响运行时行为，但可以通过静态类型检查工具（如mypy）发现潜在的类型错误。

# 2. typing模块的内容

## 基本类型注释
你可以直接使用 Python 的内置类型（如 int, str, float, bool）为变量、函数参数和返回值添加类型注解。

```py
def greeting(name: str) -> str:
    return 'Hello ' + name
```

## 常用类型工具

|类型工具|说明|示例|
|--|--|--|
|Union|表示一个值可以是几种类型中的一种|Union[int, str]表示可以是 int或 str|
|Optional|表示一个值可以是某种类型或 None，等价于 Union[Type, None]|Optional[str]表示可以是 str或 None|
|Any|表示任意类型，相当于禁用类型检查|data: Any|
|Callable|表示可调用对象（如函数），标注参数和返回值的类型|Callable[[int, int], int]表示接受两个 int参数并返回 int的函数|
|Literal|表示必须是特定的字面值|Literal["on", "off"]表示只允许字符串 "on" 或 "off"|
|Final|表示变量或属性不应被重新赋值（常量）|MAX_SIZE: Final[int] = 100|
|ClassVar|标注类变量|DEFAULT_PORT: ClassVar[int] = 8080|
|Annotated|为类型添加元数据（如约束条件、文档等），供其他工具或框架使用|Annotated[str, "Name of the user"]|

## 容器与泛型
使用 typing中的泛型类型可以明确容器中元素的类型，使类型提示更精确。

|容器类型|说明|示例|
|--|--|--|
|List[ElementType]|列表，元素类型为 ElementType|List[int]表示一个整数列表|
|Dict[KeyType, ValueType]|字典，键类型为 KeyType，值类型为 ValueType|Dict[str, int]表示键为字符串，值为整数的字典|
|Tuple[Type1, Type2, ...]|元组，固定位置和类型|Tuple[int, str]表示一个整数和一个字符串的元组|
|Set[ElementType]|集合，元素类型为 ElementType|Set[str]表示一个字符串集合|

## 高级类型特性
typing模块还支持一些更高级的类型特性。

（1）类型别名 (Type Aliases)​​：为复杂的类型声明创建一个简短的名称，提高可读性

```py
from typing import List, Dict, Union
# 类型别名
Vector = List[float]
UserProfile = Dict[str, Union[str, int]]  # 用户信息是字典，值为字符串或整数
```

（2）​​NewType​​：创建 distinct 类型（常用于避免逻辑错误，如把用户ID和普通整数混用）

```py
from typing import NewType
UserId = NewType('UserId', int)
some_id = UserId(524313)
```

（3）泛型编程 (Generics)​​：使用 TypeVar和 Generic创建可重用的泛型类或函数

```py
from typing import TypeVar, Generic, List

T = TypeVar('T')  # 声明类型变量
class Stack(Generic[T]):  # 泛型类
    def __init__(self) -> None:
        self.items: List[T] = []
    def push(self, item: T) -> None:
        self.items.append(item)
    def pop(self) -> T:
        return self.items.pop()
# 使用
int_stack: Stack[int] = Stack()  # 指定栈中元素为int
int_stack.push(1)
```

（4）Protocol (结构化子类型)​​：定义接口协议，支持"鸭子类型"，只要对象有所需的方法或属性，就视为符合类型。

```py
from typing import Protocol

class Speaker(Protocol):
    def speak(self) -> str: ...

class Dog:
    def speak(self) -> str:
        return "Woof!"

def make_sound(obj: Speaker) -> str:
    return obj.speak()

make_sound(Dog())  # 正确，因为 Dog 有 speak 方法
```

（5）​​TypedDict​​：为字典提供固定的键和对应的值类型。

```py
from typing import TypedDict

class User(TypedDict):
    name: str
    age: int

user: User = {"name": "Alice", "age": 30}
```

## 使用场景与工具

- **​静态类型检查**​​：类型提示主要供 ​​mypy​​、pyright、pyre等静态类型检查工具使用，在运行前发现类型错误。

- **​​IDE 支持**​​：类型提示能极大提升 IDE（如 PyCharm、VS Code）的代码补全、智能提示和重构能力。

- **​​运行时类型检查**​​：虽然 typing主要用于静态检查，但可结合 pydantic、typeguard等库在运行时进行类型验证。

- **​​框架集成**​​：许多现代框架（如 FastAPI）充分利用类型提示来自动生成 API 文档、处理数据验证和序列化。

# 3. 案例说明1

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

## 3.1 TYPE_CHECKING- 类型检查专用常量

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

## 3.2 Any - 动态类型或任意类型

当某个值的类型过于动态或复杂而无法精确指定，或者你想明确表示允许任何类型时，可以使用Any

用途：关闭类型检查​​。标注为Any的类型会与所有其他类型兼容。但过度使用会降低类型提示的好处

示例：

```py
from typing import Any

def flexible_function(data: Any) -> Any:
    # 这个函数接受任何类型的参数，并返回任何类型的值
    return data
```

## 3.3 Optional - 可选类型

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

## 3.4 Union - 联合类型

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

## 3.5 TypeVar - 类型变量

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

## 3.6 Generic - 泛型类基类

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