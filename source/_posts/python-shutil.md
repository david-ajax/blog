---
title: 使用 Python 的 Shutil 库操作文件
date: 2022-2-20
---
> shutil 模块提供了一系列对文件和文件集合的高阶操作, 特别是提供了一些支持文件拷贝和删除的函数. 
> *--- https://docs.python.org*
<!--more-->
# 导入库
由于 shutil 是一个 BIF(Built-in Function) 所以不需要额外的安装操作
```python
import shutil # 通过 shutil.function(args) 的形式访问
from shutil import * # 通过 function(args) 的形式访问 (会污染命名空间)
```
# 目录和文件操作
## shutil.copyfileobj()
```python
shutil.copyfileobj(fsrc, fdst[, length])
```
作用: 将文件类(file-like)对象 fsrc 的内容拷贝到文件类对象 fdst.  
解释:  
fsrc: 一个 file-like 对象,表示要复制的源文件  
fdst: 代表目标文件的 file-like 对象,将 fsrc 复制到其中.  
length (可选): 表示缓冲区大小的整数值. 默认情况下会分块读取数据以避免不受控制的内存消耗. 如果 length 为负值, 拷贝数据时不对源数据进行分块循环处理. 如果 fsrc 对象的当前文件位置不为 0,则只有从当前文件位置到文件末尾的内容会被拷贝.  
> 所谓 File-like(文件类) 对象就是像 open() 函数返回的这种有个 read() 方法的对象,在Python中统称为 File-like Object. 除了 File 外,还可以是内存的字节流,网络流,自定义流等等. File-like Object不要求从特定类继承,只要写个 read() 方法就行.  
> StringIO BytesIO 就是在内存中创建的 File-like Object,常用作临时缓冲.  

