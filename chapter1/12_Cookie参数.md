# 12_Cookie 参数

Cookie参数的定义和`Path`， `Query`参数一样。

```python
from fastapi import FastAPI, Cookie
```

## 1. 声明Cookie参数：

声明 Cookie 参数的结构和之前声明 Query , Path 参数时相同。第一个值是参数的默认值，同时也可以传递所有验证参数或元数据注解参数：
```python
from fastapi import Cookie, FastAPI

app = FastAPI()


@app.get("/items/")
async def read_items(ads_id: str | None = Cookie(default=None)):  # 声明Cookie
    return {"ads_id": ads_id}
```

注意：要声明 Cookie参数，需要使用 `Cookie`，因为否则参数将被解释为查询参数。

> 技术细节：
> 
> `Cookie` `、Path` 、`Query`是兄弟类，它们都继承自公共的 `Param` 类
>
> 但请记住，当你从 fastapi 导入的 `Query`、`Path`、`Cookie` 或其他参数声明函数，这些实际上是返回特殊类的工厂函数。

---

## 总结：

使用 `Cookie` 声明 cookie参数，使用方式与 `Query` 和 `Path` 类似。