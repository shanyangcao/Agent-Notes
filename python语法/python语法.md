# Python 语法笔记

本文是学习 Python 过程中整理的基础语法笔记，适合初学者系统梳理知识点。

### 变量与命名

变量就是给数据起的名字。Python 的变量名有几条硬性规则，违反了就会报错：

| 规则                     | 正确示例    | 错误示例         |
| ------------------------ | ----------- | ---------------- |
| 只能用字母、数字、下划线 | `user_name` | `user-name`      |
| 不能以数字开头           | `data_1`    | `1_data`         |
| 不能有空格               | `my_score`  | `my score`       |
| 不能是关键字             | `total`     | `class`、`print` |

除了规则之外，还有一些**约定俗成的习惯**：

- 变量名用小写字母，多个单词用下划线连接，比如 `student_age`
- 见名知意，不要用 `a`、`b`、`x` 这种无意义的名字（除非是临时循环变量）
- 类名用大驼峰，比如 `StudentInfo`

```
# 好的命名
student_name = "小明"
total_score = 95
is_logged_in = True

# 不好的命名（虽然不报错，但可读性差）
a = "小明"
ts = 95
flag = True
```

------

### 基础数据类型

#### Python 最常用的数据类型：

| 类型       | 说明   | 示例             |
| ---------- | ------ | ---------------- |
| `int`      | 整数   | `age = 18`       |
| `float`    | 小数   | `price = 9.9`    |
| `str`      | 字符串 | `name = "Alice"` |
| `bool`     | 布尔值 | `is_ok = True`   |
| `NoneType` | 空值   | `result = None`  |

用 `type()` 可以查看变量的类型，用 `int()`、`str()`、`float()` 可以进行类型转换：

```
x = "123"
print(type(x))      # <class 'str'>

x = int(x)
print(type(x))      # <class 'int'>
print(x + 1)        # 124
```

💡 **注意**：`True` 和 `False` 在运算中等价于 `1` 和 `0`，比如 `True + 1` 的结果是 `2`。

### 字符串

字符串是用引号括起来的文字内容，单引号和双引号都可以，效果完全一样。

#### 创建与拼接

```
s1 = "Hello"
s2 = 'World'

# 用 + 拼接
s3 = s1 + " " + s2     # "Hello World"

# 用 * 重复
s4 = "ha" * 3           # "hahaha"

# f-string（推荐写法，Python 3.6+）
name = "小明"
age = 18
s5 = f"我叫{name}，今年{age}岁"   # "我叫小明，今年18岁"
```

#### 常用方法

```
s = "  Hello, Python!  "

# 去除空格
s.strip()       # "Hello, Python!"   去掉两端空格
s.lstrip()      # "Hello, Python!  " 去掉左边空格
s.rstrip()      # "  Hello, Python!" 去掉右边空格

# 大小写转换
"hello".upper()       # "HELLO"
"HELLO".lower()       # "hello"
"hello world".title() # "Hello World"

# 替换
"Hello World".replace("World", "Python")  # "Hello Python"

# 分割
"a,b,c".split(",")    # ['a', 'b', 'c']

# 拼接列表为字符串
",".join(["a", "b", "c"])  # "a,b,c"

# 查找
"Hello".find("ll")    # 2（返回起始索引）
"Hello".count("l")    # 2（出现次数）
```

#### 索引与切片

字符串的每个字符都有一个编号（索引），**从 0 开始**，也可以用负数从尾部数：

```
字符串：  H  e  l  l  o
正向索引：0  1  2  3  4
反向索引：-5 -4 -3 -2 -1
```

```
s = "Hello"

s[0]      # "H"   取第一个字符
s[-1]     # "o"   取最后一个字符
s[1:4]    # "ell" 切片，取索引 1、2、3（不含 4）
s[:3]     # "Hel" 从头取到索引 3
s[2:]     # "llo" 从索引 2 取到末尾
s[::-1]   # "olleH" 反转字符串
```

💡 切片规则是**左闭右开**：`s[1:4]` 取的是索引 1、2、3，不包含索引 4。

#### 求长度

```
len("Hello")   # 5
len("你好")    # 2（汉字也算 1 个字符）
```

### 列表

列表用**方括号** `[]` 表示，可以存放任意类型的数据，元素之间用逗号分隔。

#### 创建

