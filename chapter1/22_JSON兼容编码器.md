# 22_JSON兼容编码器

在某些情况下我们可能需要将一种数据类型（如，Pydantic的model）转换成与JSON相兼容的类型（如 dict、list 等）。

例如，你想将它存放在数据库中。

FastAPI对此提供了一个`jsonable_encoder()`函数。

---

## 1. 使用`jsonable_encoder`

假设我们有一个数据库`fake_db`, 它只接收与JSON兼容的数据。比如，它不接收`datetime`对象，因为它不是JSON兼容的数据类型。

因此，一个`datetime`对象必须要被转换为对应[ISO 格式](https://en.wikipedia.org/wiki/ISO_8601)时间数据的`str`类型对象。

同样的，这个数据库也不接收 Pydantic model类型（该类型对象带有属性），只能接收字典数据。

我们可以使用`jsonable_encoder`方法对其进行转换。

它接收类似 Pydantic model 的对象，返回一个与JSON兼容的版本：

```python
# python 3.10
from datetime import datetime

from fastapi import FastAPI
from fastapi.encoders import jsonable_encoder
from pydantic import BaseModel

fake_db = {}


class Item(BaseModel):
    title: str
    timestamp: datetime
    description: str | None = None


app = FastAPI()


@app.put("/items/{id}")
def update_item(id: str, item: Item):
    json_compatible_item_data = jsonable_encoder(item)  # 转换item
    fake_db[id] = json_compatible_item_data
```

以上代码中，Pydantic model 对象会被转换为一个 `dict`，`datetime`对象会被转换为`str`。

调用它的结果后就可以使用Python标准的`json.dumps()`方法对其进行编码了。

该方法不会返回一个包含JSON格式(`str`格式)数据的长`str`。而是返回一个Python标准的数据结构（如`dict`)，其中的值和子值都和JSON兼容。

> 注意：
> 
> `jsonable_encoder()`实际上是FastAPI内部用来转换数据的。不过在其他一些场景中也很有用。