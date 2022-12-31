# AtExit
---
## 代码

```python
import atexit
import sys


def run_func_at_exit_1():
    print(f'func name is {sys._getframe().f_code.co_name}')
    print(f'file name is {sys._getframe().f_code.co_filename}')


def run_func_at_exit_2(hello, world):
    print(f'{hello} {world}')


atexit.register(run_func_at_exit_1)
atexit.register(run_func_at_exit_2, 'Good', 'Morning')


@atexit.register
def run_func_at_exit_3():
    run_func_at_exit_2('Hi,', 'How are you')
```



## 输出

```
Hi, How are you
Good Morning
func name is run_func_at_exit_1
file name is /root/code/python/my_atexit.py
```



## 学习点

- `sys._getframe().f_code.co_name` 可以输出当前运行函数的名称
- `sys._getframe().f_code.co_filename`可以输出当前运行函数所在模块的绝对路径
- `atexit`模块的`register`函数注册的其他函数会在当前模块的代码从上而下执行完毕后自动调用(不需要 `__name__ == '__main__'`)
- atexit.register()和@atexit.register是两种使用方法, 其运行顺序是最后注册的最先运行
- 如果程序是非正常crash，或通过os._exit()退出，注册的回调函数将不会被调用
- **atexit.unregister(func) 可取消注册函数**