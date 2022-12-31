# + += append extend 的区别

---

```python
def test1(li):
    """使用+操作两个列表"""
    raw_li = li
    print(f'原始li: \t{raw_li}')

    li  = li + li
    print(f'使用+操作后: \t{li}')

    print(f'原始li: \t{raw_li}')
    print()


def test2(li):
    """使用+=操作列表"""
    raw_li = li
    print(f'原始li: \t{raw_li}')

    li += li
    print(f'使用+=操作后: \t{li}')

    print(f'原始li: \t{raw_li}')
    print()


def test3(li: list):
    """使用append操作列表"""
    raw_li = li
    print(f'原始li: \t\t{raw_li}')

    li.append(4)
    print(f'使用append操作后: \t{li}')

    print(f'原始li: \t\t{raw_li}')
    print()


def test4(li: list):
    """使用extend操作列表"""
    raw_li = li
    print(f'原始li: \t\t{raw_li}')

    li.extend(li)
    print(f'使用extend操作后: \t{li}')

    print(f'原始li: \t\t{raw_li}')


def main():
    li = [1, 2, 3]
    test1(li)

    li = [1, 2, 3]
    test2(li)

    li = [1, 2, 3]
    test3(li)

    li = [1, 2, 3]
    test4(li)


if __name__ == '__main__':
    main()
```

输出:

```
原始li:         [1, 2, 3]
使用+操作后:     [1, 2, 3, 1, 2, 3]
原始li:         [1, 2, 3]

原始li:         [1, 2, 3]
使用+=操作后:    [1, 2, 3, 1, 2, 3]
原始li:         [1, 2, 3, 1, 2, 3]

原始li:                 [1, 2, 3]
使用append操作后:        [1, 2, 3, 4]
原始li:                 [1, 2, 3, 4]

原始li:                 [1, 2, 3]
使用extend操作后:        [1, 2, 3, 1, 2, 3]
原始li:                 [1, 2, 3, 1, 2, 3]
```

解释:

- 对于可变数据类型, 函数内部对其进行 +=、append和extend操作, 会更改原始数据

- 而 li=li+li, 则是在函数内部创建了一局部变量li, 不会对外面的li造成影响

- `+=` 内部使用了魔法方法`__iadd__`, `__iadd__`内部又调用了`extend`方法

拓展:

- `+` 只能操作类型一直的对象, 例如列表+列表, 如果是列表+远足, 则提示TyperError

- `extend`方法传入参数是可迭代对象(iterable), 所以可以 列表.extend(元组)、元组.extend(集合)等等