---
layout: post
title: "python脚本发布"
subtitle: 'python script package'
date:       2019-10-14 20:37:00
author: "mudux"
# header-img: "img/post-bg-2015.jpg"
header-style: text
catalog: false
tags:
  - technique
---
python属于解释性语言，如果想在别人的电脑上运行自己的脚本，还得要先装上python环境以及相关的包，费时费力。在最近的一个
项目中，恰好遇到了这种需求，要把自己的程序发布，借助[pyinstaller](https://www.pyinstaller.org/index.html)可以解决。
## pyinstaller安装
```python
pip install pyinstaller
```
或者去网站下载到本地安装。
## pyinstaller使用
1. 命令格式
命令为
```python
pyinstaller [options] script [script]|specfile
```
最简单使用，命令行进入相关目录，运行
```python
pyinstaller myscript.py
```
即可生成先关exe文件。 
2. 打包过程
- 在当前目录生成myscript.spec文件
- 在当前目录创建build文件夹
- 把一些log文件写入build文件夹
- 在当前目录创建dist文件夹
- 在dist文件夹下生成可执行文件
3. 常用参数详解
- -y or --onconfirm
不询问管理员权限
- -D or --onedir
只生成一个文件夹
- -F or --onefile
只生成一个exe文件
- -n or --name 
设置app文件名，默认和myscirpt.py同名
- --add-data 
添加非二进制资源文件
- --add-binary
添加非二进制资源文件
- -hidden-import
不可见导入包
- -d or --debug
调试模式，后可接all, imports, bootloader, noarchive
- -i or --icon
设置程序图标
- -c or --console or --nowindowed
打开控制台（cmd）
- -w or --windowed or --noconsole
不打开控制台（cmd）   

## 例子
1. 命令行  
windows平台
```python
pyinstaller --noconfirm --log-level=WARN ^
    --onefile --nowindow ^
    --add-data="README;." ^
    --add-data="image1.png;img" ^
    --add-binary="libfoo.so;lib" ^
    --hidden-import=secret1 ^
    --hidden-import=secret2 ^
    --icon=..\MLNMFLCN.ICO ^
    myscript.spec
```
Linux平台
```python
pyinstaller --noconfirm --log-level=WARN \
    --onefile --nowindow \
    --add-data="README:." \
    --add-data="image1.png:img" \
    --add-binary="libfoo.so:lib" \
    --hidden-import=secret1 \
    --hidden-import=secret2 \
    --upx-dir=/usr/local/share/ \
    myscript.spec
```
2. 通过spec文件  
通过命令行命令
```python
pyi-makespec myscript.py
```
生成.spec文件，在spec文件中设置资源文件等。   
在我的使用中，需要相关图标文件及实验数据，全部放在data文件夹。

```python
block_cipher = None
a = Analysis(['gui.py'],
             pathex=['D:\\py_remote_local\\imbalancedData'],
             binaries=[],
             datas=[('data', 'data')],
             hiddenimports=['sklearn.utils._cython_blas', 'cython', 'sklearn', 'sklearn.ensemble', 'sklearn.neighbors.typedefs', 'sklearn.neighbors.quad_tree', 'sklearn.tree._utils'],
             hookspath=[],
             runtime_hooks=[],
             excludes=[],
             win_no_prefer_redirects=False,
             win_private_assemblies=False,
             cipher=block_cipher,
             noarchive=False)
pyz = PYZ(a.pure, a.zipped_data,
             cipher=block_cipher)
exe = EXE(pyz,
          a.scripts,
          [],
          exclude_binaries=True,
          name='gui',
          debug=False,
          bootloader_ignore_signals=False,
          strip=False,
          upx=True,
          console=True )
coll = COLLECT(exe,
               a.binaries,
               a.zipfiles,
               a.datas,
               strip=False,
               upx=True,
               upx_exclude=[],
               name='gui')

```  
binaries列表放二进制资源文件，datas列表放非二进制资源文件。里面元素均为一元组，第一个位置为在本地目录，第二个位置为打包后目录。
>注：打包后位置应对应程序中文件路径  

之后运行
```python
pyinstaller [options] mycript.spec
```

## 报错如何解决
1. 先升级pyinstaller
```python
pip install https://github.com/pyinstaller/pyinstaller/archive/develop.zip
```
2. 可能有超过最大depth错误，修改spec文件
3. 设置为``--debug all``模式，打包后在命令行启动exe文件，观察调试信息
> 注：调试完应该编译发布版，否则运行时输出很多中间信息。

4. 经常有filenotfound错误，则设置hiddenimports  
5. [github link](https://github.com/pyinstaller/pyinstaller/wiki/How-to-Report-Bugs)
## what to do next
``docker``?