```
# 空列表
fruits = []

# 直接创建
fruits = ["苹果", "香蕉", "橙子"]

# 混合类型
mixed = [1, "hello", True, 3.14]

# 用 range 创建数字列表
nums = list(range(1, 6))      # [1, 2, 3, 4, 5]
nums = list(range(0, 10, 2))  # [0, 2, 4, 6, 8]  步长为 2

# 列表推导式（简洁写法）
squares = [x**2 for x in range(1, 6)]  # [1, 4, 9, 16, 25]
```

#### 增加元素

```
fruits = ["苹果", "香蕉"]

fruits.append("橙子")          # 末尾添加 → ["苹果", "香蕉", "橙子"]
fruits.insert(1, "葡萄")       # 指定位置插入 → ["苹果", "葡萄", "香蕉", "橙子"]
```

#### 删除元素

```
fruits = ["苹果", "香蕉", "橙子", "葡萄"]

del fruits[0]              # 按索引删除
fruits.pop()               # 删除并返回最后一个元素
fruits.pop(1)              # 删除并返回索引 1 的元素
fruits.remove("香蕉")      # 按值删除（只删第一个匹配的）
```

#### 查询与访问

```
fruits = ["苹果", "香蕉", "橙子"]

fruits[0]         # "苹果"
fruits[-1]        # "橙子"
fruits[0:2]       # ["苹果", "香蕉"]  切片同字符串

# 遍历
for fruit in fruits:
    print(fruit)

# 带索引遍历
for i, fruit in enumerate(fruits):
    print(i, fruit)

# 判断是否存在
"香蕉" in fruits    # True
"西瓜" not in fruits  # True
```

#### 修改元素

```
fruits = ["苹果", "香蕉", "橙子"]
fruits[1] = "西瓜"   # ["苹果", "西瓜", "橙子"]
```

#### 其他常用操作

```
nums = [3, 1, 4, 1, 5, 9, 2, 6]

len(nums)           # 8   长度
max(nums)           # 9   最大值
min(nums)           # 1   最小值
sum(nums)           # 31  求和

nums.sort()         # 原地排序（永久改变）
sorted(nums)        # 返回排好序的新列表（不改变原列表）
nums.reverse()      # 原地反转

# 复制列表（用切片，避免共享引用）
copy = nums[:]
```

### 元组

元组用圆括号 `()` 表示，和列表很像，但**创建之后不能修改**。

```
# 创建
point = (3, 4)
colors = ("红", "绿", "蓝")
single = (1,)    # 只有一个元素时，必须加逗号，否则不是元组

# 访问（和列表一样）
point[0]     # 3
point[-1]    # 4
point[0:2]   # (3, 4)

# 遍历
for color in colors:
    print(color)

# 不能修改（会报错）
# point[0] = 10   ← TypeError
```

#### 什么时候用元组而不是列表？

- 数据不应该被修改时，比如坐标、RGB 颜色值、函数返回多个值
- 作为字典的键（列表不能当键，元组可以）
- 语义上表示"固定结构"的数据

### 字典

字典用花括号 `{}` 表示，存储**键值对**，通过键来查找值，而不是通过位置。

#### 创建

```
# 直接创建
student = {
    "name": "小明",
    "age": 18,
    "score": 95
}

# 空字典
info = {}
```

#### 增加与修改

```
student = {"name": "小明", "age": 18}

# 键不存在 → 新增
student["score"] = 95

# 键已存在 → 修改
student["age"] = 19

# 批量更新
student.update({"age": 20, "city": "北京"})
```

#### 删除

```
student = {"name": "小明", "age": 18, "score": 95}

del student["score"]                  # 直接删除
age = student.pop("age")              # 删除并返回值
age = student.pop("age", 0)           # 键不存在时返回默认值 0，不报错
last = student.popitem()              # 删除并返回最后插入的键值对
```

#### 查询

```
student = {"name": "小明", "age": 18}

student["name"]                  # "小明"（键不存在会报错）
student.get("name")              # "小明"（键不存在返回 None）
student.get("score", 0)          # 键不存在时返回默认值 0

"name" in student                # True  判断键是否存在
```

#### 遍历

```
student = {"name": "小明", "age": 18, "score": 95}

# 遍历键
for key in student:
    print(key)

# 遍历值
for value in student.values():
    print(value)

# 同时遍历键和值（最常用）
for key, value in student.items():
    print(f"{key}: {value}")
```

### 条件判断

#### 基本结构

