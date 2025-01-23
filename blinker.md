# Blinker

Blinker provides fast & simple object-to-object and broadcast signaling for Python objects.


## 为什么用blinker

1、灵活，允许多个事件sender，多个事件触发器receiver

2、性能，事件发送和处理是异步的，不会阻塞主线程，当有耗时操作时，可以异步执行

3、可扩展，可以添加新的事件和触发器，而不需要修改现有的代码

4、可维护，事件和触发器可以单独管理和维护，不会相互影响

常见用法：
- **解耦事件触发和事件处理**，使代码更灵活，更好拓展；
- **对于固定事件流程，提供事件钩子**，供外部扩展
- 配合celery，解耦长时间操作，常见如数据库操作、文件操作、日志记录

## 实践1 事件模块

```python
# src:Dify
from blinker import signal

# 事件
app_was_started = signal('app_was_started')

# 事件触发器 Receiver
@app_was_started.connect
def handle(sender, **kwargs):
    ... # 处理事件操作

# 事件发送者
class AppService:
    def create_app(self, account):
        app = ... # init
        # 事件触发，将 sender 发给 signal，signal 会调用所有注册的 Receiver
        app_was_started.send(app, account=account)

```
注意：事件触发是将 sender 发给 signal，由 signal 调用所有注册的 Receiver

## 实践2 定义事件钩子，供外部扩展

```python
# src:Flask
from blinker import Namespace

# This namespace is only for signals provided by Flask itself.
_signals = Namespace()

request_started = _signals.signal("request-started")
request_finished = _signals.signal("request-finished")
request_tearing_down = _signals.signal("request-tearing-down")
got_request_exception = _signals.signal("got-request-exception")

class Flask(App):
    ...
    def handle_exception(self, e) -> Response:
        exc_info = sys.exc_info()
        got_request_exception.send(self, _async_wrapper=self.ensure_sync, exception=e)
        ...
    
    def finalize_request(self, rv, from_error_handler=False) -> Response:
        ...
        request_finished.send(
                self, _async_wrapper=self.ensure_sync, response=response
            )
        ...

```
你可以简单看下 Flask 的源码，Flask 一个请求的生命周期基于这些事件钩子供外部拓展的。并且你可以看到，这些 signal 在 Flask 基本只有`signal.send`，没有 Handler，即`signal.connect`。

需要注意`signal.send`的参数，来定义合适的Hanlder。


## 资料

- [Blinker 文档](https://blinker.readthedocs.io/en/stable/)
- [Blinker 源码](https://github.com/pallets-eco/blinker)
- [Flask 源码](https://github.com/pallets/flask)
- [Dify 源码](https://github.com/langgenius/dify)