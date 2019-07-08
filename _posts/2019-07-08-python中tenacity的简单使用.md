---
layout:     post
title:      python中tenacity的简单使用
subtitle:   工作中常用到的命令
date:       2019-07-08
author:     永泉狂客
header-img: img/post-bg-map.jpg
catalog: true
tags:
    - python
---
### 1.tenacity有什么用？
Tenacity是一个通用的retry库，简化为任何任务加入重试的功能。

它还包含如下特性:

通用的装饰器API
可以设定重试停止的条件（比如设定尝试次数）
可以设定重试间的等待时间（比如在尝试之间使用幂数级增长的wait等待）
自定义在哪些Exception进行重试
自定义在哪些返回值的情况进行重试
协程的重试

### 2.为什么使用tenacity
很多时候，我们都喜欢为代码加入retry功能。比如oauth验证，有时候网络不太灵，我们希望多试几次。这些retry应用的场景看起来不同，其实又很类似。都是判断代码是否正常运行，如果不是则重新开始。那么，有没有一种通用的办法来实现呢？
### 3.使用介绍
1）简单的使用方法，是直接给需要重试的代码加上@retry修饰器，代码抛出异常会被装饰器捕获并进行重试，异常抛出时会不断重试直到代码执行成功
```
from tenacity import retry

import requests

@retry
def conn_req():
    response = requests.get(url="http://www.baidu.com")
    if response.status_code == 200:
        return response.text
    raise Exception

res = conn_req()
print(res)
```

2）加上终止条件的retry

 - 给retry加上一个参数，让它重试3次后不再重试并抛出异常

```
from tenacity import retry,stop_after_attempt

import requests

@retry(stop=stop_after_attempt(3))
def conn_req():
    response = requests.get(url="http://www.baibai.com")
    if response.status_code == 200:
        return response.text
    raise Exception

res = conn_req()
print(res)
```

 - 使用@stop_after_delay可以指定重试间隔，比如下面代码指定5秒后重试

```
from tenacity import retry,stop_after_attempt,stop_after_delay

import requests

@retry(stop=stop_after_delay(5))
def conn_req():
    response = requests.get(url="http://www.baibai.com")
    if response.status_code == 200:
        return response.text
    raise Exception

res = conn_req()
print(res)
```
 - 同时可以使用“|”把多个条件进行组合使用
```
from tenacity import retry, stop_after_delay, stop_after_attempt
import requests

@retry(stop=stop_after_delay(5) |stop_after_attempt(5))
def stop_after_5_s():
    response = requests.get(url='http://www.baidu.com')
    if response.status_code == 200:
        return response.text
    raise Exception

res = stop_after_5_s()
print(res)
```

3）代码重试前等待间隔
 - 使用@wait_fixed在程序重试前等待固定时间，下面就是每隔2秒进行重试
```
from tenacity import retry, wait_fixed
import requests

@retry(wait=wait_fixed(2))
def wait_2_s():
    response = requests.get(url='http://www.baibai.com')
    if response.status_code == 200:
        return response.text
    raise Exception

res = wait_2_s()
print res
```

 - 使用@wait_random随机等待，主要应用爬虫的场景比较多
```
from tenacity import retry, wait_random
import requests

@retry(wait=wait_random(min=1, max=2))
def wait_2_s():
    response = requests.get(url='http://www.baibai.com')
    if response.status_code == 200:
        return response.text
    raise Exception

res = wait_2_s()
print res
```

 - 使用@wait_exponential可以按照指数的等待时间
```
from tenacity import retry, wait_exponential
import requests

@retry(wait=wait_exponential(multiplier=2, min=3, max=100))    # 重试时间间隔满足：2^n * multiplier, n为重试次数，但最多间隔10秒
def wait_2_s():
    response = requests.get(url='http://www.baibai.com')
    if response.status_code == 200:
        return response.text
    raise Exception

res = wait_2_s()
print res
```

 4）带有触发条件的retry语句
```
from tenacity import retry, retry_if_exception_type, retry_if_result

@retry(retry=retry_if_exception_type(IOError))
def might_io_error():
    print("Retry forever with no wait if an IOError occurs, raise any other errors")
    raise Exception

def is_none_p(value):
    """Return True if value is None"""
    return value is None

@retry(retry=retry_if_result(is_none_p))
def might_return_none():
    print("Retry with no wait if return value is None")

@retry(retry=(retry_if_result(is_none_p) | retry_if_exception_type()))
def might_return_none():
    print("Retry forever ignoring Exceptions with no wait if return value is None")
```

 5）在retry前后增加log

```
from tenacity import retry, stop_after_attempt, before_log, after_log, before_sleep_log
import logging

logger = logging.getLogger(__name__)


@retry(stop=stop_after_attempt(3), before=before_log(logger, logging.DEBUG))
def raise_my_exception():
    raise MyException("Fail")


@retry(stop=stop_after_attempt(3), after=after_log(logger, logging.DEBUG))
def raise_my_exception():
    raise MyException("Fail")


@retry(stop=stop_after_attempt(3),
       before_sleep=before_sleep_log(logger, logging.DEBUG))
def raise_my_exception():
    raise MyException("Fail")
```