返回类型：此方法不返回任何值.
## shutil.copyfile()
```python
shutil.copyfile(src, dst, *, follow_symlinks=True)
```
示例:
```python
shutil.copyfile("a.file", "b.file") # 将 a.file 的内容复制到 b.file. 如果 a.file 指向 c.file, 则 b.file 指向 c.file, 而非复制 c.file
```
作用: 将名为 src 的文件的内容(不包括元数据)复制到名为 dst 的文件.  
解释:  
src 和 dst 均为路径类对象或以字符串形式给出的路径名.  
dst 必须是完整的目标文件名.  
如果 src 和 dst 指定了同一个文件, 则将引发 *SameFileError.*  
目标位置必须是可写的, 否则将引发 *OSError* 异常. 如果 dst 已经存在,它将被替换.  
特殊文件如字符或块设备以及管道无法用此函数来拷贝.(比如 /dev/sda1(Linux 下的硬盘设备) 等等)  
如果 follow_symlinks 为 Flase 且 src 为符号链接, 则将**创建一个新的符号链接**而不是拷贝 src 所指向的文件(即拷贝符号链接本身).  
如果 follow_symlinks 为 True 且 src 为符号链接, 则将**拷贝 src 所指向的文件**(即拷贝符号链接本身).  
版本变化:  
v3.3: 引发 IOError 而不是 *OSError*. 增加了 follow_symlinks 参数.  
v3.4: 引发 SameFileError 而不是 *Error*. 由于前者是后者的子类, 此改变向后兼容.  
v3.8: 可能会在内部使用平台专属的快速拷贝系统调用以更高效地拷贝文件.  
## shutil.copymode()
```python
shutil.copymode(src, dst, *, follow_symlinks=True)
```
作用: 从 src 拷贝权限位信息到 dst. 文件的内容、所有者和分组将不受影响.(类似于 chmod)  
解释:  
src 和 dst 均为路径类对象或字符串形式的路径名.  
如果 follow_symlinks 为 Flase, 并且 src 和 dst 均为符号链接, shutil.copymode() 将尝试**修改 dst 本身**的模式(而非它所指向的文件).  
如果 follow_symlinks 为 True, 并且 src 和 dst 均为符号链接, shutil.copymode() 将尝试**修改 det 指向的文件**的模式.  
此功能**并不是在所有平台上均可用**(不同操作系统的文件系统权限机制不同).(见 shutil.copystat())  
如果 copymode() 无法修改本机平台上的符号链接,而它被要求这样做, 它将**不做任何操作**.  
版本变化:  
v3.3: 加入 follow_symlinks 参数.  
## shutil.copystat()
```python
shutil.copystat(src, dst, *, follow_symlinks=True)
```
作用: 从 src 拷贝权限位、最近访问时间、最近修改时间以及旗标到 dst. 在 Linux 系统上, copystat() 还会在可能的情况下拷贝"扩展属性". 文件的内容、所有者(owner)和分组(group)属性将不受影响. src 和 dst 均为路径类对象或字符串形式的路径名.  
解释:  
如果 follow_symlinks 为 False, 并且 src 和 dst 均指向符号链接, copystat() 将**作用于符号链接本身**而非该符号链接所指向的文件 -- 从 src 符号链接读取信息,并将信息写入 dst 符号链接.  
> 关于平台功能差异的注解:(记得 import os)  
> 并非所有平台都提供检查和修改符号链接的功能.Python 本身可以告诉你哪些功能是在本机上可用的.  
> 如果 os.chmod in os.supports_follow_symlinks 为 True, 则 copystat() 可以修改符号链接的权限位.  
> 如果 os.utime in os.supports_follow_symlinks 为 True, 则 copystat() 可以修改符号链接的最近访问和修改时间.  
> 如果 os.chflags in os.supports_follow_symlinks 为 True, 则 copystat() 可以修改符号链接的旗标.  
在此功能部分或全部不可用的平台上, 当被要求修改一个符号链接时, copystat() 将尽量拷贝所有内容. copystat() **一定不会**返回失败信息.  
版本变化:  
v3.3: 添加了 follow_symlinks 参数并且支持 Linux 扩展属性.  
## shutil.copy()
```python
shutil.copy(src, dst, *, follow_symlinks=True)
```
作用: 将文件 src 拷贝到文件或目录 dst, 并复制文件数据和文件的权限模式.  
解释:  
src 和 dst 应为 路径类对象或字符串.  
如果 dst 指定了一个目录,文件将使用 src 中的基准文件名拷贝到 dst 中, 并返回新创建文件所对应的路径.  
如果 follow_symlinks 为 False 且 src 为符号链接, 则 dst 也将被创建为符号链接. 如果 follow_symlinks 为 True 且 src 为符号链接, dst 将拥有 src 所指向的文件的内容.  
shutil.copy() 会拷贝文件数据和文件的权限模式. 其他元数据, 例如文件的创建和修改时间不会被保留. 要保留所有原有的元数据, 请改用 shutil.copy2().  
版本变化:  
v3.3: 添加了 follow_symlinks 参数. 现在会返回新创建文件的路径.  
v3.8: 可能会在内部使用平台专属的快速拷贝系统调用以更高效地拷贝文件.  
## shutil.copy2()
```python
shutil.copy2(src, dst, *, follow_symlinks=True)
```
作用: copy() + copystat() 即保留文件元数据的(metadada) 的 copy()  
解释:   
当 follow_symlinks 为 False 且 src 为符号链接时, copy2() 会尝试将来自 src 符号链接本身的所有元数据拷贝到新创建的 dst 符号链接.  
但是, 此功能不是在所有平台上均可用. 在此功能部分或全部不可用的平台上, copy2() 将尽量保留所有元数据, copy2() **一定不会**由于无法保留文件元数据而引发异常.  
copy2() 会使用 copystat() 来拷贝文件元数据.  
版本更改:  
v3.3: 添加了 follow_symlinks 参数, 还会尝试拷贝扩展文件系统属性 (目前仅限 Linux)..现在会返回新创建文件的路径.  
v3.8: 可能会在内部使用平台专属的快速拷贝系统调用以更高效地拷贝文件.
## shutil.copytree()
```python
shutil.copytree(src, dst, symlinks=False, ignore=None, copy_function=copy2, ignore_dangling_symlinks=False, dirs_exist_ok=False)
```
作用: 将以 src 为 Root 的整个目录树拷贝到名为 dst 的目录并返回目标目录.  
解释:  
dirs_exist_ok: 指明是否要在一个或多个目标文件已存在的情况下引发异常.  
目录的权限和时间会通过 copystat() 来拷贝, 单个文件则会使用 copy2() 来拷贝.  
如果 symlinks 为 True, 源目录树中的符号链接会在新目录树中也表示为符号链接, 并且原链接的元数据在平台允许的情况下也会被拷贝; 如果为 False 或省略, 则会将被链接文件的内容和元数据拷贝到新目录树.  
当 symlinks 为 False 时,如果符号链接所指向的文件不存在, 则会在拷贝进程的末尾将一个异常添加到 Error 异常中的错误列表. 如果你希望屏蔽此异常, 那就将 ignore_dangling_symlinks 设为 True.  
注意: 此选项在不支持 os.symlink() 的平台上将不起作用.  
ignore: 一个可调用对象, 该对象将接受 copytree() 所访问的目录以及 os.listdir() 所返回的目录内容列表作为其参数. 由于 copytree() 是递归地被调用的, ignore 可调用对象对于每个被拷贝目录都将被调用一次. 该可调用对象必须返回一个相对于当前目录的目录和文件名序列(即其第二个参数的子集); 随后这些名称将在拷贝进程中被忽略. ignore_patterns() 可被用于创建这种基于 glob 风格模式来忽略特定名称的可调用对象. 如果发生了异常,将引发一个附带原因列表的 Error.  
copy_function: 一个将被用来拷贝每个文件的可调用对象(用于替换 copy2()). 它在被调用时会将源路径和目标路径作为参数传入. 默认情况下, copy2() 将被使用,但任何支持同样签名(与 copy() 一致)都可以使用.  
版本变化:  
v3.3: 当 symlinks 为假值时拷贝元数据. 现在会返回 dst.
v3.2: 添加了 copy_function 参数以允许提供定制的拷贝函数. 添加了 ignore_dangling_symlinks 参数以便在 symlinks 为假值时屏蔽符号链接错误.
v3.8: 可能会在内部使用平台专属的快速拷贝系统调用以更高效地拷贝文件. 增加 dirs_exist_ok 形参.

## shutil.ignore_patterns()
```python
shutil.ignore_patterns(*patterns)
# e.g. 忽略后缀名为 pyc 与前缀是 tmp 的文件
shutil.copytree(src, dst, ignore=shutil.ignore_patterns('*.pyc', 'tmp*'))
```
作用: 辅助 copytree() 忽略掉一些文件  
解释:  
*patterns: 忽略文件的参数列表, 以星号作为通配符
