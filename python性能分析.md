# python性能分析

## profile
最近升级了推理服务版本，发现响应时长陡增。但是并没有修改主流程，想看看究竟推理链路那里出现问题了？为了给不熟悉性能分析的伙伴一个参考，决定还是文档化。
功欲善其事欲，必先利其器。首先选择合适的性能分析工具。
其中cProfile和profile是python提供的两个内置的确定性性能分析工具。个人比较钟爱cProfile，所以自然就选择了cProfile啦。


## 尝鲜
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

当然，在实际工程中肯定不能像上一节那么玩。那怎么玩呢？
当然是cProfile和pstats配置一起干活啊。cProfile用来收集性能数据，pstats用于统计分析cProfile记录的性能数据。

这里重点介绍的就是cProfile.Profile类。
python3.8及以上引入了上下文管理器的支持，所以开启性能数据收集非常简单。

```python
with cProfile.Profile() as profile:
   # do some thing
   ...
```
对于python3.8以前的版本，还得老老实实手动开始和关闭。
         `enable()`
             开始收集分析数据。仅在 cProfile 可用。

         `disable()`

             停止收集分析数据。仅在 cProfile 可用。

         `create_stats()`

             停止收集分析数据，并在内部将结果记录为当前 profile。

         `print_stats(sort=- 1)`

             Create a Stats object based on the current profile and print the results to stdout.

         `dump_stats(filename)`

             将当前profile 的结果写入 filename 。


为了方便调用，可以做一个装饰器，方便使用

python3.8以前代码采用：
```python
import os
from typing import Callable, Union
from pathlib import Path


def do_profile(path: Union[str, Path], sortby="tottime"):
    def decorator(func: Callable):
        def _profile(*args, **kwargs):
            flag = os.getenv("PROFILE_ENABLE")
            if flag:
                profile = cProfile.Profile()
                profile.enable()
                result = func(*args, **kwargs)
                profile.disable()
                profile.dump_stats(Path(path).joinpath(f"{func.__name__}.prof"))
                return result
            else:
                result = func(*args, **kwargs)
            return result
        return _profile
    return decorator

```

python3.8及以上版本采用
```python
def do_profile(path: Union[str, Path], sortby="tottime"):
    def decorator(func: Callable):
        def _profile(*args, **kwargs):
            flag = os.getenv("PROFILE_ENABLE")
            if flag:
                with cProfile.Profile() as profile:
                    result = func(*args, **kwargs)
                    profile.dump_stats(Path(path).joinpath(f"{func.__name__}.prof"))
                return result
            else:
                result = func(*args, **kwargs)
            return result
        return _profile
    return decorator
```

那怎么用这个工具函数呢，很简单，需要测试那个调用调用链路，用do_profile装饰一下，运行时开启PROFILE_ENABLE就可以了。如下所示`profile_test.py`。

```python
@do_profile(sys.argv[1], sortby="tottime")
def test():
    import numpy as np
    eye = np.eye(10)
    print(eye)


test()

p = pstats.Stats(os.path.join(sys.argv[1], "test.prof"))
```
运行: `└─▪ PROFILE_ENABLE=1 python profile_test.py ./`, 就能在当前目录看到生成的性能数据文件： `test.prof`


接下来就是展示的时候，该pstats登场了，两行行搞定：
```python
p = pstats.Stats(os.path.join(sys.argv[1], "test.prof"))
print(p.strip_dirs().sort_stats("tottime").print_stats())
```

简单的性能分析就到这里了，后面有时间再写点可视化分析的东西。


## Reference
[1] https://docs.python.org/3/library/profile.html
