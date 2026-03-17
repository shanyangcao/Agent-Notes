## 1. 什么是 Pydantic

Pydantic 是一个 Python 库，用来：

- **定义数据结构**
- **自动校验数据**
- **自动类型转换**
- **生成结构化数据**（字典、JSON）
- **配合 FastAPI 实现接口参数校验**

一句话：**Pydantic 模型 = 带校验规则的数据类。**

## 2. 核心作用

1. **数据校验**：类型不对、格式不对直接报错
2. **类型自动转换**：字符串数字 → int/float
3. **结构化数据**：接口入参、出参统一格式
4. **自动生成文档**：FastAPI 自动用它生成 `/docs`
5. **ORM 模型转换**：数据库对象 ↔ 接口数据

## 3. 基础使用

### 3.1 安装

```
pip install pydantic
```

### 3.2 定义最简单的模型

```
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    email: str
```

这就是一个 **Pydantic 模型**。

## 4. 自动校验 & 自动类型转换

```
user = User(name="张三", age="20", email="xxx@qq.com")
print(user.age)  # 输出 int 20（自动转）
```

传错类型会直接报错：

```
user = User(name="张三", age="abc", email="xxx")
# 直接抛出 ValidationError
```

## 5. 常用字段校验

```
from pydantic import BaseModel, Field, EmailStr

class User(BaseModel):
    name: str = Field(..., min_length=2, max_length=20)
    age: int = Field(..., ge=18, le=100)
    email: EmailStr  # 自动校验邮箱格式
```

常用规则：

- `min_length` / `max_length`：长度
- `ge`：大于等于
- `gt`：大于
- `le`：小于等于
- `lt`：小于
- `...`：表示必填

## 6. 模型继承

```
class UserBase(BaseModel):
    name: str
    age: int

class UserCreate(UserBase):
    password: str

class UserOut(UserBase):
    id: int
```

实际开发中非常常用：

- `Base`：公共字段
- `Create`：创建时需要的字段
- `Out`：返回给前端的字段

## 7. 模型转字典 / JSON

```
user = User(name="张三", age=20, email="xxx@qq.com")

# 转字典
user.dict()

# 转 JSON 字符串
user.json()
```

## 8. 嵌套模型

```
class Address(BaseModel):
    city: str
    street: str

class User(BaseModel):
    name: str
    address: Address  # 嵌套模型
```

## 9. 列表、泛型

```
from typing import List

class UserOut(BaseModel):
    id: int
    name: str

class UserList(BaseModel):
    total: int
    items: List[UserOut]
```

## 10. ORM 模式

FastAPI + SQLAlchemy 必用：

```
class UserOut(BaseModel):
    id: int
    name: str

    class Config:
        orm_mode = True
```

这样就能直接：

```
user_out = UserOut.from_orm(db_user)
```

## 11. 可选字段 & 默认值

```
class User(BaseModel):
    name: str
    age: int | None = None  # 可选
    is_vip: bool = False    # 默认值
```

## 12. Pydantic 在 FastAPI 中的作用

1. **请求体校验**

```
@app.post("/user")
def create_user(user: User):
    return user
```

1. **查询参数校验**
2. **路径参数校验**
3. **自动生成接口文档** `**/docs**`
4. **统一返回格式**