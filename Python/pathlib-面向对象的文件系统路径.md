### 面向对象的文件系统路径

> `pathlib`模块提供表示文件系统路径的类，其语义适用于不同的操作系统。路径类被分为提供纯计算操作而没有I/O的纯路径，以及从纯路径继承而来但提供I/O操作的具体路径。

![Pathlib文件路径模型](../../../WorkSpace/notes/img/pathlib-6610433.png)

#### 基本用法

##### 导入主类

```python
from pathlib import Path
```

##### 路径解析

```python
>>> p = Path('.')
>>> p.resolve()
PosixPath('/Users/fanyun/Documents')   # 这里返回的是`PosixPath`类,代表是Unix下的路径
```

##### 获取文件名

```python
>>> files = [x for x in p.iterdir()]
>>> files[0]
PosixPath('.config')
>>> files[0].name   # 使用`name`属性获取指定路径对象的名称
'.config'
```

##### 获取文件名后缀

```python
>>> files[0].stem   # 使用`stem`属性获取指定路径对象的后缀部分
'.config'
```

##### 路径拼接

```python
>>> p = Path('/Users')
>>> p = Path(p, 'fanyun')
>>> p
PosixPath('/Users/fanyun')
```

##### 获取父目录

```python
>>> p.parent     # 获取上一级目录
PosixPath('/Users')
```

##### 创建文件夹

```python
>>> p = Path('/Users/fanyun/Workspace/test')  # 创建`test`目录时, 上一级目录必须存在,否则报错
>>> p.mkdir(exist_ok=True)

>>> p.mkdir(exist_ok=True, parents=True)   # 递归创建目录
```

##### 列出子目录

```python
>>> [x for x in p.iterdir() if x.is_dir()]   # iterdir() 方法用于遍历当前目录下的文件/文件夹
[PosixPath('产品使用手册'), PosixPath('test'), PosixPath('稳定性测试'), PosixPath('QData-Tools'), PosixPath('TencentMeeting')]
```

##### 查询路径属性

```python
>>> p.is_dir()   # 查询路径是否为目录
True
>>> p.exists()   # 查询路径是否存在
True
>>> p.is_file()  # 查询路径是否为文件
False
```





#### 对应的`os`模块的工具

| os  和 os.path                                               | Path lib                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`os.path.abspath()`](https://docs.python.org/zh-cn/3/library/os.path.html#os.path.abspath) | [`Path.resolve()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.Path.resolve) |
| [`os.mkdir()`](https://docs.python.org/zh-cn/3/library/os.html#os.mkdir) | [`Path.mkdir()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.Path.mkdir) |
| [`os.makedirs()`](https://docs.python.org/zh-cn/3/library/os.html#os.makedirs) | [`Path.mkdir()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.Path.mkdir) |
| [`os.rename()`](https://docs.python.org/zh-cn/3/library/os.html#os.rename) | [`Path.rename()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.Path.rename) |
| [`os.replace()`](https://docs.python.org/zh-cn/3/library/os.html#os.replace) | [`Path.replace()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.Path.replace) |
| [`os.getcwd()`](https://docs.python.org/zh-cn/3/library/os.html#os.getcwd) | [`Path.cwd()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.Path.cwd) |
| [`os.path.exists()`](https://docs.python.org/zh-cn/3/library/os.path.html#os.path.exists) | [`Path.exists()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.Path.exists) |
| [`os.listdir()`](https://docs.python.org/zh-cn/3/library/os.html#os.listdir) | [`Path.iterdir()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.Path.iterdir) |
| [`os.path.isdir()`](https://docs.python.org/zh-cn/3/library/os.path.html#os.path.isdir) | [`Path.is_dir()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.Path.is_dir) |
| [`os.path.isfile()`](https://docs.python.org/zh-cn/3/library/os.path.html#os.path.isfile) | [`Path.is_file()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.Path.is_file) |
| [`os.path.isabs()`](https://docs.python.org/zh-cn/3/library/os.path.html#os.path.isabs) | [`PurePath.is_absolute()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.PurePath.is_absolute) |
| [`os.path.join()`](https://docs.python.org/zh-cn/3/library/os.path.html#os.path.join) | [`PurePath.joinpath()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.PurePath.joinpath) |
| [`os.path.basename()`](https://docs.python.org/zh-cn/3/library/os.path.html#os.path.basename) | [`PurePath.name`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.PurePath.name) |
| [`os.path.dirname()`](https://docs.python.org/zh-cn/3/library/os.path.html#os.path.dirname) | [`PurePath.parent`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.PurePath.parent) |
| [`os.path.samefile()`](https://docs.python.org/zh-cn/3/library/os.path.html#os.path.samefile) | [`Path.samefile()`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.Path.samefile) |
| [`os.path.splitext()`](https://docs.python.org/zh-cn/3/library/os.path.html#os.path.splitext) | [`PurePath.suffix`](https://docs.python.org/zh-cn/3/library/pathlib.html#pathlib.PurePath.suffix) |

