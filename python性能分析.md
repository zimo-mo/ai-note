# python性能分析

## profile
最近升级了推理服务版本，发现响应时长陡增。但是并没有修改主流程，想看看究竟推理链路那里出现问题了。功欲善其事欲，必先利其器。首先选择合适的性能分析工具。
其中cProfile和profile是python提供的两个内置的确定性性能分析工具。个人比较钟爱cProfile，所以自然就选择了cProfile啦。


**尝鲜**
惯例，先按照官方文档来个`hello word`: `test.py`
```python
import cProfile
import re

# sort按运行时间排序
cProfile.run('re.compile("foo|bar")', sort="tottime")

```

运行看看结果：
```
└─▪ python test.py
         244 function calls (237 primitive calls) in 0.001 seconds

   Ordered by: internal time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.001    0.001 {built-in method builtins.exec}
        2    0.000    0.000    0.000    0.000 sre_parse.py:493(_parse)
      3/1    0.000    0.000    0.000    0.000 sre_compile.py:71(_compile)
      3/1    0.000    0.000    0.000    0.000 sre_parse.py:174(getwidth)
        1    0.000    0.000    0.000    0.000 enum.py:797(_create_pseudo_member_)
        1    0.000    0.000    0.000    0.000 enum.py:869(_decompose)
        1    0.000    0.000    0.000    0.000 re.py:289(_compile)
        ...
```
**结果解读**：
其中总共运行时长为0.001s, 第一行显示监听了244个调用。在这些调用中，有237个是原始接口 ，这意味着调用不是通过递归引发的。
Ordered by: internal time ，表示最右边列中的文本字符串用于对输出进行排序。
  **ncalls**： 调用次数
  **tottime**： 在指定函数中消耗的总时间（不包括调用子函数的时间）
  **percall**： 是 tottime 除以 ncalls 的商
  **cumtime**: 指定的函数及其所有子函数（从调用到退出）消耗的累积时间。这个数字对于递归函数来说是准确的。
  **percall**： 函数运行一次的平均时间
  **filename**: 函数名称

## 最佳实践


## Reference
[1] https://docs.python.org/zh-cn/3/library/profile.html
