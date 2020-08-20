---
layout: single
title:  "Python Exception Log"
date:   2020-08-20 22:50:00 +0800
categories: python
tags: [python, exception]
---

今天学到的一个 Python 小技巧：如何正确地在日志中记录 exception？
方法就是，捕获到异常后打印异常信息，使用 `repr(e)` 而不是 `str(e)`，因为前者包含了异常类型和异常的 message，而后者只有 message。

```python
>>> e = Exception("oops")
>>> repr(e)
"Exception('oops',)"
>>> str(e)
'oops'
```

记住了吗？
