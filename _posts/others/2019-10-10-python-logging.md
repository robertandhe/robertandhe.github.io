---
layout: post
title: "python日志记录"
subtitle: 'python logging'
date:       2019-10-10 21:00:00
author: "mudux"
# header-img: "img/post-bg-2015.jpg"
header-style: text
catalog: true
tags:
  - technique
---
# <center>logging package</center>
之所以产生这个需求是因为在做一个ui界面程序是，需要把程序中间状态print到``wx.TextCtrl``，在操作过程中发现大规模写入
时出现程序卡，虽然在运行，但文本框无法及时写入，当前程序运行完才显示结果。猜测和多线程有关，但暂时不会,orz,所以转向日志
记录及时把结果输出到控制台。
## 日志功能
日志功能应以所追踪事件级别或严重性而定。各级别适用性如下：
<!-- <table>
    <tr>
        <th>级别</th>
        <th>何时使用</th>
    </tr>
    <tr>
        <th>DEBUG</th>
        <th>细节信息，仅当诊断问题时适用</th>
    </tr>
    <tr>
        <th>INFO</th>
        <th>确认程序按预期运行</th>
    </tr>
    <tr>
        <th>WARNING</th>
        <th>表明有已经或即将发生的意外（例如：磁盘空间不足）。程序仍按预期进行</th>
    </tr>
    <tr>
        <th>ERROR</th>
        <th>由于严重的问题，程序的某些功能已经不能正常执行</th>
    </tr>
    <tr>
        <th>CRITICAL</th>
        <th>严重的错误，表明程序已不能继续执行</th>
    </tr>
</table> -->

|级别       |何时使用   |
| ----------- | --------- |
|DEBUG  |细节信息，仅当诊断问题时适用   |
|INFO  |确认程序按预期运行   |
|WARNING  |表明有已经或即将发生的意外（例如：磁盘空间不足）,程序仍按预期进行   |
|ERROR |由于严重的问题，程序的某些功能已经不能正常执行   |
|CRITICAL  |严重的错误，表明程序已不能继续执行   |  
 
默认的级别是``WARNING``，意味着只会追踪该级别及以上的事件，除非更改日志配置。
## 基本概念
logging日志库采用模块化方法，提供几类组件：记录器、处理程序、过滤器和格式化程序。
- 记录器暴露了应用程序代码直接使用的接口
- 处理程序将日志记录（由记录器创建）发送到适当的目标
- 过滤器提供了更精细的附加功能，用于确定要输出的日志记录
- 格式化程序指定最终输出中日志记录的样式
日志事件信息在 LogRecord 实例中的记录器、处理程序、过滤器和格式化程序之间传递。  
通过调用Logger类实例来执行日志记录。在命名记录器时使用的一个好习惯是在每个使用日志记录的模块中使用模块级记录器，命名如下:
```python
logger = logging.getLogger(__name__)
``` 

## 记录流程
![logging](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-10-10/logging_outline.PNG)
## 记录器
Logger对象三个作用：1.向应用程序代码公开几个方法；2. 根据严重性或过滤器对象确定要处理的日志消息；3. 将相关的日志消息
传递给所有感兴趣的日志处理程序。
记录器方法：配置和消息发送。
1. 配置
- Logger.setLevel()指定处理的最低严重性日志消息
- Logger.addHandler()和Logger.removeHandler()从记录器对象添加和删除处理程序对象
- Logger.addFilter()和Logger.removeFilter()添加或移除记录器对象中的过滤器
2. 创建消息
- Logger.debug(),Logger().info(),Logger.warning(),Logger.error()和Logger.critical()创建日志记录
- Logger.exception()创建于Logger.error()相似日志。区别在于Logger.exception()同时记录堆栈追踪
- Logger.log()将日志级别作为显式参数
getLogger()返回对具有指定名称的记录器实例的引用。

## 处理程序
Handler对象负责将适当的日志消息分派给处理程序的指定目标。Logger对象使用addHandler()方法添加处理程序对象。
标准库包含很多处理程序类型：[StreamHandler()](https://docs.python.org/zh-cn/3/library/logging.handlers.html#logging.StreamHandler)和[FileHandler()](https://docs.python.org/zh-cn/3/library/logging.handlers.html#logging.FileHandler)。
StreamHandler()把消息发送到sys.stdout,sys.stderr等。
FileHandler()把消息写入磁盘文件。  
处理程序配置：
- setLevel()
- setFormatter()
- addFilter()

## 格式化程序
格式化程序对象配置日志消息的最终顺序、结构和内容。应用程序代码可以实例化格式化程序类。
```python
logging.Formatter.__init__(fmt=None, datefmt=None, style='%')¶
```
style 是`` ％``，``'{ '`` 或 ``'$' ``之一。分别对应``%(<dictionary key>)s``,``str.format()``和``string.Template.substitute()``
## 配置日志记录
三种方式：
1. 使用调用上面列出的配置方法的 Python 代码显式创建记录器、处理程序和格式化程序
2. 创建日志配置文件并使用``fileConfig()``函数读取它
3. 创建配置信息字典并将其传递给``dictConfig()``函数[link](https://docs.python.org/zh-cn/3/library/logging.config.html#logging-config-api)
  
Example:
```python
import logging
import logging.config

logging.config.fileConfig('logging.conf')

# create logger
logger = logging.getLogger('simpleExample')

# 'application' code
logger.debug('debug message')
logger.info('info message')
logger.warning('warn message')
logger.error('error message')
logger.critical('critical message')
```
``logging.conf``文件
```python
[loggers]
keys=root,simpleExample

[handlers]
keys=consoleHandler

[formatters]
keys=simpleFormatter

[logger_root]
level=DEBUG
handlers=consoleHandler

[logger_simpleExample]
level=DEBUG
handlers=consoleHandler
qualname=simpleExample
propagate=0

[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=simpleFormatter
args=(sys.stdout,)

[formatter_simpleFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
datefmt=
```
## 我的使用
显式配置
```python
logging.basicConfig(level=logging.DEBUG, format="%(asctime)s %(name)s %(levelname)s %(message)s")
self.logger = logging.getLogger(__name__)
handler1 = logging.StreamHandler(sys.stdout)
handler2 = logging.FileHandler(filename='./result/log.log')
self.logger.setLevel(logging.NOTSET)
handler1.setLevel(logging.NOTSET)
handler2.setLevel(logging.NOTSET)
self.logger.addHandler(handler1)
self.logger.addHandler(handler2)
```
需要输出时
```python
self.logger.debug("select sample size: %d" % int(i))
```
console输出
![logging](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-10-10/logging_output.PNG)
打包后控制台输出
![logging](https://raw.githubusercontent.com/robertandhe/MarkDownPhotos/master/2019-10-10/logging_output2.PNG)
## next
``fileConfig()`` and ``dictConfig()``
[link](https://docs.python.org/zh-cn/3/library/logging.config.html#logging-config-api)