```
score = 85

if score >= 90:
    print("优秀")
elif score >= 70:
    print("良好")
elif score >= 60:
    print("及格")
else:
    print("不及格")
```

#### 条件运算符

| 运算符         | 含义                 | 示例              |
| -------------- | -------------------- | ----------------- |
| `==`           | 等于                 | `a == b`          |
| `!=`           | 不等于               | `a != b`          |
| `>``<``>=``<=` | 比较大小             | `a > b`           |
| `and`          | 且，两个条件都满足   | `a > 0 and b > 0` |
| `or`           | 或，满足其中一个即可 | `a > 0 or b > 0`  |
| `not`          | 非，取反             | `not a > 0`       |
| `in`           | 是否在其中           | `"a" in lst`      |
| `not in`       | 是否不在其中         | `"a" not in lst`  |

```
age = 20
city = "北京"

if age >= 18 and city == "北京":
    print("符合条件")

# 检查元素是否在列表中
allowed = ["苹果", "香蕉", "橙子"]
if "苹果" in allowed:
    print("可以吃")
```

------

### 循环

#### for 循环

```
# 遍历列表
fruits = ["苹果", "香蕉", "橙子"]
for fruit in fruits:
    print(fruit)

# 遍历数字范围
for i in range(5):          # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 6):       # 1, 2, 3, 4, 5
    print(i)

for i in range(0, 10, 2):   # 0, 2, 4, 6, 8
    print(i)

# 遍历字典
info = {"name": "小明", "age": 18}
for key, value in info.items():
    print(key, value)
```

#### while 循环

```
count = 0
while count < 5:
    print(count)
    count += 1
```

### break 和 continue

```
# break：直接跳出循环
for i in range(10):
    if i == 5:
        break        # 遇到 5 就停
    print(i)         # 输出 0 1 2 3 4

# continue：跳过本次，继续下一次
for i in range(10):
    if i % 2 == 0:
        continue     # 跳过偶数
    print(i)         # 输出 1 3 5 7 9
```

### 函数

函数就是把一段可以复用的代码打包起来，起个名字，以后直接调用。

#### 定义与调用

```
# 定义
def greet(name):
    print(f"你好，{name}！")

# 调用
greet("小明")     # 你好，小明！
greet("小红")     # 你好，小红！
```

#### 返回值

```
def add(a, b):
    return a + b

result = add(3, 5)   # result = 8

# 返回多个值（实际上返回的是元组）
def min_max(lst):
    return min(lst), max(lst)

low, high = min_max([3, 1, 4, 1, 5, 9])
# low = 1, high = 9
```

#### 参数类型

```
# 默认参数：调用时可以不传
def greet(name, msg="你好"):
    print(f"{msg}，{name}！")

greet("小明")              # 你好，小明！
greet("小红", "早上好")    # 早上好，小红！

# 关键字参数：传参时指定名称，顺序可以随意
def describe(name, age, city):
    print(f"{name}，{age}岁，来自{city}")

describe(age=18, city="北京", name="小明")

# *args：接收任意数量的位置参数（打包成元组）
def total(*args):
    return sum(args)

total(1, 2, 3)      # 6
total(1, 2, 3, 4, 5)  # 15

# **kwargs：接收任意数量的关键字参数（打包成字典）
def show_info(**kwargs):
    for k, v in kwargs.items():
        print(f"{k}: {v}")

show_info(name="小明", age=18, city="北京")
```

#### 参数顺序规则

定义函数时，参数顺序必须遵守：

```
普通参数 → 默认参数 → *args → **kwargs
```

```
def func(a, b, c=10, *args, **kwargs):
    pass
```

### 类与对象

类是一个模板，用来描述一类事物共有的属性和行为。对象是根据模板创造出来的具体实例。

#### 定义类

```
class Dog:
    # 类属性：所有狗都有的属性
    species = "犬科"

    # 初始化方法：创建对象时自动调用
    def __init__(self, name, age):
        # 实例属性：每只狗自己的属性
        self.name = name
        self.age = age

    # 方法：狗能做的事
    def bark(self):
        print(f"{self.name} 说：汪汪！")

    def info(self):
        print(f"名字：{self.name}，年龄：{self.age}岁")
```

#### 创建对象与调用

