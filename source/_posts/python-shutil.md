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
fsrc: 一个 file-like 对象，表示要复制的源文件  
fdst: 代表目标文件的 file-like 对象，将 fsrc 复制到其中.  
length (可选): 表示缓冲区大小的整数值. 默认情况下会分块读取数据以避免不受控制的内存消耗. 如果 length 为负值, 拷贝数据时不对源数据进行分块循环处理. 如果 fsrc 对象的当前文件位置不为 0，则只有从当前文件位置到文件末尾的内容会被拷贝.  
> 所谓 File-like(文件类) 对象就是像 open() 函数返回的这种有个 read() 方法的对象，在Python中统称为 File-like Object. 除了 File 外，还可以是内存的字节流，网络流，自定义流等等. File-like Object不要求从特定类继承，只要写个 read() 方法就行.  
> StringIO BytesIO 就是在内存中创建的 File-like Object，常用作临时缓冲.  

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
目标位置必须是可写的, 否则将引发 *OSError* 异常. 如果 dst 已经存在，它将被替换.  
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
如果 copymode() 无法修改本机平台上的符号链接，而它被要求这样做, 它将**不做任何操作**.  
版本变化:  
v3.3: 加入 follow_symlinks 参数.  
## shutil.copystat()
```python
shutil.copystat(src, dst, *, follow_symlinks=True)
```
作用: 从 src 拷贝权限位、最近访问时间、最近修改时间以及旗标到 dst. 在 Linux 系统上, copystat() 还会在可能的情况下拷贝"扩展属性". 文件的内容、所有者(owner)和分组(group)属性将不受影响. src 和 dst 均为路径类对象或字符串形式的路径名.  
解释:  
如果 follow_symlinks 为 False, 并且 src 和 dst 均指向符号链接, copystat() 将**作用于符号链接本身**而非该符号链接所指向的文件 -- 从 src 符号链接读取信息，并将信息写入 dst 符号链接.  
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
如果 dst 指定了一个目录，文件将使用 src 中的基准文件名拷贝到 dst 中, 并返回新创建文件所对应的路径.  
如果 follow_symlinks 为 False 且 src 为符号链接, 则 dst 也将被创建为符号链接. 如果 follow_symlinks 为 True 且 src 为符号链接, dst 将拥有 src 所指向的文件的内容.  
shutil.copy() 会拷贝文件数据和文件的权限模式. 其他元数据, 例如文件的创建和修改时间不会被保留. 要保留所有原有的元数据, 请改用 shutil.copy2().  
版本变化:  
v3.3: 添加了 follow_symlinks 参数。 现在会返回新创建文件的路径.  
v3.8: 可能会在内部使用平台专属的快速拷贝系统调用以更高效地拷贝文件.  
