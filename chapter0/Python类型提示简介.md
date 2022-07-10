# Python类型提示简介

python在 3.6 版本加入了对 [类型提示](https://peps.python.org/pep-0526/) 的支持，并在之后的版本中不断对其进行了优化。

类型提示 type hints 是一种新的语法，用来声明一个变量的类型。通过声明变量的类型，编辑器和一些工具能够为用户提供更好的支持。

本章内容知识对于Python类型提示的**快速入门/复习**，仅涵盖与FastAPI一起使用的部分功能。



FastAPI代码库实现了100% 的类型提示，整个FastAPI都基于这些类型提示构建，他带了许多优点和好处。

即使你不需要使用FastAPI，阅读本章内容也会让你从中收益。如果你已经精通Python，则可以忽略本章内容。

---

## 类型提示是什么？

我们都知道Python是动态类型语言，不需要对变量的类型进行声明，一个变量也可以被赋值为不同的类型。与之相对的是C/C++，java等静态语言，他们要求对变量和函数返回值必须进行严格的类型声明。

1. 方法形参和返回值类型提示（type hints）：
    ```python
    def add(x: int, y: int) -> int:
        result = x + y
        print('%s + %s = %s' % (x, y, result))
        return result


    add(1, 2)
    add(2.2, 3.3)  # 编辑器提示类型错误
    add('3.4', '5.6')  # 编辑器提示类型错误
    ```


    形参类型在变量后使用 `:`表示，同时后面可以使用`=`指定参数默认值，返回值类型使用 `->`指定。

    在方法的形参中，我们可以指定参数的类型为`int`，返回值也为 `int`。当我们调用方法并传入其他类型的实参时，编辑器会进行静态检查并进行提示。

    执行以上代码：

    ```python
    1 + 2 = 3
    2.2 + 3.3 = 5.5
    3.4 + 5.6 = 3.45.6
    ```


    可见，对于`int`类型的入参，方法正常执行。而对于`float`和`str`类型的入参，编辑器会在静态检查时提示类型错误，但不会阻止错误类型的参数使用和运算。



2. 变量注解（annotations）：

    ```python
    name: str = "Jack"
    age: int = 20

    name = 30  # 编辑器提示类型错误
    age = '21'  # 编辑器提示类型错误
    print(name)
    print(age)
    ```


    在定义变量时也可以指定变量的类型。当给变量赋值其他类型数据时，编辑器会提示类型错误。但是，并不会阻止你这样做。代码仍可以正常运行。

    ```Python
    30
    21
    ```


    运行后可见，`name`虽然声明为`str`，但让能被赋值为`int`，`age`变量亦然。



### 既然类型声明不能阻止变量使用其他类型赋值，那么其意义何在呢？

类型提示带来的好处有三个：

1. 方便开发者阅读代码，提高代码易读性。

2. 方便编辑器/IDE 对代码进行提示、自动补全和静态检查。

3. 可通过 mypy （`pip install mypy`）进行类型检查。如对上面的脚本，使用mypy命令行工具进行检查：

    ```python

    D:\code\fast_api_learning>mypy type_hints.py
    type_hints.py:8: error: Argument 1 to "add" has incompatible type "float"; expected "int"
    type_hints.py:8: error: Argument 2 to "add" has incompatible type "float"; expected "int"
    type_hints.py:9: error: Argument 1 to "add" has incompatible type "str"; expected "int"
    type_hints.py:9: error: Argument 2 to "add" has incompatible type "str"; expected "int"
    type_hints.py:15: error: Incompatible types in assignment (expression has type "int", variable has type "str")
    type_hints.py:16: error: Incompatible types in assignment (expression has type "str", variable has type "int")
    Found 6 errors in 1 file (checked 1 source file)
    ```


### 支持的类型有哪些？

Python自带的`int`、`float`、`str`、`bool`、`bytes`、`list`、`dict`、`tuple`、`set`等基本数据类型和容器。

其他复杂的数据类型可以通过Python提供的`typing`库来实现，如`List`、`Dict`、`Set`、`Tuple`、`Any`、`Union`等，同时还可以使用 `[]` 指定容器中的数据类型，如`List[int]`。

typing库文档：[https://docs.python.org/3/library/typing.html](https://docs.python.org/3/library/typing.html)

有关typing库中的常用类型，会在下面结合FastAPI的使用给出示例。更多详细信息请阅读typing的文档进行学习。

---

## 在FastAPI中使用类型提示：

和上面提到了的使用场景相同，FastAPI中最常见的类型提示是对方法参数进行提示。另外就是对于数据类/schema 中的字段进行注解。

### 类型声明示例：

#### 1. 简单类型：

`int`、`float`、`str`、`bool`等

```Python
def get_items(item_a: str, item_b: int, item_c: float, item_d: bool, item_e: bytes):
    return item_a, item_b, item_c, item_d, item_d, item_e

```


#### 2. 带类型参数的泛型（generic types）：

有些容器数据结构可以包含其他的值，比如 `dict`、`list`、`set` 和 `tuple`。它们内部的值也会拥有自己的类型。

这些具有内部类型的类型称为“泛型”类型。我们也可以声明它们，甚至可以声明它们的内部类型。这时就可以使用 Python 的 `typing` 标准库来声明这些类型以及子类型。



使用 typing 的语法与所有Python版本兼容，从 Python 3.6到最新版本，包括 Python 3.9、 Python 3.10等等。随着 Python 的发展，新版本对这些类型注释提供了更好的支持，在许多情况下，您甚至不需要导入和使用typing模块来声明类型注释。

如果你能为你的项目选择使用较新版本的Python，那么你就能够享受这种额外的简单性。

操作示例：

1. 列表：
   
   定义一个元素为 `str` 的`list`变量。列表中的每个变量都是`str`类型。

   python3.9及之后版本：

   ```Python
   def process_items(items: list[str]):
       for item in items:
           print(item)
   ```


   不需要从typing中导入类型，直接使用`list`类型。元素的类型要放入`[]` 中。

   python3.6-3.8

   ```Python
   from typing import List


   def process_items(items: List[str]):
       for item in items:
           print(item)

   ```


   注意：导入的类型是`List` 不是`list`。元素的类型要放入`[]` 中。

   方括号中的内部类型被称为**类型参数**（type parameters）。



2. 元组和集合：
   
   元组合集合的使用与列表相同。

    python3.9及之后版本：

    ```Python
    def process_items(items_t: tuple[int, int, str], items_s: set[bytes]):
        return items_t, items_s

    ```

    python3.6-3.8：

    ```Python
    from typing import Set, Tuple


    def process_items(items_t: Tuple[int, int, str], items_s: Set[bytes]):
        return items_t, items_s

    ```


    以上代码表示：

    - 变量 `items_t`是一个由3个元素组成的元组，包括两个`int`和一个`str`。

    - 变量`items_s`是一个集合，集合中的每个元素都是`bytes`类型。



3. 字典：
   
   要定义一个字典需要两个类型参数，使用`逗号`分割。第一个类型参数指定 `keys`的类型，第二个指定`values`的类型。

    python3.9及之后版本：

    ```Python
    def process_items(prices: dict[str, float]):
        for item_name, item_price in prices.items():
            print(item_name)
            print(item_price)
    ```


    python3.6-3.8：

    ```Python
    from typing import Dict


    def process_items(prices: Dict[str, float]):
        for item_name, item_price in prices.items():
            print(item_name)
            print(item_price)

    ```


    这表示，`prices` 变量是一个字典：

    - 字典的`keys` 是`str`类型（每个item的`key`都是`str`）。

    - 字典的`values`是`float`类型（每个item的`value`都是`float`）。



 4. 联合体Union：
    
    我们可以通过`Union`来声明变量的类型是几种类型中的任何一种，例如`int`或`str`。

    在python3.6之后（包括python3.10），我们可以使用 `typing` 库中的 `Union` 类型并通过方括号指定可选的类型参数实现它。

    在Python 3.10中，提供了另一种可选的语法：通过`竖线 '|'` 来分割可能的类型。

    python3.10及之后版本：

    ```Python
    def process_item(item: int | str):
        print(item)
    ```


    python3.6及之后版本：

    ```Python
    from typing import Union


    def process_item(item: Union[int, str]):
        print(item)
    ```


    两种方法都表示参数`item`可以是一个`int`类型的变量，也可以是一个`str`类型的变量。



5. Optional：可能为 None
   
   我们可以声明一个变量可以有一个类型，如`str`，但也可以是`None`。

    在python3.6之后（包括python3.10），我们可以使用 `typing` 库中的 `Optional`类型来声明这种情况。

    ```Python
    from typing import Optional


    def say_hi(name: Optional[str] = None):
        if name is not None:
            print(f"Hey {name}!")
        else:
            print("Hello World")
    ```


    使用`Optional[str]`而不仅仅是`str`可以让编辑器帮助您检测错误，您可以假设一个值始终是`str`，而实际上它也可以是`None`。

    `Optional[Something]`实际上是 `Union[Something，None]`的一个缩写，它们是等价的。

    这意味着，在python3.10中你可以使用 `Something | None`：

    python3.10及之后版本：

    ```Python
    def say_hi(name: str | None = None):
        if name is not None:
            print(f"Hey {name}!")
        else:
            print("Hello World")
    ```


    python3.6及之后版本：

    ```Python
    from typing import Optional


    def say_hi(name: Optional[str] = None):
        if name is not None:
            print(f"Hey {name}!")
        else:
            print("Hello World")
            
    # 或者使用Union写法
    from typing import Union


    def say_hi(name: Union[str, None] = None):
        if name is not None:
            print(f"Hey {name}!")
        else:
            print("Hello World")

    ```


6. `Union` 或 `Optional`的使用建议：
    
    如果你使用的是3.10以下的 Python 版本，这里有一个来自我（指官方文档）非常主观的提示:

    - 🚨避免使用`Optional[SomeType]`.

    - ✨建议使用 `Union[SomeType, None]`.

    二者是等价的，而且在底层是相同的。但是我建议使用`Union`而不是`Optional`。因为`Optional`这个词似乎暗示这个值是可选的，实际上他的意思是**“它可以是None”**，即使他不是可选的它依然是必须的参数。

    而 `Union[str, SomeType]` 的含义更加明确。

    它只是关于单词和名字。但是这些话会影响你和你的队友对代码的思考。

    例如，在下面的方法中：

    ```Python
    from typing import Optional


    def say_hi(name: Optional[str]):
        print(f"Hey {name}!")
    ```


    参数 `name` 定义为 `Optional[str]` ，但参数本身不是可选的（必须传递的未知参数）。你不能在没有传递参数的情况下调用它：`say_hi()  # Oh, no, this throws an error! 😱`

    `name` 参数始终是必需的（不是可选的）因为他没有默认值。当然，`name`可以接收一个`None`值：`say_hi(name=None) # This works, None is valid` 🎉



    好消息是，一旦你使用了python3.10，你讲不用再担心，因为你可以使用更简单的 `| 语法`来定义联合体类型：

    ```Python
    def say_hi(name: str | None):
        print(f"Hey {name}!")
    ```


    这样你就不用在担心像`Optional` 和 `Union`这样的名字了。



7. 泛型类型：
   
   这些在方括号中接受类型参数的类型称为泛型类型或泛型，例如:

    |python3.6及之后版本|python3.9及之后版本 (可以像泛型一样使用内置类型)|python3.10及之后版本(可以像泛型一样使用内置类型)|
    |-|-|-|
    |List|list|list|
    |Tuple|tuple|tuple|
    |Set|set|set|
    |Dict|dict|dict|
    |Union|Union|Union|
    |Optional|Optional|Optional|
    |...|...|...|
    |||可以使用 竖线 &#124; 语法来表示Union或者Optional|




8. Class 作为类型：
   
   我们还可以将类声明为变量的类型。

    如，我们有一个`Person`类，他有一个`name`字段：

    ```Python
    class Person:
        def __init__(self, name: str):
            self.name = name
    ```


    之后，而我们可以声明一个变量是`Person`类型的：

    ```Python
    def get_person_name(one_person: Person):
        return one_person.name
    ```






#### 3. Pydantic models：

[Pydantic ](https://pydantic-docs.helpmanual.io/)是一个用于执行数据验证的 Python 库。

我们可以将数据的形状（shape）声明为类中的属性。同时每个属性都具有一个类型。

然后创建一个包含一些值的类实例，它将验证这些值，将它们转换为适当的类型(如果是这种情况) ，并为您提供一个包含所有数据的对象。

同时，你可以得到所有编辑器对结果对象的支持。



下面是一个Pydantic官方文档中的示例：

python3.10+：

```Python
from datetime import datetime
from pydantic import BaseModel


class User(BaseModel):
    id: int
    name = "John Doe"
    signup_ts: datetime | None = None
    friends: list[int] = []


external_data = {
    "id": "123",
    "signup_ts": "2017-06-01 12:22",
    "friends": [1, "2", b"3"],
}
user = User(**external_data)
print(user)
# > User id=123 name='John Doe' signup_ts=datetime.datetime(2017, 6, 1, 12, 22) friends=[1, 2, 3]
print(user.id)
# > 123
```


python3.9+：

```Python
from datetime import datetime
from typing import Union
from pydantic import BaseModel


class User(BaseModel):
    id: int
    name = "John Doe"
    signup_ts: Union[datetime, None] = None
    friends: list[int] = []


external_data = {
    "id": "123",
    "signup_ts": "2017-06-01 12:22",
    "friends": [1, "2", b"3"],
}
user = User(**external_data)
print(user)
# > User id=123 name='John Doe' signup_ts=datetime.datetime(2017, 6, 1, 12, 22) friends=[1, 2, 3]
print(user.id)
# > 123
```


python3.6+：

```Python
from datetime import datetime
from typing import List, Union
from pydantic import BaseModel


class User(BaseModel):
    id: int
    name = "John Doe"
    signup_ts: Union[datetime, None] = None
    friends: List[int] = []


external_data = {
    "id": "123",
    "signup_ts": "2017-06-01 12:22",
    "friends": [1, "2", b"3"],
}
user = User(**external_data)
print(user)
# > User id=123 name='John Doe' signup_ts=datetime.datetime(2017, 6, 1, 12, 22) friends=[1, 2, 3]
print(user.id)
# > 123
```




FastAPI是完全基于Pydantic的。在之后的教程中我们还可以看到更多的例子。

> 当你使用没有默认值的`Optional`或者 `Union[SomethingNothing]`时，Pydantic 有一个特殊的行为，你可以在 Pydantic 文档中关于[必选字段](https://pydantic-docs.helpmanual.io/usage/models/#required-optional-fields)的更多信息。对于可以接收`None`的必需参数可以用`...`来和`Optional`配合使用。

---

## FastAPI中的类型提示：

**FastAPI** 利用这些类型提示来做下面几件事。

使用 **FastAPI** 时用类型提示声明参数可以获得：

- **编辑器支持**。

- **类型检查**。

...并且 **FastAPI** 还会用这些类型声明来：

- **定义参数要求**：声明对请求路径参数、查询参数、请求头、请求体、依赖等的要求。

- **转换数据**：将来自请求的数据转换为需要的类型。

- **校验数据**： 对于每一个请求：
  - 当数据校验失败时自动生成**错误信息**返回给客户端。



- 使用 OpenAPI **记录** API：
  - 然后用于自动生成交互式文档的用户界面。



听上去有点抽象。不过不用担心。你将在[ 教程 - 用户指南](https://fastapi.tiangolo.com/zh/tutorial/)[ ](https://fastapi.tiangolo.com/zh/tutorial/)中看到所有的实战。

最重要的是，通过使用标准的 Python 类型，只需要在一个地方声明（而不是添加更多的类、装饰器等），**FastAPI** 会为你完成很多的工作。

> 如果你已经阅读了所有教程，回过头来想了解有关类型的更多信息，[来自 ](https://mypy.readthedocs.io/en/latest/cheat_sheet_py3.html)`[mypy](https://mypy.readthedocs.io/en/latest/cheat_sheet_py3.html)`[ 的"速查表"](https://mypy.readthedocs.io/en/latest/cheat_sheet_py3.html)是不错的资源。



