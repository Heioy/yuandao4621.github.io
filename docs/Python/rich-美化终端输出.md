#### 简介

`Rich`第三方库是用来将输出到终端的文本信息以更加美观、优雅的方式展示出来，能够带给人以美的感受。同时支持表格、markdown和代码的语法高亮。

#### 安装

```python
pip install rich
```

#### 使用

##### print()

`print`函数可以以更优化的方式输出python对象，如果打印的对象长度不需要分行显示，它将用一行的方式展示给你。`locals()`用于返回当前位置的全部局部变量，并以字典的方式展示；

```python
>>> from rich import print
>>> 
>>> print("[italic red]Hello[/italic red] World!", locals())
Hello World!
{
    '__name__': '__main__',
    '__doc__': None,
    '__package__': None,
    '__loader__': <class '_frozen_importlib.BuiltinImporter'>,
    '__spec__': None,
    '__annotations__': {},
    '__builtins__': <module 'builtins' (built-in)>,
    'Console': <class 'rich.console.Console'>,
    'rich': <module 'rich' from '/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/rich/__init__.py'>,
    'print': <function print at 0x7fa8800b5c20>
}
```

##### Console()

在 `rich` 中 `Console` 对象是一个重点对象了, 首先你需要实例化一个 `Console` 对象, 在进行使用, 而你实例化的对象在渲染的时候将会检测以下几个属性值：

- `force_terminal` 强制以终端中的形式写入(哪怕是文件中)

```python
from rich.console import Console

console = Console(force_terminal=True)
```

- `encoding` 是默认编码(通常为`utf-8`)
- `file` 打开的文件对象，将终端的信息写入到文件中

```python
from rich.console import Console
from rich import print as rprint

logfile = "/Users/fanyun/console.log"
with open(logfile, "w") as lf:
    console = Console(file=lf, force_terminal=True)
    console.print("test if to write to file")
    # console.print(console.width)
    console.print(console.get_datetime())
    console.log("test if to write to file")
    console.input("😀 please input the string: ")
    console.out("this is [out] print")
```

- `print`以美化的格式将信息输出到终端, 支持html语法

```python
>>> console.print("this is [bold]print[/bold] info")
this is print info
```

- `input`用来接收终端输入

```python
>>> console.input("😀 please input the string: ")
😀 please input the string: this is a test
```

- `clear`清除当前终端的内容，类似于Linux终端的`clear`

```python
>>> console.clear()
>>> 
```

- `line`将写入的内容放在新的一行，类似于回行

```python
>>> console.line()

>>>
```

- `out` 将信息输出到终端，输出的内容没有`console.log()`详细

```python
>>> console.out("this is [out] print")
this is [out] print
```

- `print_exception` 打印程序执行中报错产生的堆栈信息

```python
>>> try:
...     div = 1 / 0
... except Exception as e:
...     console.print_exception()
... 
╭─────────────────────────────── Traceback (most recent call last) ────────────────────────────────╮
│ <stdin>:2 in <module>                                                                            │
╰──────────────────────────────────────────────────────────────────────────────────────────────────╯
ZeroDivisionError: division by zero
```

- `save_text` 使用屏幕录制模式下，可以将终端的所有信息保存到文件中

```python
>>> console = Console(record=True, force_terminal=True)
>>> console.print("this is [bold]print[/bold] info")
this is print info
>>> console.out("this is [log] print")
this is [log] print
>>> console.print(console, locals())
<console width=187 ColorSystem.TRUECOLOR>
{
    '__name__': '__main__',
    '__doc__': None,
    '__package__': None,
    '__loader__': <class '_frozen_importlib.BuiltinImporter'>,
    '__spec__': None,
    '__annotations__': {},
    '__builtins__': <module 'builtins' (built-in)>,
    'Console': <class 'rich.console.Console'>,
    'rich': <module 'rich' from '/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/rich/__init__.py'>,
    'print': <function print at 0x7fa8800b5c20>,
    'pretty': <module 'rich.pretty' from '/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/rich/pretty.py'>,
    'inspect': <function inspect at 0x7fa8800b5ef0>,
    'Color': <class 'rich.color.Color'>,
    'color': Color('red', ColorType.STANDARD, number=1),
    'console': <console width=187 ColorSystem.TRUECOLOR>,
    'Theme': <class 'rich.theme.Theme'>,
    'custom_theme': <rich.theme.Theme object at 0x7fa8705dfed0>,
    'bell': None,
    'page': <rich.console.PagerContext object at 0x7fa8703978d0>
}
>>> console.save_text(path="/Users/fanyun/console.log")
```

