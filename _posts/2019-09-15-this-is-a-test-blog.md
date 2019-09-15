---
layout: post
title:  "test blog"
date:   2019-09-15 10:50:00 +0800
categories: tech
tags: [tech, test]
---

# 一级标题
## 二级标题
### 三级标题
#### 四级标题

测试代码块：
以下两种方式都可以：
```python
def func():
# comment it
    print("hello)
```

{% highlight python %}
def func():
# comment it
    print("hello)
{% endhighlight %}

行号似乎不太 work：
{% highlight ruby linenos %}
def foo
  puts 'foo'
end
{% endhighlight %}

```java
public class Hello() {
    public static void main(String[] args) {
        System.out.println("Go!");
    }
}
```

测试插入 gist：
{% gist 682d1cb87ef0bdf8e650649476727c11 %}


测试插入图片：
![lovely dog](/assets/dog.jpg)


测试表格：

|  表头   | 表头  |
|  ----  | ----  |
| 单元格  | 单元格 |
| 单元格  | 单元格 |


| 左对齐 | 右对齐 | 居中对齐 |
| :-----| ----: | :----: |
| 单元格 | 单元格 | 单元格 |
| 单元格 | 单元格 | 单元格 |