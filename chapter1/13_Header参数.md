# 13_Header参数

Header参数的定义方法和前面的`Query`、`Path`、`Cookie`参数一样。

```python
from fastapi import FastAPI, Header
```

---

## 1. 声明Header参数：

和之前一样，`Header` 参数第一个值是默认值，后面可以传递所有的额外验证或元数据注解：
```python
from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(user_agent: str | None = Header(default=None)):
    return {"User-Agent": user_agent}
```

为了声明headers， 你需要使用`Header`, 因为否则参数将被解释为查询参数。

> 技术细节：
> 
> `Header` 是 `Path`, `Query` 和 `Cookie` 的兄弟类型。它也继承自通用的 `Param` 类.
>
> 但是请记得，当你从fastapi导入 `Query`, `Path`, `Header`, 或其他时，实际上导入的是返回特定类型的工厂函数。

---

## 2. 自动转换：

`Header` 在 `Path`, `Query` 和 `Cookie` 提供的功能之上有一点额外的功能。

大多数标准的headers用 "连字符" 分隔，也称为 "减号" (`-`)。

但是像 `user-agent` 这样的变量在Python中是无效的。因此, 默认情况下, Header 将把参数名称的字符从下划线 (`_`) 转换为连字符 (`-`) 来提取并记录 headers.

同时，HTTP headers 是`大小写不敏感`的，因此，因此可以使用标准Python样式(也称为 `snake_case`)声明它们。因此，你可以像通常在Python代码中那样使用 `user_agent` ，而不需要将首字母大写为 `User_Agent` 或类似的东西。

如果出于某些原因，你需要禁用下划线到连字符的自动转换，设置`Header`的参数 `convert_underscores` 为 `False`:

```python
from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(
    strange_header: str | None = Header(default=None, convert_underscores=False)
):
    return {"strange_header": strange_header}
```

> 注意：在设置 `convert_underscores` 为 `False` 之前，请记住，一些HTTP 代理（proxies）和服务器不允许使用带有下划线的headers。

---

## 3. 重复的headers：

有可能会收到重复的headers，这意味着，同一个header有多个值。

此时，可以在类型声明中使用一个`list`来定义这些情况。通过一个Python list 的形式获得重复header的所有值。

比如, 为了声明一个 `X-Token` header 可以出现多次，你可以这样写：
```python
from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(x_token: list[str] | None = Header(default=None)):
    return {"X-Token values": x_token}
```

如果调用API时给路径操作发送了两个 HTTP headers，就像：
```
X-Token: foo
X-Token: bar
```
则上面代码的响应就是：
```json
{
    "X-Token values": [
        "bar",
        "foo"
    ]
}
```

---

## 总结：

使用 `Header` 来声明 header , 使用和 `Query`, `Path` 与 `Cookie` 相同的模式。

不用担心变量中的`下划线`，FastAPI 会负责转换它们。