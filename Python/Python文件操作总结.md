# Python文件操作总结

> 本文为译文，原文链接 [working-with-files-in-python](https://link.juejin.im/?target=https%3A%2F%2Frealpython.com%2Fworking-with-files-in-python)

Python中有几个内置模块和方法来处理文件。这些方法被分割到例如`os`, `os.path` , `shutil` 和 `pathlib` 等等几个模块中。文章将列举Python中对文件最常用的操作和方法。

在这篇文章中，你将学习如何：

- 获取文件属性
- 创建目录
- 文件名模式匹配
- 遍历目录树
- 创建临时文件和目录
- 删除文件和目录
- 复制、移动和重命名文件和目录
- 创建和解压ZIP和TAR档案
- 使用`fileinput` 模块打开多个文件

# Python中文件数据的读和写

使用Python对文件进行读和写是十分简单的。为此，你首先必须使用合适的模式打开文件。这里有一个如何打开文本文件并读取其内容的例子。

```
with open('data.txt', 'r') as f:
    data = f.read()
    print('context: {}'.format(data))

```

`open()` 接收一个文件名和一个模式作为它的参数，`r` 表示以只读模式打开文件。想要往文件中写数据的话，则用`w` 作为参数。

```
with open('data.txt', 'w') as f:
    data = 'some data to be written to the file'
    f.write(data)

```

在上述例子中，open()打开用于读取或写入的文件并返回文件句柄(本例子中的 `f` )，该句柄提供了可用于读取或写入文件数据的方法。阅读 [Working With File I/O in Python](https://link.juejin.im/?target=https%3A%2F%2Fdbader.org%2Fblog%2Fpython-file-io) 获取更多关于如何读写文件的信息。


# 获取目录列表

假设你当前的工作目录有一个叫 `my_directory` 的子目录，该目录包含如下内容：

```
.
├── file1.py
├── file2.csv
├── file3.txt
├── sub_dir
│   ├── bar.py
│   └── foo.py
├── sub_dir_b
│   └── file4.txt
└── sub_dir_c
    ├── config.py
    └── file5.txt

```

Python内置的 `os` 模块有很多有用的方法能被用来列出目录内容和过滤结果。为了获取文件系统中特定目录的所有文件和文件夹列表，可以在遗留版本的Python中使用 `os.listdir()` 或 在Python 3.x 中使用 `os.scandir()` 。 如果你还想获取文件和目录属性(如文件大小和修改日期)，那么 `os.scandir()` 则是首选的方法。

## 使用遗留版本的Python获取目录列表

```
import os
entries = os.listdir('my_directory')

```

`os.listdir()` 返回一个Python列表，其中包含path参数所指目录的文件和子目录的名称。

```
['file1.py', 'file2.csv', 'file3.txt', 'sub_dir', 'sub_dir_b', 'sub_dir_c']

```

目录列表现在看上去不容易阅读，对 `os.listdir()` 的调用结果使用循环打印有助于查看。

```
for entry in entries:
    print(entry)

"""
file1.py
file2.csv
file3.txt
sub_dir
sub_dir_b
sub_dir_c
"""


```

## 使用现代版本的Python获取目录列表

在现代Python版本中，可以使用 `os.scandir()` 和 `pathlib.Path` 来替代 `os.listdir()` 。

`os.scandir()` 在Python 3.5 中被引用，其文档为 [PEP 471](https://link.juejin.im/?target=https%3A%2F%2Fwww.python.org%2Fdev%2Fpeps%2Fpep-0471%2F) 。

`os.scandir()` 调用时返回一个迭代器而不是一个列表。

```
import os
entries = os.scandir('my_directory')
print(entries)
# <posix.ScandirIterator at 0x105b4d4b0>
```

ScandirIterator 指向了当前目录中的所有条目。你可以遍历迭代器的内容，并打印文件名。

```
import os
with os.scandir('my_directory') as entries:
    for entry in entries:
        print(entry.name)
```

这里 `os.scandir()` 和with语句一起使用，因为它支持上下文管理协议。使用上下文管理器关闭迭代器并在迭代器耗尽后自动释放获取的资源。在 `my_directory` 打印文件名的结果就和在 `os.listdir()` 例子中看到的一样：

```
file1.py
file2.csv
file3.txt
sub_dir
sub_dir_b
sub_dir_c
```

另一个获取目录列表的方法是使用 `pathlib` 模块：

```
from pathlib import Path

entries = Path('my_directory')
for entry in entries.iterdir():
    print(entry.name)
```

`pathlib.Path()` 返回的是 `PosixPath` 或 `WindowsPath` 对象，这取决于操作系统。

`pathlib.Path()` 对象有一个 `.iterdir()` 的方法用于创建一个迭代器包含该目录下所有文件和目录。由 `.iterdir()` 生成的每个条目都包含文件或目录的信息，例如其名称和文件属性。`pathlib`在Python3.4时被第一次引入，并且是对Python一个很好的加强，它为文件系统提供了面向对象的接口。

在上面的例子中，你调用 `pathlib.Path()` 并传入了一个路径参数。然后调用 `.iterdir()` 来获取 `my_directory` 下的所有文件和目录列表。

`pathlib` 提供了一组类，以简单并且面向对象的方式提供了路径上的大多数常见的操作。使用 `pathlib` 比起使用 `os` 中的函数更加有效。和 `os` 相比，使用 `pathlib` 的另一个好处是减少了操作文件系统路径所导入包或模块的数量。想要了解更多信息，可以阅读 [Python 3’s pathlib Module: Taming the File System](https://link.juejin.im/?target=https%3A%2F%2Frealpython.com%2Fpython-pathlib%2F) 。

运行上述代码会得到如下结果:

```
file1.py
file2.csv
file3.txt
sub_dir
sub_dir_b
sub_dir_c
```

使用 `pathlib.Path()` 或 `os.scandir()` 来替代 `os.listdir()` 是获取目录列表的首选方法，尤其是当你需要获取文件类型和文件属性信息的时候。`pathlib.Path()` 提供了在 `os` 和 `shutil` 中大部分处理文件和路径的功能，并且它的方法比这些模块更加有效。我们将讨论如何快速的获取文件属性。

| 函数                     | 描述                                                     |
| ------------------------ | -------------------------------------------------------- |
| os.listdir()             | 以列表的方式返回目录中所有的文件和文件夹                 |
| os.scandir()             | 返回一个迭代器包含目录中所有的对象，对象包含文件属性信息 |
| pathlib.Path().iterdir() | 返回一个迭代器包含目录中所有的对象，对象包含文件属性信息 |

这些函数返回目录中所有内容的列表，包括子目录。这可能并总是你一直想要的结果，下一节将向你展示如何从目录列表中过滤结果。

## 列出目录中的所有文件

这节将向你展示如何使用 `os.listdir()` ，`os.scandir()` 和 `pathlib.Path()` 打印出目录中文件的名称。为了过滤目录并仅列出 `os.listdir()` 生成的目录列表的文件，要使用 `os.path` ：

```
import os

basepath = 'my_directory'
for entry in os.listdir(basepath):
    # 使用os.path.isfile判断该路径是否是文件类型
    if os.path.isfile(os.path.join(base_path, entry)):
        print(entry)
```

在这里调用 `os.listdir()` 返回指定路径中所有内容的列表，接着使用 `os.path.isfile()` 过滤列表让其只显示文件类型而非目录类型。代码执行结果如下：

```
file1.py
file2.csv
file3.txt
```

一个更简单的方式来列出一个目录中所有的文件是使用 `os.scandir()` 或 `pathlib.Path()` :

```
import os

basepath = 'my_directory'
with os.scandir(basepath) as entries:
    for entry in entries:
        if entry.is_file():
            print(entry.name)
```

使用 `os.scandir()` 比起 `os.listdir()` 看上去更清楚和更容易理解。对 `ScandirIterator` 的每一项调用 `entry.isfile()` ，如果返回 `True` 则表示这一项是一个文件。上述代码的输出如下：

```
file1.py
file3.txt
file2.csv
```

接着，展示如何使用 `pathlib.Path()` 列出一个目录中的文件：

```
from pathlib import Path

basepath = Path('my_directory')
for entry in basepath.iterdir():
    if entry.is_file():
        print(entry.name)
```

在 `.iterdir()` 产生的每一项调用 `.is_file()` 。产生的输出结果和上面相同：

```
file1.py
file3.txt
file2.csv
```

如果将for循环和if语句组合成单个生成器表达式，则上述的代码可以更加简洁。关于生成器表达式，推荐一篇[Dan Bader](https://link.juejin.im/?target=https%3A%2F%2Fdbader.org%2Fblog%2Fpython-generator-expressions) 的文章。

修改后的版本如下：

```
from pathlib import Path

basepath = Path('my_directory')
files_in_basepath = (entry for entry in basepath.iterdir() if entry.is_file())
for item in files_in_basepath:
    print(item.name)
```

上述代码的执行结果和之前相同。本节展示使用 `os.scandir()` 和 `pathlib.Path()` 过滤文件或目录比使用 `os.listdir()` 和 `os.path` 更直观，代码看起来更简洁。

## 列出子目录

如果要列出子目录而不是文件，请使用下面的方法。现在展示如何使用 `os.listdir()` 和 `os.path()` :

```
import os

basepath = 'my_directory'
for entry in os.listdir(basepath):
    if os.path.isdir(os.path.join(basepath, entry)):
        print(entry)
```

当你多次调用 `os.path,join()` 时，以这种方式操作文件系统就会变得很笨重。在我电脑上运行此代码会产生以下输出：

```
sub_dir
sub_dir_b
sub_dir_c
```

下面是如何使用 `os.scandir()` ：

```
import os

basepath = 'my_directory'
with os.scandir(basepath) as entries:
    for entry in entries:
        if entry.is_dir():
            print(entry.name)
```

与文件列表中的示例一样，此处在 `os.scandir()` 返回的每一项上调用 `.is_dir()` 。如果这项是目录，则 `is_dir()` 返回 True，并打印出目录的名称。输出结果和上面相同：

```
sub_dir_c
sub_dir_b
sub_dir
```

下面是如何使用 `pathlib.Path()` ：

```
from pathlib import Path

basepath = Path('my_directory')
for entry in basepath.iterdir():
    if entry.is_dir():
        print(entry.name)
```

在 `.iterdir()` 迭代器返回的每一项上调用 `is_dir()` 检查是文件还是目录。如果该项是目录，则打印其名称，并且生成的输出与上一示例中的输出相同：

```
sub_dir_c
sub_dir_b
sub_dir
```

------

# 获取文件属性

Python可以很轻松的获取文件大小和修改时间等文件属性。可以通过使用 `os.stat()` ， `os.scandir()` 或 `pathlib.Path` 来获取。

`os.scandir()` 和 `pathlib.Path()` 能直接获取到包含文件属性的目录列表。这可能比使用 `os.listdir()` 列出文件然后获取每个文件的文件属性信息更加有效。

下面的例子显示了如何获取 `my_directory` 中文件的最后修改时间。以时间戳的方式输出：

```
import os

with os.scandir('my_directory') as entries:
    for entry in entries:
        info = entry.stat()
        print(info.st_mtime)
        
"""
1548163662.3952665
1548163689.1982062
1548163697.9175904
1548163721.1841028
1548163740.765162
1548163769.4702623
"""
```

`os.scandir()` 返回一个 `ScandirIterator` 对象。`ScandirIterator` 对象中的每一项有 `.stat()`方法能获取关于它指向文件或目录的信息。`.stat()` 提供了例如文件大小和最后修改时间的信息。在上面的示例中，代码打印了 `st_time` 属性，该属性是上次修改文件内容的时间。

`pathlib` 模块具有相应的方法，用于获取相同结果的文件信息:

```
from pathlib import Path

basepath = Path('my_directory')
for entry in basepath.iterdir():
    info = entry.stat()
    print(info.st_mtime)

"""
1548163662.3952665
1548163689.1982062
1548163697.9175904
1548163721.1841028
1548163740.765162
1548163769.4702623
"""
```

在上面的例子中，循环 `.iterdir()` 返回的迭代器并通过对其中每一项调用 `.stat()` 来获取文件属性。`st_mtime` 属性是一个浮点类型的值，表示的是时间戳。为了让 `st_time` 返回的值更容易阅读，你可以编写一个辅助函数将其转换为一个 `datetime` 对象：

```
import datetime                                                                   
from pathlib import Path                                                          
                                                                                  
                                                                                  
def timestamp2datetime(timestamp, convert_to_local=True, utc=8, is_remove_ms=True)
    """                                                                           
    转换 UNIX 时间戳为 datetime对象                                                       
    :param timestamp: 时间戳                                                         
    :param convert_to_local: 是否转为本地时间                                             
    :param utc: 时区信息，中国为utc+8                                                     
    :param is_remove_ms: 是否去除毫秒                                                   
    :return: datetime 对象                                                          
    """                                                                           
    if is_remove_ms:                                                              
        timestamp = int(timestamp)                                                
    dt = datetime.datetime.utcfromtimestamp(timestamp)                            
    if convert_to_local:                                                          
        dt = dt + datetime.timedelta(hours=utc)                                   
    return dt                                                                     
                                                                                  
                                                                                  
def convert_date(timestamp, format='%Y-%m-%d %H:%M:%S'):                          
    dt = timestamp2datetime(timestamp)                                            
    return dt.strftime(format)                                                    
                                                                                  
                                                                                  
basepath = Path('my_directory')                                                   
for entry in basepath.iterdir():
    if entry.is_file()
    	info = entry.stat()                                                           
    	print('{} 上次修改时间为 {}'.format(entry.name, timestamp2datetime(info.st_mtime)))  
```

首先得到 `my_directory` 中文件的列表以及它们的属性，然后调用 `convert_date()` 来转换文件最后修改时间让其以一种人类可读的方式显示。`convert_date()` 使用 `.strftime()` 将datetime类型转换为字符串。

上述代码的输出结果：

```
file3.txt 上次修改时间为 2019-01-24 09:04:39
file2.csv 上次修改时间为 2019-01-24 09:04:39
file1.py 上次修改时间为 2019-01-24 09:04:39
```

将日期和时间转换为字符串的语法可能会让你感到混乱。如果要了解更多的信息，请查询相关的[官方文档](https://link.juejin.im/?target=https%3A%2F%2Fdocs.python.org%2F3%2Flibrary%2Fdatetime.html%23strftime-and-strptime-behavior) 。另一个方式则是阅读 [strftime.org](https://link.juejin.im/?target=http%3A%2F%2Fstrftime.org) 。

# 创建目录

你编写的程序迟早需要创建目录以便在其中存储数据。 `os` 和 `pathlib` 包含了创建目录的函数。我们将会考虑如下方法：

| 方法                 | 描述                       |
| -------------------- | -------------------------- |
| os.mkdir()           | 创建单个子目录             |
| os.makedirs()        | 创建多个目录，包括中间目录 |
| Pathlib.Path.mkdir() | 创建单个或多个目录         |

## 创建单个目录

要创建单个目录，把目录路径作为参数传给 `os.mkdir()` :

```
import os

os.mkdir('example_directory')
```

如果该目录已经存在，`os.mkdir()` 将抛出 `FileExistsError` 异常。或者，你也可以使用 `pathlib` 来创建目录:

```
from pathlib import Path

p = Path('example_directory')
p.mkdir()
```

如果路径已经存在，`mkdir()` 会抛出 `FileExistsError` 异常:

```
FileExistsError: [Errno 17] File exists: 'example_directory'
```

为了避免像这样的错误抛出， 当发生错误时捕获错误并让你的用户知道:

```
from pathlib import Path

p = Path('example_directory')
try:
    p.mkdir()
except FileExistsError as e:
    print(e)
```

或者，你可以给 `.mkdir()` 传入 `exist_ok=True` 参数来忽略 `FileExistsError` 异常:

```
from pathlib import Path

p = Path('example_directory')
p.mkdir(exist_ok=True)
```

如果目录已存在，则不会引起错误。

## 创建多个目录

`os.makedirs()` 和 `os.mkdir()` 类似。两者之间的区别在于，`os.makedirs()` 不仅可以创建单独的目录，还可以递归的创建目录树。换句话说，它可以创建任何必要的中间文件夹，来确保存在完整的路径。

`os.makedirs()` 和在bash中运行 `mkdir -p` 类似。例如，要创建一组目录像 2018/10/05，你可以像下面那样操作:

```
import os

os.makedirs('2018/10/05', mode=0o770)
```

上述代码创建了 `2018/10/05` 的目录结构并为所有者和组用户提供读、写和执行权限。默认的模式为 `0o777` ，增加了其他用户组的权限。有关文件权限以及模式的应用方式的更多详细信息，请参考 [文档](https://link.juejin.im/?target=https%3A%2F%2Fdocs.python.org%2F3%2Flibrary%2Fos.html%23os.makedirs) 。

运行 `tree` 命令确认我们应用的权限:

```
$ tree -p -i .
.
[drwxrwx---]  2018
[drwxrwx---]  10
[drwxrwx---]  05
```

上述代码打印出当前目录的目录树。 `tree` 通常被用来以树形结构列出目录的内容。传入 `-p` 和 `-i` 参数则会以垂直列表打印出目录名称以及其文件权限信息。`-p` 用于输出文件权限，`-i` 则用于让 `tree` 命令产生一个没有缩进线的垂直列表。

正如你所看到的，所有的目录都拥有 770 权限。另一个方式创建多个目录是使用 `pathlib.Path` 的 `.mkdir()` :

```
from pathlib import Path

p = Path('2018/10/05')
p.mkdir(parents=True, exist_ok=True)
```

通过给 `Path.mkdir()` 传递 `parents=True` 关键字参数使它创建 `05` 目录和使其路径有效的所有父级目录。

在默认情况下，`os.makedirs()` 和 `pathlib.Path.mkdir()` 会在目标目录存在的时候抛出 `OSError`。通过每次调用函数时传递 `exist_ok=True` 作为关键字参数则可以覆盖此行为（从Python3.2开始）。

运行上述代码会得到像下面的结构：

```
└── 2018
    └── 10
        └── 05
```

我更喜欢在创建目录时使用 `pathlib` ，因为我可以使用相同的函数方法来创建一个或多个目录。

# 文件名模式匹配

使用上述方法之一获取目录中的文件列表后，你可能希望搜索和特定的模式匹配的文件。

下面这些是你可以使用的方法和函数：

- `endswith()` 和 `startswith()` 字符串方法
- `fnmatch.fnmatch()`
- `glob.glob()`
- `pathlib.Path.glob()`

这些方法和函数是下面要讨论的。本小节的示例将在名为 `some_directory` 的目录下执行，该目录具有以下的结构：

```
.
├── admin.py
├── data_01_backup.txt
├── data_01.txt
├── data_02_backup.txt
├── data_02.txt
├── data_03_backup.txt
├── data_03.txt
├── sub_dir
│   ├── file1.py
│   └── file2.py
└── tests.py
```

如果你正在使用 Bash shell，你可以使用以下的命令创建上述目录结构:

```
mkdir some_directory
cd some_directory
mkdir sub_dir
touch sub_dir/file1.py sub_dir/file2.py
touch data_{01..03}.txt data_{01..03}_backup.txt admin.py tests.py
```

这将会创建 `some_directory` 目录并进入它，接着创建 `sub_dir` 。下一行在 `sub_dir` 创建 `file1.py` 和 `file2.py` ，最后一行使用扩展创建其它所有文件。想要学习更多关于shell扩展，请阅读 [这里](https://link.juejin.im/?target=http%3A%2F%2Flinuxcommand.org%2Flc3_lts0080.php) 。

## 使用字符串方法

Python有几个内置 [修改和操作字符串](https://link.juejin.im/?target=https%3A%2F%2Frealpython.com%2Fpython-strings%2F) 的方法。当在匹配文件名时，其中的两个方法 `.startswith()` 和 `.endswith()` 非常有用。要做到这点，首先要获取一个目录列表，然后遍历。

```
import os

for f_name in os.listdir('some_directory'):
    if f_name.endswith('.txt'):
        print(f_name)
```

上述代码找到 `some_directory` 中的所有文件，遍历并使用 `.endswith()` 来打印所有扩展名为 `.txt` 的文件名。运行代码在我的电脑上输出如下:

```
data_01.txt
data_01_backup.txt
data_02.txt
data_02_backup.txt
data_03.txt
data_03_backup.txt
```

## 使用 `fnmatch` 进行简单文件名模式匹配

字符串方法匹配的能力是有限的。`fnmatch` 有对于模式匹配有更先进的函数和方法。我们将考虑使用 `fnmatch.fnmatch()` ，这是一个支持使用 `*` 和 `?` 等通配符的函数。例如，使用 `fnmatch` 查找目录中所有 `.txt` 文件，你可以这样做:

```
import os
import fnmatch

for f_name in os.listdir('some_directory'):
    if fnmatch.fnmatch(f_name, '*.txt'):
        print(f_name)
```

迭代 `some_directory` 中的文件列表，并使用 `.fnmatch()` 对扩展名为 `.txt` 的文件执行通配符搜索。

## 更先进的模式匹配

假设你想要查找符合特定掉件的 `.txt` 文件。例如，你可能指向找到包含单次 `data` 的 `.txt`文件，一组下划线之间的数字，以及文件名中包含单词 `backup` 。就类似于 `data_01_backup`, `data_02_backup`, 或 `data_03_backup` 。

你可以这样使用 `fnmatch.fnmatch()` :

```
import os
import fnmatch

for f_name in os.listdir('some_directory'):
    if fnmatch.fnmatch(f_name, 'data_*_backup.txt'):
        print(f_name)
```

这里就仅仅打印出匹配 `data_*_backup.txt` 模式的文件名称。模式中的 `*` 将匹配任何字符，因此运行这段代码则将查找文件名以 `data` 开头并以 `backup.txt` 的所有文本文件，就行下面的输出所示 :

```
data_01_backup.txt
data_02_backup.txt
data_03_backup.txt

```

## 使用 `glob` 进行文件名模式匹配

另一个有用的模式匹配模块是 `glob` 。

`.glob()` 在 `glob` 模块中的左右就像 `fnmatch.fnmatch()`，但是与 `fnmach.fnmatch()` 不同的是，它将以 `.` 开头的文件视为特殊文件。

UNIX和相关系统在文件列表中使用通配符像 `?` 和 `*` 表示全匹配。

例如，在UNIX shell中使用 `mv *.py python_files` 移动所有 `.py` 扩展名 的文件从当前目录到 `python_files` 。这 `*` 是一个通配符表示任意数量的字符，`*.py` 是一个全模式。Windows操作系统中不提供此shell功能。但 `glob` 模块在Python中添加了此功能，使得Windows程序可以使用这个特性。

这里有一个使用 `glob` 模块在当前目录下查询所有Python代码文件:

```
import glob

print(glob.glob('*.py'))

```

`glob.glob('*.py')` 搜索当前目录中具有 `.py` 扩展名的文件，并且将它们以列表的形式返回。 `glob` 还支持 shell 样式的通配符来进行匹配 :

```
import glob

for name in glob.glob('*[0-9]*.txt'):
    print(name)

```

这将找到所有文件名中包含数字的文本文件(`.txt`) :

```
data_01.txt
data_01_backup.txt
data_02.txt
data_02_backup.txt
data_03.txt
data_03_backup.txt

```

`glob` 也很容易在子目录中递归的搜索文件:

```
import glob

for name in glob.iglob('**/*.py', recursive=True):
    print(name)

```

这里例子使用了 `glob.iglob()` 在当前目录和子目录中搜索所有的 `.py` 文件。传递 `recursive=True` 作为 `.iglob()` 的参数使其搜索当前目录和子目录中的 `.py` 文件。`glob.glob()` 和 `glob.iglob()` 不同之处在于，`iglob()` 返回一个迭代器而不是一个列表。

运行上述代码会得到以下结果:

```
admin.py
tests.py
sub_dir/file1.py
sub_dir/file2.py

```

`pathlib` 也包含类似的方法来灵活的获取文件列表。下面的例子展示了你可以使用 `.Path.glob()`列出以字母 `p` 开始的文件类型的文件列表。

```
from pathlib import Path

p = Path('.')

for name in p.glob('*.p*'):
    print(name)

```

调用 `p.glob('*.p*')` 会返回一个指向当前目录中所有扩展名以字母 `p` 开头的文件的生成器对象。

`Path.glob()` 和上面讨论过的 `os.glob()` 类似。正如你看到的， `pathlib` 混合了许多 `os` ， `os.path` 和 `glob` 模块的最佳特性到一个模块中，这使得使用起来很方便。

回顾一下，这是我们在本节中介绍的功能表:

| 函数                               | 描述                                                       |
| ---------------------------------- | ---------------------------------------------------------- |
| startswith()                       | 测试一个字符串是否以一个特定的模式开始，返回 True 或 False |
| endswith()                         | 测试一个字符串是否以一个特定的模式结束，返回 True 或 False |
| fnmatch.fnmatch(filename, pattern) | 测试文件名是否匹配这个模式，返回 True 或 False             |
| glob.glob()                        | 返回一个匹配该模式的文件名列表                             |
| pathlib.Path.glob()                | 返回一个匹配该模式的生成器对象                             |

------

# 遍历目录和处理文件

一个常见的编程任务是遍历目录树并处理目录树中的文件。让我们来探讨一下如何使用内置的Python函数 `os.walk()` 来实现这一功能。`os.walk()` 用于通过从上到下或从下到上遍历树来生成目录树中的文件名。处于本节的目的，我们想操作以下的目录树:

```
├── folder_1
│   ├── file1.py
│   ├── file2.py
│   └── file3.py
├── folder_2
│   ├── file4.py
│   ├── file5.py
│   └── file6.py
├── test1.txt
└── test2.txt

```

以下是一个示例，演示如何使用 `os.walk()` 列出目录树中的所有文件和目录。

`os.walk()` 默认是从上到下遍历目录:

```
import os
for dirpath, dirname, files in os.walk('.'):
   print(f'Found directory: {dirpath}')
   for file_name in files:
       print(file_name)

```

`os.walk()` 在每个循环中返回三个值：

1. 当前文件夹的名称
2. 当前文件夹中子文件夹的列表
3. 当前文件夹中文件的列表

在每次迭代中，会打印出它找到的子目录和文件的名称：

```
Found directory: .
test1.txt
test2.txt
Found directory: ./folder_1
file1.py
file3.py
file2.py
Found directory: ./folder_2
file4.py
file5.py
file6.py

```

要以自下而上的方式遍历目录树，则将 `topdown=False` 关键字参数传递给 `os.walk()` ：

```
for dirpath, dirnames, files in os.walk('.', topdown=False):
    print(f'Found directory: {dirpath}')
    for file_name in files:
        print(file_name)

```

传递 `topdown=False` 参数将使 `os.walk()` 首先打印出它在子目录中找到的文件:

```
Found directory: ./folder_1
file1.py
file3.py
file2.py
Found directory: ./folder_2
file4.py
file5.py
file6.py
Found directory: .
test1.txt
test2.txt

```

如你看见的，程序在列出根目录的内容之前列出子目录的内容。 这在在你想要递归删除文件和目录的情况下非常有用。 你将在以下部分中学习如何执行此操作。 默认情况下，`os.walk` 不会访问通过软连接创建的目录。 可以通过使用 `followlinks = True` 参数来覆盖默认行为。

------

# 创建临时文件和目录

Python提供了 `tempfile` 模块来便捷的创建临时文件和目录。

`tempfile` 可以在你程序运行时打开并存储临时的数据在文件或目录中。 `tempfile` 会在你程序停止运行后删除这些临时文件。

现在，让我们看看如何创建一个临时文件:

```
from tempfile import  TemporaryFile

# 创建一个临时文件并为其写入一些数据
fp = TemporaryFile('w+t')
fp.write('Hello World!')
# 回到开始，从文件中读取数据
fp.seek(0)
data = fp.read()
print(data)
# 关闭文件，之后他将会被删除
fp.close()

```

第一步是从 `tempfile` 模块导入 `TemporaryFile` 。 接下来，使用 `TemporaryFile()` 方法并传入一个你想打开这个文件的模式来创建一个类似于对象的文件。这将创建并打开一个可用作临时存储区域的文件。

在上面的示例中，模式为 `w + t`，这使得 `tempfile` 在写入模式下创建临时文本文件。 没有必要为临时文件提供文件名，因为在脚本运行完毕后它将被销毁。

写入文件后，您可以从中读取并在完成处理后将其关闭。 一旦文件关闭后，将从文件系统中删除。 如果需要命名使用 `tempfile` 生成的临时文件，请使用 `tempfile.NamedTemporaryFile()` 。

使用 `tempfile` 创建的临时文件和目录存储在用于存储临时文件的特殊系统目录中。 Python将在目录列表搜索用户可以在其中创建文件的目录。

在Windows上，目录按顺序为 `C:\TEMP`，`C:\TMP`，`\TEMP` 和 `\TMP`。 在所有其他平台上，目录按顺序为 `/ tmp`，`/var/tmp` 和 `/usr/tmp` 。 如果上述目录中都没有，`tempfile` 将在当前目录中存储临时文件和目录。

`.TemporaryFile()` 也是一个上下文管理器，因此它可以与with语句一起使用。 使用上下文管理器会在读取文件后自动关闭和删除文件：

```
with TemporaryFile('w+t') as fp:
    fp.write('Hello universe!')
    fp.seek(0)
    fp.read()
# 临时文件现在已经被关闭和删除

```

这将创建一个临时文件并从中读取数据。 一旦读取文件的内容，就会关闭临时文件并从文件系统中删除。

`tempfile` 也可用于创建临时目录。 让我们看一下如何使用 `tempfile.TemporaryDirectory()`来做到这一点：

```
import tempfile
import os

tmp = ''
with tempfile.TemporaryDirectory() as tmpdir:
    print('Created temporary directory ', tmpdir)
    tmp = tmpdir
    print(os.path.exists(tmpdir))

print(tmp)
print(os.path.exists(tmp))

```

调用 `tempfile.TemporaryDirectory()` 会在文件系统中创建一个临时目录，并返回一个表示该目录的对象。 在上面的示例中，使用上下文管理器创建目录，目录的名称存储在 `tmpdir` 变量中。 第三行打印出临时目录的名称，`os.path.exists(tmpdir)` 来确认目录是否实际在文件系统中创建。

在上下文管理器退出上下文后，临时目录将被删除，并且对 `os.path.exists(tmpdir)`的调用将返回False，这意味着该目录已成功删除。

------

# 删除文件和目录

您可以使用 `os`，`shutil` 和 `pathlib` 模块中的方法删除单个文件，目录和整个目录树。 以下将介绍如何删除你不再需要的文件和目录。

## Python中删除文件

要删除单个文件，请使用 `pathlib.Path.unlink()`，`os.remove()` 或 `os.unlink()`。

`os.remove()` 和 `os.unlink()` 在语义上是相同的。 要使用 `os.remove()`删除文件，请执行以下操作：

```
import os

data_file = 'C:\\Users\\vuyisile\\Desktop\\Test\\data.txt'
os.remove(data_file)

```

使用 `os.unlink()` 删除文件与使用 `os.remove()` 的方式类似：

```
import os

data_file = 'C:\\Users\\vuyisile\\Desktop\\Test\\data.txt'
os.unlink(data_file)

```

在文件上调用 `.unlink()` 或 `.remove()` 会从文件系统中删除该文件。 如果传递给它们的路径指向目录而不是文件，这两个函数将抛出 `OSError` 。 为避免这种情况，可以检查你要删除的内容是否是文件，并在确认是文件时执行删除操作，或者可以使用异常处理来处理 `OSError` ：

```
import os

data_file = 'home/data.txt'
# 如果类型是文件则进行删除
if os.path.is_file(data_file):
    os.remove(data_file)
else:
    print(f'Error: {data_file} not a valid filename')

```

`os.path.is_file()` 检查 `data_file` 是否实际上是一个文件。 如果是，则通过调用 `os.remove()`删除它。 如果 `data_file` 指向文件夹，则会向控制台输出错误消息。

以下示例说明如何在删除文件时使用异常处理来处理错误：

```
import os

data_file = 'home/data.txt'
# 使用异常处理
try:
    os.remove(data_file)
except OSError as e:
    print(f'Error: {data_file} : {e.strerror}')

```

上面的代码尝试在检查其类型之前先删除该文件。 如果 `data_file` 实际上不是文件，则抛出的 `OSError` 将在except子句中处理，并向控制台输出错误消息。 打印出的错误消息使用 [Python f-strings ](https://link.juejin.im/?target=https%3A%2F%2Frealpython.com%2Fpython-f-strings%2F)格式化。

最后，你还可以使用 `pathlib.Path.unlink()` 删除文件：

```
from pathlib import Path

data_file = Path('home/data.txt')
try:
    data_file.unlink()
except IsADirectoryError as e:
    print(f'Error: {data_file} : {e.strerror}')

```

这将创建一个名为 `data_file` 的 `Path` 对象，该对象指向一个文件。 在 `data_file` 上调用.unlink（）将删除 `home / data.txt` 。 如果 `data_file` 指向目录，则引发 `IsADirectoryError` 。 值得注意的是，上面的Python程序和运行它的用户具有相同的权限。 如果用户没有删除文件的权限，则会引发 `PermissionError` 。

## 删除目录

标准库提供了一下函数来删除目录:

- os.rmdir()
- pathlib.Path.rmdir()
- shutil.rmtree()

要删除单个目录或文件夹可以使用 `os.rmdir()` 或 `pathlib.Path.rmdir()` 。这两个函数只在你删除空目录的时候有效。如果目录不为空，则会抛出 `OSError` 。下面演示如何删除一个文件夹:

```
import os

trash_dir = 'my_documents/bad_dir'

try:
    os.rmdir(trash_dir)
except OSError as e:
    print(f'Error: {trash_dir} : {e.strerror}')

```

现在，`trash_dir` 已经通过 `os.rmdir()` 被删除了。如果目录不为空，则会在屏幕上打印错误信息:

```
Traceback (most recent call last):
  File '<stdin>', line 1, in <module>
OSError: [Errno 39] Directory not empty: 'my_documents/bad_dir'

```

同样，你也可使用 `pathlib` 来删除目录:

```
from pathlib import Path

trash_dir = Path('my_documents/bad_dir')

try:
    trash_dir.rmdir()
except OSError as e:
    print(f'Error: {trash_dir} : {e.strerror}')

```

这里创建了一个 `Path` 对象指向要被删除的目录。如果目录为空，调用 `Path` 对象的 `.rmdir()`方法删除它。

## 删除完整的目录树

要删除非空目录和完整的目录树，Python提供了 `shutil.rmtree()` :

```
import shutil

trash_dir = 'my_documents/bad_dir'

try:
    shutil.rmtree(trash_dir)
except OSError as e:
    print(f'Error: {trash_dir} : {e.strerror}')

```

当调用 `shutil.rmtree()` 时，`trash_dir` 中的所有内容都将被删除。 在某些情况下，你可能希望以递归方式删除空文件夹。 你可以使用上面讨论的方法之一结合 `os.walk()` 来完成此操作:

```
import os

for dirpath, dirnames, files in os.walk('.', topdown=False):
    try:
        os.rmdir(dirpath)
    except OSError as ex:
        pass

```

这将遍历目录树并尝试删除它找到的每个目录。 如果目录不为空，则引发OSError并跳过该目录。 下表列出了本节中涉及的功能：

| 函数                  | 描述                                 |
| --------------------- | ------------------------------------ |
| os.remove()           | 删除单个文件，不能删除目录           |
| os.unlink()           | 和os.remove()一样，职能删除单个文件  |
| pathlib.Path.unlink() | 删除单个文件，不能删除目录           |
| os.rmdir()            | 删除一个空目录                       |
| pathlib.Path.rmdir()  | 删除一个空目录                       |
| shutil.rmtree()       | 删除完整的目录树，可用于删除非空目录 |

------

# 复制、移动和重命名文件和目录

Python附带了 `shutil` 模块。 `shutil` 是shell实用程序的缩写。 它为文件提供了许多高级操作，来支持文件和目录的复制，归档和删除。 在本节中，你将学习如何移动和复制文件和目录。

## 复制文件

`shutil` 提供了一些复制文件的函数。 最常用的函数是 `shutil.copy()` 和 `shutil.copy2()` 。 使用`shutil.copy()` 将文件从一个位置复制到另一个位置，请执行以下操作：

```
import shutil

src = 'path/to/file.txt'
dst = 'path/to/dest_dir'
shutil.copy(src, dst)

```

`shutil.copy()` 与基于UNIX的系统中的 `cp` 命令相当。 `shutil.copy(src，dst)` 会将文件 `src`复制到 `dst` 中指定的位置。 如果 `dst` 是文件，则该文件的内容将替换为 `src` 的内容。 如果 `dst` 是目录，则 `src` 将被复制到该目录中。 `shutil.copy()` 仅复制文件的内容和文件的权限。 其他元数据（如文件的创建和修改时间）不会保留。

要在复制时保留所有文件元数据，请使用 `shutil.copy2()` ：

```
import shutil

src = 'path/to/file.txt'
dst = 'path/to/dest_dir'
shutil.copy2(src, dst)

```

使用 `.copy2()` 保留有关文件的详细信息，例如上次访问时间，权限位，上次修改时间和标志。

## 复制目录

虽然 `shutil.copy()` 只复制单个文件，但 `shutil.copytree()` 将复制整个目录及其中包含的所有内容。 `shutil.copytree(src，dest)` 接收两个参数：源目录和将文件和文件夹复制到的目标目录。

以下是如何将一个文件夹的内容复制到其他位置的示例：

```
import shutil
dst = shutil.copytree('data_1', 'data1_backup')
print(dst)  # data1_backup

```

在此示例中，`.copytree()` 将 `data_1` 的内容复制到新位置 `data1_backup` 并返回目标目录。 目标目录不能是已存在的。 它将被创建而不带有其父目录。 `shutil.copytree()` 是备份文件的一个好方法。

## 移动文件和目录

要将文件或目录移动到其他位置，请使用 `shutil.move(src，dst)` 。

`src` 是要移动的文件或目录，`dst` 是目标：

```
import shutil
dst = shutil.move('dir_1/', 'backup/')
print(dst)  # 'backup'

```

如果 `backup/` 存在，则 `shutil.move('dir_1/'，'backup/')` 将 `dir_1/` 移动到 `backup/` 。 如果 `backup/` 不存在，则 `dir_1/` 将重命名为 `backup` 。

## 重命名文件和目录

Python包含用于重命名文件和目录的 `os.rename(src，dst)`：

```
import os
os.rename('first.zip', 'first_01.zip')

```

上面的行将 `first.zip` 重命名为 `first_01.zip` 。 如果目标路径指向目录，则会抛出 `OSError`。

重命名文件或目录的另一种方法是使用 `pathlib` 模块中的 `rename（）`：

```
from pathlib import Path
data_file = Path('data_01.txt')
data_file.rename('data.txt')

```

要使用 `pathlib` 重命名文件，首先要创建一个 `pathlib.Path()` 对象，该对象包含要替换的文件的路径。 下一步是在路径对象上调用 `rename()` 并传入你要重命名的文件或目录的新名称。

------

# 归档

归档是将多个文件打包成一个文件的便捷方式。 两种最常见的存档类型是ZIP和TAR。 你编写的Python程序可以创建存档文件，读取存档文件和从存档文件中提取数据。 你将在本节中学习如何读取和写入两种压缩格式。

## 读取ZIP文件

`zipfile` 模块是一个底层模块，是Python标准库的一部分。 `zipfile` 具有可以轻松打开和提取ZIP文件的函数。 要读取ZIP文件的内容，首先要做的是创建一个 `ZipFile` 对象。`ZipFile` 对象类似于使用 `open()` 创建的文件对象。`ZipFile` 也是一个上下文管理器，因此支持with语句：

```
import zipfile

with zipfile.ZipFile('data.zip', 'r') as zipobj:
    pass

```

这里创建一个 `ZipFile` 对象，传入ZIP文件的名称并以读取模式下打开。 打开ZIP文件后，可以通过 `zipfile` 模块提供的函数访问有关存档文件的信息。 上面示例中的 `data.zip` 存档是从名为 `data` 的目录创建的，该目录包含总共5个文件和1个子目录：

```
.
|
├── sub_dir/
|   ├── bar.py
|   └── foo.py
|
├── file1.py
├── file2.py
└── file3.py

```

要获取存档文件中的文件列表，请在 `ZipFile` 对象上调用 `namelist()` ：

```
import zipfile

with zipfile.ZipFile('data.zip', 'r') as zipobj:
    zipobj.namelist()

```

这会生成一个文件列表:

```
['file1.py', 'file2.py', 'file3.py', 'sub_dir/', 'sub_dir/bar.py', 'sub_dir/foo.py']

```

`.namelist()` 返回存档文件中文件和目录的名称列表。要检索有关存档文件中文件的信息，使用 `.getinfo()` ：

```
import zipfile

with zipfile.ZipFile('data.zip', 'r') as zipobj:
    bar_info = zipobj.getinfo('sub_dir/bar.py')
    print(bar_info.file_size)

```

这将输出:

```
15277

```

`.getinfo()` 返回一个 `ZipInfo` 对象，该对象存储有关存档文件的单个成员的信息。 要获取有关存档文件中文件的信息，请将其路径作为参数传递给 `.getinfo()` 。 使用 `getinfo()` ，你可以检索有关存档文件成员的信息，例如上次修改文件的日期，压缩大小及其完整文件名。 访问 `.file_size` 将以字节为单位检索文件的原始大小。

以下示例说明如何在Python REPL中检索有关已归档文件的更多详细信息。 假设已导入 `zipfile`模块，`bar_info` 与在前面的示例中创建的对象相同：

```
>>> bar_info.date_time
(2018, 10, 7, 23, 30, 10)
>>> bar_info.compress_size
2856
>>> bar_info.filename
'sub_dir/bar.py'

```

`bar_info` 包含有关 `bar.py` 的详细信息，例如压缩的大小及其完整路径。

第一行显示了如何检索文件的上次修改日期。 下一行显示了如何在归档后获取文件的大小。 最后一行显示了存档文件中 `bar.py` 的完整路径。

`ZipFile` 支持上下文管理器协议，这就是你可以将它与with语句一起使用的原因。 操作完成后会自动关闭 `ZipFile` 对象。 尝试从已关闭的 `ZipFile` 对象中打开或提取文件将导致错误。

## 提取ZIP文件

`zipfile` 模块允许你通过 `.extract()` 和 `.extractall()` 从ZIP文件中提取一个或多个文件。

默认情况下，这些方法将文件提取到当前目录。 它们都采用可选的路径参数，允许指定要将文件提取到的其他指定目录。 如果该目录不存在，则会自动创建该目录。 要从压缩文件中提取文件，请执行以下操作：

```
>>> import zipfile
>>> import os

>>> os.listdir('.')
['data.zip']

>>> data_zip = zipfile.ZipFile('data.zip', 'r')

>>> # 提取单个文件到当前目录
>>> data_zip.extract('file1.py')
'/home/test/dir1/zip_extract/file1.py'

>>> os.listdir('.')
['file1.py', 'data.zip']

>>> # 提所有文件到指定目录
>>> data_zip.extractall(path='extract_dir/')

>>> os.listdir('.')
['file1.py', 'extract_dir', 'data.zip']

>>> os.listdir('extract_dir')
['file1.py', 'file3.py', 'file2.py', 'sub_dir']

>>> data_zip.close()

```

第三行代码是对 `os.listdir()` 的调用，它显示当前目录只有一个文件 `data.zip` 。

接下来，以读取模式下打开 `data.zip` 并调用 `.extract()` 从中提取 `file1.py` 。 `.extract()` 返回提取文件的完整文件路径。 由于没有指定路径，`.extract()` 会将 `file1.py` 提取到当前目录。

下一行打印一个目录列表，显示当前目录现在包括除原始存档文件之外的存档文件。 之后显示了如何将整个存档提取到指定目录中。`.extractall()` 创建 `extract_dir` 并将 `data.zip` 的内容提取到其中。 最后一行关闭ZIP存档文件。

## 从加密的文档提取数据

`zipfile` 支持提取受密码保护的ZIP。 要提取受密码保护的ZIP文件，请将密码作为参数传递给 `.extract()` 或`.extractall()` 方法：

```
>>> import zipfile

>>> with zipfile.ZipFile('secret.zip', 'r') as pwd_zip:
...     # 从加密的文档提取数据
...     pwd_zip.extractall(path='extract_dir', pwd='Quish3@o')

```

将以读取模式打开 `secret.zip` 存档。 密码提供给 `.extractall()` ，并且压缩文件内容被提取到 `extract_dir` 。 由于with语句，在完成提取后，存档文件会自动关闭。

## 创建新的存档文件

要创建新的ZIP存档，请以写入模式（w）打开 `ZipFile` 对象并添加要归档的文件：

```
>>> import zipfile

>>> file_list = ['file1.py', 'sub_dir/', 'sub_dir/bar.py', 'sub_dir/foo.py']
>>> with zipfile.ZipFile('new.zip', 'w') as new_zip:
...     for name in file_list:
...         new_zip.write(name)

```

在该示例中，`new_zip` 以写入模式打开，`file_list` 中的每个文件都添加到存档文件中。 with语句结束后，将关闭 `new_zip` 。 以写入模式打开ZIP文件会删除压缩文件的内容并创建新存档文件。

要将文件添加到现有的存档文件，请以追加模式打开 `ZipFile` 对象，然后添加文件：

```
>>> with zipfile.ZipFile('new.zip', 'a') as new_zip:
...     new_zip.write('data.txt')
...     new_zip.write('latin.txt')

```

这里打开在上一个示例中以追加模式创建的 `new.zip` 存档。 在追加模式下打开 `ZipFile` 对象允许将新文件添加到ZIP文件而不删除其当前内容。 将文件添加到ZIP文件后，with语句将脱离上下文并关闭ZIP文件。

## 打开TAR存档文件

TAR文件是像ZIP等未压缩的文件存档。 它们可以使用 `gzip`，`bzip2` 和 `lzma` 压缩方法进行压缩。 `TarFile` 类允许读取和写入TAR存档。

下面是从存档中读取：

```
import tarfile

with tarfile.open('example.tar', 'r') as tar_file:
    print(tar_file.getnames())

```

`tarfile` 对象像大多数类似文件的对象一样打开。 它们有一个 `open()` 函数，它采用一种模式来确定文件的打开方式。

使用“r”，“w”或“a”模式分别打开未压缩的TAR文件以进行读取，写入和追加。 要打开压缩的TAR文件，请将模式参数传递给 `tarfile.open()`，其格式为 `filemode [:compression]` 。 下表列出了可以打开TAR文件的可能模式：

| 模式  | 行为                          |
| ----- | ----------------------------- |
| r     | 以无压缩的读取模式打开存档    |
| r:gz  | 以gzip压缩的读取模式打开存档  |
| r:bz2 | 以bzip2压缩的读取模式打开存档 |
| w     | 以无压缩的写入模式打开存档    |
| w:gz  | 以gzip压缩的写入模式打开存档  |
| w:xz  | 以lzma压缩的写入模式打开存档  |
| a     | 以无压缩的追加模式打开存档    |

`.open()` 默认为'r'模式。 要读取未压缩的TAR文件并检索其中的文件名，请使用 `.getnames()` ：

```
>>> import tarfile

>>> tar = tarfile.open('example.tar', mode='r')
>>> tar.getnames()
['CONTRIBUTING.rst', 'README.md', 'app.py']

```

这以列表的方式返回存档中内容的名字。

> 注意：为了向你展示如何使用不同的tarfile对象方法，示例中的TAR文件在交互式REPL会话中手动打开和关闭。
>
> 通过这种方式与TAR文件交互，你可以查看运行每个命令的输出。 通常，你可能希望使用上下文管理器来打开类似文件的对象。

此外可以使用特殊属性访问存档中每个条目的元数据：

```
>>> for entry in tar.getmembers():
...     print(entry.name)
...     print(' Modified:', time.ctime(entry.mtime))
...     print(' Size    :', entry.size, 'bytes')
...     print()
CONTRIBUTING.rst
 Modified: Sat Nov  1 09:09:51 2018
 Size    : 402 bytes

README.md
 Modified: Sat Nov  3 07:29:40 2018
 Size    : 5426 bytes

app.py
 Modified: Sat Nov  3 07:29:13 2018
 Size    : 6218 bytes

```

在此示例中，循环遍历 `.getmembers()` 返回的文件列表，并打印出每个文件的属性。`.getmembers()` 返回的对象具有可以通过编程方式访问的属性，例如归档中每个文件的名称，大小和上次修改时间。 在读取或写入存档后，必须关闭它以释放系统资源。

## 从TAR存档中提取文件

在本节中，你将学习如何使用以下方法从TAR存档中提取文件：

- `.extract()`
- `.extractfile()`
- `.extractall()`

要从TAR存档中提取单个文件，请使用 `extract()` ，传入文件名：

```
>>> tar.extract('README.md')
>>> os.listdir('.')
['README.md', 'example.tar']

```

`README.md` 文件从存档中提取到文件系统。 调用 `os.listdir()` 确认 `README.md` 文件已成功提取到当前目录中。 要从存档中解压缩或提取所有内容，请使用 `.extractall()` ：

```
>>> tar.extractall(path="extracted/")

```

`.extractall()` 有一个可选的 `path` 参数来指定解压缩文件的去向。 这里，存档被提取到 `extracted` 目录中。 以下命令显示已成功提取存档：

```
$ ls
example.tar  extracted  README.md

$ tree
.
├── example.tar
├── extracted
|   ├── app.py
|   ├── CONTRIBUTING.rst
|   └── README.md
└── README.md

1 directory, 5 files

$ ls extracted/
app.py  CONTRIBUTING.rst  README.md

```

要提取文件对象以进行读取或写入，请使用 `.extractfile()` ，它接收 文件名或 `TarInfo` 对象作为参数。 `.extractfile()` 返回一个可以读取和使用的类文件对象：

```
>>> f = tar.extractfile('app.py')
>>> f.read()
>>> tar.close()

```

打开的存档应在读取或写入后始终关闭。 要关闭存档，请在存档文件句柄上调用 `.close()` ，或在创建 `tarfile`对象时使用with语句，以便在完成后自动关闭存档。 这将释放系统资源，并将你对存档所做的任何更改写入文件系统。

## 创建新的TAR存档

创建新的TAR存档，你可以这样操作:

```
>>> import tarfile

>>> file_list = ['app.py', 'config.py', 'CONTRIBUTORS.md', 'tests.py']
>>> with tarfile.open('packages.tar', mode='w') as tar:
...     for file in file_list:
...         tar.add(file)

>>> # Read the contents of the newly created archive
>>> with tarfile.open('package.tar', mode='r') as t:
...     for member in t.getmembers():
...         print(member.name)
app.py
config.py
CONTRIBUTORS.md
tests.py

```

首先，你要创建要添加到存档的文件列表，这样你就不必手动添加每个文件。

下一行使用with光线文管理器在写入模式下打开名为 `packages.tar` 的新存档。 以写入模式（'w'）打开存档使你可以将新文件写入存档。 将删除存档中的所有现有文件，并创建新存档。

创建并填充存档后，with上下文管理器会自动关闭它并将其保存到文件系统。 最后三行打开刚刚创建的存档，并打印出其中包含的文件的名称。

要将新文件添加到现有存档，请以追加模式（'a'）打开存档：

```
>>> with tarfile.open('package.tar', mode='a') as tar:
...     tar.add('foo.bar')

>>> with tarfile.open('package.tar', mode='r') as tar:
...     for member in tar.getmembers():
...         print(member.name)
app.py
config.py
CONTRIBUTORS.md
tests.py
foo.bar

```

在追加模式下打开存档允许你向其添加新文件而不删除其中已存在的文件。

## 使用压缩存档

`tarfile` 可以读取和写入使用 `gzip`，`bzip2` 和 `lzma` 压缩的TAR存档文件。 要读取或写入压缩存档，请使用`tarfile.open()` ，为压缩类型传递适当的模式。

例如，要读取或写入使用 `gzip` 压缩的TAR存档的数据，请分别使用 `'r:gz'` 或 `'w:gz'` 模式：

```
>>> files = ['app.py', 'config.py', 'tests.py']
>>> with tarfile.open('packages.tar.gz', mode='w:gz') as tar:
...     tar.add('app.py')
...     tar.add('config.py')
...     tar.add('tests.py')

>>> with tarfile.open('packages.tar.gz', mode='r:gz') as t:
...     for member in t.getmembers():
...         print(member.name)
app.py
config.py
tests.py

```

`'w:gz'` 以写模式模式打开 `gzip` 压缩的存档，`'r:gz'` 以读模式打开 `gzip` 压缩的存档。 无法在追加模式下打开压缩存档。 要将文件添加到压缩存档，你必须创建新存档。

------

# 一个更简单的方式创建存档

Python标准库还支持使用 `shutil` 模块中的高级方法创建TAR和ZIP存档。 `shutil` 中的归档实用工具允许你创建，读取和提取ZIP和TAR归档。 这些实用工具依赖于较底层的 `tarfile` 和 `zipfile` 模块。

## 使用 **shutil.make_archive()** 创建存档

`shutil.make_archive()` 至少接收两个参数：归档的名称和归档格式。

默认情况下，它将当前目录中的所有文件压缩为 `format` 参数中指定的归档格式。 你可以传入可选的 `root_dir` 参数来压缩不同目录中的文件。 `.make_archive()` 支持 `zip` ，`tar` ，`bztar` 和 `gztar` 存档格式。

以下是使用 `shutil` 创建TAR存档的方法：

```
import shutil

# shutil.make_archive(base_name, format, root_dir)
shutil.make_archive('data/backup', 'tar', 'data/')

```

这将复制 `data /` 中的所有内容，并在文件系统中创建名为 `backup.tar` 的存档并返回其名称。 要提取存档，请调用 `.unpack_archive()` ：

```
shutil.unpack_archive('backup.tar', 'extract_dir/')

```

调用 `.unpack_archive()` 并传入存档名称和目标目录，将 `backup.tar` 的内容提取到 `extract_dir/` 中。 ZIP存档可以以相同的方式创建和提取。

------

# 读取多个文件

Python支持通过 `fileinput` 模块从多个输入流或文件列表中读取数据。 此模块允许你快速轻松地循环遍历一个或多个文本文件的内容。 以下是使用 `fileinput` 的典型方法：

```
import fileinput
for line in fileinput.input()
    process(line)

```

`fileinput` 默认从传递给 `sys.argv` 的命令行参数获取其输入。

## 使用 **fileinput** 循环遍历多个文件

让我们使用 `fileinput` 构建一个普通的UNIX工具 `cat` 的原始版本。 `cat` 工具按顺序读取文件，将它们写入标准输出。 当在命令行参数中给出多个文件时，`cat` 将连接文本文件并在终端中显示结果：

```
# File: fileinput-example.py
import fileinput
import sys

files = fileinput.input()
for line in files:
    if fileinput.isfirstline():
        print(f'\n--- Reading {fileinput.filename()} ---')
    print(' -> ' + line, end='')
print()

```

在当前目录中有两个文本文件，运行此命令会产生以下输出：

```
$ python3 fileinput-example.py bacon.txt cupcake.txt
--- Reading bacon.txt ---
 -> Spicy jalapeno bacon ipsum dolor amet in in aute est qui enim aliquip,
 -> irure cillum drumstick elit.
 -> Doner jowl shank ea exercitation landjaeger incididunt ut porchetta.
 -> Tenderloin bacon aliquip cupidatat chicken chuck quis anim et swine.
 -> Tri-tip doner kevin cillum ham veniam cow hamburger.
 -> Turkey pork loin cupidatat filet mignon capicola brisket cupim ad in.
 -> Ball tip dolor do magna laboris nisi pancetta nostrud doner.

--- Reading cupcake.txt ---
 -> Cupcake ipsum dolor sit amet candy I love cheesecake fruitcake.
 -> Topping muffin cotton candy.
 -> Gummies macaroon jujubes jelly beans marzipan.

```

`fileinput` 允许你检索有关每一行的更多信息，例如它是否是第一行(.isfirstline())，行号(.lineno())和文件名(.filename())。 你可以在 [这里](https://link.juejin.im/?target=https%3A%2F%2Fdocs.python.org%2F3%2Flibrary%2Ffileinput.html) 读更多关于它的内容。

------

# 总结

你现在知道如何使用Python对文件和文件组执行最常见的操作。 你已经了解使用不同的内置模块来读取，查找和操作文件。

你现在可以用Python来实现:

- 获取目录内容和文件属性
- 创建目录和目录树
- 使用匹配模式匹配文件名
- 创建临时文件和目录
- 移动，重命名，复制和删除文件或目录
- 从不同类型的存档文件中读取和提取数据
- 使用 *fileinput* 同时读取多个文件