- `save_html` 同`save_text`，将终端信息保存为html文件

```python
>>> console.print(console, locals())
<console width=187 ColorSystem.TRUECOLOR>
{
    '__name__': '__main__',
    '__doc__': None,
    '__package__': None,
    '__loader__': <class '_frozen_importlib.BuiltinImporter'>,
    '__spec__': None,
    '__annotations__': {},
    '__builtins__': <module 'builtins' (built-in)>,
    'Console': <class 'rich.console.Console'>,
    'rich': <module 'rich' from '/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/rich/__init__.py'>,
    'print': <function print at 0x7fa8800b5c20>,
    'pretty': <module 'rich.pretty' from '/Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/rich/pretty.py'>,
    'inspect': <function inspect at 0x7fa8800b5ef0>,
    'Color': <class 'rich.color.Color'>,
    'color': Color('red', ColorType.STANDARD, number=1),
    'console': <console width=187 ColorSystem.TRUECOLOR>,
    'Theme': <class 'rich.theme.Theme'>,
    'custom_theme': <rich.theme.Theme object at 0x7fa8705dfed0>,
    'bell': None,
    'page': <rich.console.PagerContext object at 0x7fa8703978d0>
}
>>> console.save_html(path="/Users/fanyun/console.html")
```

- `stderr` 

```python
from rich.console import Console
error_console = Console(stderr=True, style="bold red")
```

#### Text()

- `Text`用来标记带有颜色和样式属性的字符串

```python
>>> from rich.text import Text
>>> 
>>> console = Console()
>>> text = Text("hello, world")
>>> text.stylize("bold magenta", 0, 6)
>>> console.print(text)
hello, world
>>> type(Text())
<class 'rich.text.Text'>
>>> type(text)
<class 'rich.text.Text'>
```

```python
>>> text = Text()
>>> text.append("hello", style="bold magenta")
hello
>>> text.append(" world")
hello world
>>> console.print(text)
hello world
```

#### Logging Handler

- `RichHandler` 进行日志记录

```python
import logging
from rich.logging import RichHandler

FORMAT = "%(message)s"
logging.basicConfig(
    level="NOTSET", format=FORMAT, datefmt="[%X]", handlers=[RichHandler()]
)

log = logging.getLogger("rich")
log.info("Hello, World!")
log.error("[bold red blink]Server is shutting down![/]", extra={"markup": True})
```

- `Handler Exception`

```python
logging.basicConfig(
    level="NOTSET", format="%(message)s", datefmt="[%X]", handlers=[RichHandler(rich_tracebacks=True)]
)

log = logging.getLogger("rich")

try:
    print(1 / 0)
except Exception as e:
    log.exception("unable print!")
```

#### Progress

- `Progress()`动态展示当前执行任务的进度条

```python
>>> import time
>>> from rich.progress import Progress
>>> 
>>> 
>>> with Progress() as pg:
...     task1 = pg.add_task("[red]Downloading...", total=1000)
...     task2 = pg.add_task("[green]Processing...", total=1000)
...     while not pg.finished:
...         pg.update(task1, advance=0.5)
...         pg.update(task2, advance=0.3)
...         time.sleep(0.02)
... 
Downloading... ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
Processing...  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
```





##### 参考：[Appendix — Rich 10.4.0 documentation](https://rich.readthedocs.io/en/stable/appendix.html)

