# python 列表和字典排序

# 一、函数介绍

### 1.sorted()

sorted 函数：

```
sorted(iterable[,key][,reverse])
```

函数接收三个参数：排序的变量、排序的规则、升降序选择

依据 lambda 的解释，关注的主要是上述例子中 k: 之后的表达式，其中 k 应该代表 sorted()函数默认的 key 值。
比如：在 3 中，字典默认排序 key 是字典的键，所以 lambda 中 k 代表字典的键，想以字典的值排序，就应该是让排序中的 key=dict[k]
在 4 中，列表默认排序 key 是列表中的元素，此处列表中的元素依旧是嵌入的列表，所以排序 key 取嵌入列表的第一项时，可以 key=k[0]

## 2.lambda

lambda 匿名函数
一般形式

```
lambda arguments: expression
```

写成函数形式

```
def <lambda>(arguments):
        return expression
```

# 二、排序

## 1 简单列表(list)排序

```
list = ['a', 'b', 'c']
print(sorted(list))
# ['a', 'b', 'c']
print(sorted(list, reverse=True))
# ['c', 'b', 'a']
```

## 2 字典(dict)的键(key)排序

```
dict = {'c': 1, 'b': 2, 'a': 3}
print(sorted(dict))
# ['a', 'b', 'c']
print(sorted(dict, reverse=True))
# ['c', 'b', 'a']
```

## 3 字典(dict)的值(value)排序

使用 lambda 函数让 key 为字典值即 dict[]

```
dict = {'c': 1, 'b': 2, 'a': 3}
print(sorted(dict, key=lambda k: dict[k]))
# ['c', 'b', 'a']
print(sorted(dict, key=lambda k: dict[k], reverse=True))
# ['a', 'b', 'c']
```

## 4 列表(list)内嵌套列表(list)排序

```
list = [[4, 2, 9], [1, 5, 6], [7, 8, 3]]
# 以列表中列表的第一个数排序
print(sorted(list, key=lambda k: k[0]))
# [[1, 5, 6], [4, 2, 9], [7, 8, 3]]
# 以列表中列表的第二个数排序
print(sorted(list, key=lambda k: k[1]))
# [[4, 2, 9], [1, 5, 6], [7, 8, 3]]
# 以列表中列表的第三个数排序
print(sorted(list, key=lambda k: k[2]))
# [[7, 8, 3], [1, 5, 6], [4, 2, 9]]
# 以列表中列表的第一个数排序，且降序
print(sorted(list, key=lambda k: k[0], reverse=True))
# [[7, 8, 3], [4, 2, 9], [1, 5, 6]]
```

## 5 字典(dict)内嵌套字典(dict)排序

```
dict = {
        'a': {'x': 3, 'y': 2, 'z': 1},
        'b': {'x': 2, 'y': 1, 'z': 3},
        'c': {'x': 1, 'y': 3, 'z': 2}
}
# 以内部字典的'x'对应的值排序
print(sorted(dict, key=lambda k: dict[k]['x']))
# ['c', 'b', 'a']
# 以内部字典的'y'对应的值排序
print(sorted(dict, key=lambda k: dict[k]['y']))
# ['b', 'a', 'c']
# 以内部字典的'z'对应的值排序
print(sorted(dict, key=lambda k: dict[k]['z']))
# ['a', 'c', 'b']
# 以内部字典的'x'对应的值排序,并降序
print(sorted(dict, key=lambda k: dict[k]['x'], reverse=True))
# ['a', 'b', 'c']
```

## 6 列表(list)中嵌套字典(dict)排序

```
list = [
    {'x': 3, 'y': 2, 'z': 1},
    {'x': 2, 'y': 1, 'z': 3},
    {'x': 1, 'y': 3, 'z': 2},
]
print(sorted(list, key=lambda k: k['x']))
# [{'z': 2, 'x': 1, 'y': 3}, {'z': 3, 'x': 2, 'y': 1}, {'z': 1, 'x': 3, 'y': 2}]
print(sorted(list, key=lambda k: k['y']))
# [{'z': 3, 'x': 2, 'y': 1}, {'z': 1, 'x': 3, 'y': 2}, {'z': 2, 'x': 1, 'y': 3}]
print(sorted(list, key=lambda k: k['z']))
# [{'z': 1, 'x': 3, 'y': 2}, {'z': 2, 'x': 1, 'y': 3}, {'z': 3, 'x': 2, 'y': 1}]
print(sorted(list, key=lambda k: k['x'], reverse=True))
# [{'z': 1, 'x': 3, 'y': 2}, {'z': 3, 'x': 2, 'y': 1}, {'z': 2, 'x': 1, 'y': 3}]
```

## 7.字典(dict)中嵌套列表(list)排序

```
dic = {
    'a': [1, 2, 3],
    'b': [2, 1, 3],
    'c': [3, 1, 2],
}
print(sorted(dic, key=lambda k: dic[k][0]))
# ['a', 'b', 'c']
print(sorted(dic, key=lambda k: dic[k][1]))
# ['b', 'c', 'a']
print(sorted(dic, key=lambda k: dic[k][2]))
# ['c', 'b', 'a']
print(sorted(dic, key=lambda k: dic[k][0], reverse=True))
# ['c', 'b', 'a']

```

## 8.字典(dict)中嵌套列表(list)排序, 输出排序后的字典

```
dic = {
    'a': [1, 2, 3],
    'b': [2, 6, 3],
    'c': [3, 1, 2],
}
# 根据value的第1项和第2项的和来排序（从大到小）
# item[1]表示字典的value
print({k: v for k, v in sorted(dic.items(), key=lambda item: item[1][0] + item[1][1], reverse=True )})
# {'b': [2, 6, 3], 'c': [3, 1, 2], 'a': [1, 2, 3]}

```
