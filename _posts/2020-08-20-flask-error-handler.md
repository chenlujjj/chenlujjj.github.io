---
layout: single
title:  "Flask 中的异常处理"
date:   2020-08-20 23:00:00 +0800
categories: python
tags: [python, flask]
---

在开发 flask 应用时，一个常见的需求是能够全局捕获异常并根据异常类型返回相应的 response。
flask 框架提供了这种能力，通过注册 `errorhandler` 函数来实现。它会将某种异常和对应的异常处理函数“绑定”在一起，当异常发生时调用该处理函数。
一般地，可以在 `create_app` 函数中添加：
```python
app.register_error_handler(SomeException, handle_function)
```

这里的异常可以是我们根据业务逻辑自定义的异常，例如 `DeviceNotFound`，关于这部分参见 [Implementing API Exceptions](https://flask.palletsprojects.com/en/1.1.x/patterns/apierrors/)。

也可以是 HTTP 4xx 和 5xx 的异常，这些异常 `werkzeug` 已经定义好了，在 `werkzeug.exceptions` 里，不用我们再自行定义。基类是 `HTTPException`，子类有 `BadRequest`，`InternalServerError` 等。
一种常见地处理 `HTTPException` 的方法是将它的 `name`，`code`，`description` 放在 `json` 中返回给客户端，而不是框架默认的返回一个 html 网页。
对于 `500 InternalServerError` 应该额外处理，因为这个异常往往不是代码中主动 `raise` 的，而是一些“意料外”的状况引起。如果不处理，它默认的 `description` 对于排查问题基本没啥信息量，这对客户端是不友好地。
官方文档给出的方法是，获取到该异常的 `original_exception`，将其信息返回给客户端，见 [Application Errors#Unhandled Exceptions](https://flask.palletsprojects.com/en/1.1.x/errorhandling/#unhandled-exceptions)。

注意，当同时注册了 `InternalServerError` 和 `HTTPException` 的 `handler` 时，因为 `InternalServerError` 是 `HTTPException` 的子类，当遇到 `InternalServerError` 时，会**仅仅**触发其 `handler`，而遇到**其他** `HTTPException` 时，才会触发 `HTTPException` 对应的 `handler`。这一点上，`werkzeug` 做的很巧妙了。