# 分析brew无用的包

---

```python
"""分析指定包的依赖, 得出可以删除的无用的包"""

from os import popen

my_package = ['mariadb', 'postgresql@14', 'redis', 'python@3.10']
brew_list = [i.strip() for i in popen('brew list').readlines()]
dependencies = []
done = []


def demo(packages):
    for package in packages:
        if package in done:
            continue
        done.append(package)
        cmd = f'brew info {package}|grep Required:|cut -d : -f 2'
        ret = popen(cmd).read().strip()
        if ret:
            depend_packages = [_.strip() for _ in ret.split(',')]

            for depend_package in depend_packages:
                if depend_package.strip() not in dependencies:
                    dependencies.append(depend_package)

            demo(depend_packages)


demo(my_package)


brew_list_set = set(brew_list)
my_package.extend(dependencies)
dependencies_set = set(my_package)

print('这些包需要使用brew uninstall命令删除:')
print(brew_list_set - dependencies_set)

```