```
# 创建对象（实例化）
dog1 = Dog("旺财", 3)
dog2 = Dog("小白", 1)

# 调用方法
dog1.bark()     # 旺财 说：汪汪！
dog2.info()     # 名字：小白，年龄：1岁

# 访问属性
print(dog1.name)        # 旺财
print(Dog.species)      # 犬科（类属性可以用类名访问）
print(dog1.species)     # 犬科（也可以用实例访问）

# 修改属性
dog1.age = 4            # 直接修改
```

#### 继承

子类可以继承父类的所有方法和属性，还可以新增或覆盖：

```
class Animal:
    def __init__(self, name):
        self.name = name

    def eat(self):
        print(f"{self.name} 在吃东西")

class Cat(Animal):   # Cat 继承 Animal
    def meow(self):
        print(f"{self.name} 说：喵～")

    # 覆盖父类方法
    def eat(self):
        print(f"{self.name} 优雅地吃东西")

cat = Cat("咪咪")
cat.eat()     # 咪咪 优雅地吃东西（调用子类的方法）
cat.meow()    # 咪咪 说：喵～
```

用 `super()` 调用父类的方法：

```
class GuideDog(Dog):
    def __init__(self, name, age, owner):
        super().__init__(name, age)   # 调用 Dog 的 __init__
        self.owner = owner            # 新增属性

    def guide(self):
        print(f"{self.name} 正在引导主人 {self.owner}")
```

#### 方法种类

| 方法类型 | 写法                               | 说明                                     |
| -------- | ---------------------------------- | ---------------------------------------- |
| 普通方法 | `def method(self)`                 | 操作实例属性，用实例名调用               |
| 类方法   | `@classmethod` + `def method(cls)` | 操作类属性，用类名调用                   |
| 静态方法 | `@staticmethod` + `def method()`   | 不依赖实例或类，相当于放在类里的普通函数 |

```
class MathHelper:
    pi = 3.14159

    @classmethod
    def get_pi(cls):
        return cls.pi

    @staticmethod
    def add(a, b):
        return a + b

MathHelper.get_pi()    # 3.14159
MathHelper.add(3, 5)   # 8
```

### 常用内置函数速查

| 函数                             | 说明               | 示例                           |
| -------------------------------- | ------------------ | ------------------------------ |
| `print()`                        | 打印输出           | `print("hello")`               |
| `input()`                        | 获取用户输入       | `name = input("请输入名字：")` |
| `len()`                          | 获取长度           | `len([1, 2, 3])` → `3`         |
| `type()`                         | 查看类型           | `type(123)` → `<class 'int'>`  |
| `int()``float()``str()`          | 类型转换           | `int("123")` → `123`           |
| `range()`                        | 生成数字序列       | `range(1, 6)`                  |
| `list()``tuple()``dict()``set()` | 类型转换           | `list(range(5))`               |
| `max()``min()``sum()`            | 最大、最小、求和   | `max([3,1,4])` → `4`           |
| `abs()`                          | 绝对值             | `abs(-5)` → `5`                |
| `round()`                        | 四舍五入           | `round(3.567, 2)` → `3.57`     |
| `sorted()`                       | 返回排序后的新列表 | `sorted([3,1,2])` → `[1,2,3]`  |
| `enumerate()`                    | 遍历时同时获取索引 | `for i, v in enumerate(lst)`   |
| `zip()`                          | 同时遍历多个列表   | `for a, b in zip(lst1, lst2)`  |

### 容易混淆的几个点

#### 赋值不是复制

```
a = [1, 2, 3]
b = a           # b 和 a 指向同一个列表，不是复制！
b.append(4)
print(a)        # [1, 2, 3, 4]  a 也变了

# 正确复制方式
b = a[:]        # 或 b = list(a)
```

#### `=` 和 `==` 的区别

```
x = 5       # 赋值，把 5 赋给 x
x == 5      # 比较，判断 x 是否等于 5，返回 True 或 False
```

#### 整数除法

```
10 / 3      # 3.3333...  普通除法，结果是小数
10 // 3     # 3          整除，只取整数部分
10 % 3      # 1          取余数
```

#### 列表 vs 元组选哪个

- 数据会变化 → 用**列表**
- 数据固定不变（比如坐标、配置） → 用**元组**
- 需要用作字典的键 → 只能用**元组**（列表不可哈希）

#### 字典 `[]` 和 `.get()` 的区别

```
d = {"name": "小明"}

d["age"]            # KeyError！键不存在直接报错
d.get("age")        # None，键不存在返回 None，不报错
d.get("age", 0)     # 0，键不存在时返回自定义默认值
```