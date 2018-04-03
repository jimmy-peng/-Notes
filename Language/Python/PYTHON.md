# PYTHON

### 目录

[***集合数据类型筛选***](#sld-pane)

[***集合可读索引***](#set-pane)

[***统计词频***](#statics-pane)

[***字典值排序***](#sort-pane)

[***字典公共键***](#inter-pane)

[***对象持久化存储***](#obj-pane)

---

### <a name="sld-pane">筛选</a>
***列表***

* Filter

```
data = [randomint(-10, 10) for _ in range(10)]
newlist = filter(lamda x: x > 0, data)
```
* 列表解析

```
data = [randomin(-10, 10) for _ in range(10)]
newlist = [x 		  for x in data     if x > 0]
		表达式		      循环			 条件
```

***字典***

* Filter

```

dict = filter(lambda x: x[0] > 10, dict.items())

{x[0]: x[1]  for x in data}
 key  value
```
* 列表解析

```
data = {x: randint(1, 20) for x in range(1 , 100)}        
newdict = {k: v        for k, v in data.items()     if v > 10}
			表达式			循环				            条件
```

***集合***

* Filter

```
data = {randint(1, 23) for _ in range(10)}
#filter返回迭代器
set = filter(lambda x: x > 10, data)
```

* 列表解析

```

set = {randint(0, 100) for _ in range(10)}
```

### <a name="set-pane">集合可读索引</a>

* 变量索引

```
NAME = 0
AGE  = 1
GENDER = 2
EMAIL = 3
student = ("Jim", 18, "male", "jim@126.com")
student[NAME]

```
* collections.nametuple索引

```
from collections import namedtuple

Student = namedtuple('Student', ['name', 'age', 'gender', 'email'])
新类					新类名			元素名称列表
s 		= Student("Jim", 18, 'male','jim@126.com')
新对象					数据
print(s.name)
```

### <a name="statics-pane">统计词频</a>

* 常规方法

```
#生成有重复元素的列表
data = [randint(1,10) for _ in range(1, 100)]
#列表转换成字典
d = dict.fromkeys(data, 		0)
				列表元素作为键值   初始值为0
for x in date:
    d[x] += 1
```

* collections

```
from collections import Counter

data = [randint(1,10) for _ in range(1, 100)]

counter = Counter(data)
print(counter.most_common(  		2		))
							词频最大的两个元素
counter[		4		]
		元素是4出现的频率
```

### <a name="sort-pane">字典值排序</a>
* 常规方法

```
sorted(d.items(), 		key=lambda x: x[1])
		(k, v)元组列表 		使用元组第二元素排序
```

* 库函数

```
from collections import OrderedDict

D = OrderedDict()
D["jim"] = (1, 23)
D["ad"] = (2, 234)
D["34"] = (2, 56)
print(D)
```

### <a name="inter-pane">字典公共键</a>

* 常规方法

```
A = {"X":1, "Y": 2}
B = {"Y": 2, "Z": 1}
A & B = {"Y"}

s1 = {x: randint(1, 4) for x in sample("abcdef", randint(3, 6))}
s2 = {x: randint(1, 4) for x in sample("abcdef", randint(3, 6))}
s3 = {x: randint(1, 4) for x in sample("abcdef", randint(3, 6))}

print(s1.keys()& s2.keys())
```

* mapRecude

```
#转换集合
newlist = map(lambda x: x.keys(),[s1, s2, s3])
reduce(lambda x, y: x & y ,newlist)
```

### <a name="obj-pane">对象持久化存储</a>

```
from random import randint
from collections import deque
import pickle
import os


def guess_and_recode(n, right_number):

    if n == right_number:
        print("You guess right")
        return True
    elif n > right_number:
        print("You guess is too big")
    else:
        print("You guess is too little")
    return False


def generate_lucky_number():
    return randint(0, 100)


def load_record_from_file(filename):
    with open(filename, 'rb+') as f:
        obj = pickle.load(f)

    return obj


def dump_record_to_file(obj, filename):
    # 对象存储必须使用字节存储
    with open(filename, 'wb+') as f:
        pickle.dump(obj, f)


if __name__ == "__main__":
    filepath = "./history"
    magic = generate_lucky_number()
    if os.path.exists(filepath):
        historic_queue = load_record_from_file(filepath)
    else:
        historic_queue = deque([], 5)
    while True:
        toGuessed = input("Please input your guess number:")
        if toGuessed.isdigit():
            toGuessed = int(toGuessed)
            historic_queue.append(toGuessed)
            if guess_and_recode(toGuessed, magic):
                break
        elif toGuessed == "h?":
            dump_record_to_file(historic_queue, filepath)
            print("History recode:", list(historic_queue))
        else:
            exit(0)
```