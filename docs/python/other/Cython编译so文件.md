# Cython编译so文件

---

Demo1:

```python
from os import walk, path, system


def py2so():
	for root, dirs, files in walk('./app'):
		for file in files:
			if not file.startswith('setup_') and file.endswith('.py'):
				abs_f = path.abspath(path.join(root, file))
				aps_p = path.dirname(abs_f)
				setup_f = path.join(aps_p, f'setup_{file}')
				with open(setup_f, 'w', encoding='utf-8') as f:
					f.write(f"""from distutils.core import setup
from Cython.Build import cythonize

setup(name='Flask Demo Test', ext_modules=cythonize(r'{abs_f}'))
""")
				my_cmd = f'python {setup_f} build_ext --inplace'
				system(my_cmd)

	del_cmd = "find ./app -name '*.py' | xargs rm"
	system(del_cmd)
	del_cmd = "find ./app -name '*.c' | xargs rm"
	system(del_cmd)
	del_cmd = "rm -rf build"
	system(del_cmd)


def main():
	py2so()


if __name__ == '__main__':
	main()

```

Demo2:

```python
from os import walk, path, system, cpu_count
from concurrent.futures import ThreadPoolExecutor

thread_pool = ThreadPoolExecutor(max_workers=cpu_count() * 2)


def py2so():
	for root, dirs, files in walk('./app'):
		thread_pool.submit(kernel_code, *[files, root])
	thread_pool.shutdown(True)
	del_cmd = "find ./app -name '*.py' | xargs rm;find ./app -name '*.c' | xargs rm;rm -rf build"
	system(del_cmd)


def kernel_code(files, root):
	for file in files:
		if not file.startswith('setup_') and file.endswith('.py'):
			abs_f = path.abspath(path.join(root, file))
			aps_p = path.dirname(abs_f)
			setup_f = path.join(aps_p, f'setup_{file}')
			with open(setup_f, 'w', encoding='utf-8') as f:
				f.write(f"""from distutils.core import setup
from Cython.Build import cythonize

setup(name='Flask Demo Test', ext_modules=cythonize(r'{abs_f}'))
""")
			my_cmd = f'python {setup_f} build_ext --inplace'
			system(my_cmd)


def main():
	py2so()


if __name__ == '__main__':
	main()

```

