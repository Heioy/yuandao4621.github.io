#### 会话

在网页请求中，通常需要多次在元素、页面间进行操作，但网页请求绝大多数都是基于 HTTP 协议进行应用层的数据传输，而 HTTP 协议是一种 `无连接` 的协议，它并不会长久的保存客户端与服务端之间的连接。

回想通常我们浏览网页时，却是登录一次后在后续的一段时间内可以多次进行网页访问而不需要重新登录，正是因为使用了 `会话(Session)`。

`会话` 是一种保存在服务端用来鉴别客户端连接的认证机制。通常第一次登录后，会在服务端生成一个 `session` 用来记录保存客户端的连接信息，之后客户端在规定的时间内连接服务端则会跳过认证。

#### Requests中的会话

requests 库实现了对 HTTP 协议的封装。正是基于 HTTP 协议 `无连接&无状态` 的性质，requests 的请求方式也默认是 `无连接&无状态` 的。

但这不是今天所要说的，事实上，requests 库中同样支持创建 `会话` 对象，通过 `会话` 可以实现跨多个请求的参数使用，同样的，基于一个 `会话` 的多个请求之间保持相同的 Cookie。

通过跨请求保持 cookie

```python
import requests


s = requests.Session()

# 请求1
s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')
# 请求2
r = s.get("http://httpbin.org/cookies")

print(r.text)
# '{"cookies": {"sessioncookie": "123456789"}}'
```

在上面的例子中，通过创建一个会话对象 `s` ，通过 **请求1** 设置 **cookie** 后，并将 **cookie** 信息保存到会话 `s` 中，当 **请求2** 发送 get 请求时，使用的是 **请求1** 的 cookie。

会话也可用来为请求方法提供缺省数据。这是通过为会话对象的属性提供数据来实现的：

```python
s = requests.Session()
s.auth = ('user', 'pass')
s.headers.update({'x-test': 'true'})

# both 'x-test' and 'x-test2' are sent
s.get('http://httpbin.org/headers', headers={'x-test2': 'true'})
```

> 任何你传递给请求方法的字典都会与已设置会话层数据合并。方法层的参数覆盖会话的参数。

通过预先定义 会话对象 `s` 需要使用的 **headers** 信息，在实际发送请求时，可以灵活选择是否传入 **headers** 对象。倘若在请求中传入 **headers**，则最终的 **headers** 信息为 `{'x-test': 'true', 'x-test2': 'true'}`；倘若在请求中未传入 `headers={'x-test2': 'true'}`，则最终使用的 **headers** 信息为 `{'x-test': 'true'}`。

需要注意的是，方法中传入的参数并不会在下一次请求中生效

```python
s = requests.Session()

r = s.get('http://httpbin.org/cookies', cookies={'from-my': 'browser'})
print(r.text)
# '{"cookies": {"from-my": "browser"}}'

r = s.get('http://httpbin.org/cookies')
print(r.text)
# '{"cookies": {}}'
```

通过在方法 `s.get()` 中传入的参数 `cookies={'from-my': 'browser'}` 只会在当前方法中使用，下一次请求中使用默认的 **cookie** 信息。

使用上下文管理器管理会话

```python
with requests.Session() as s:
    s.get('http://httpbin.org/cookies/set/sessioncookie/123456789')
```

在 `with` 语句中，任何在 `s.get()` 之后的请求都会使用同一个 **cookie**，且在退出 `with` 语句时，会话被关闭。

#### `会话` 应用

思路：通过建立会话，使所有的请求方法通过会话进行发送

```python
import json
from typing import (
    Dict, Optional, AnyStr, List, Union
)
from requests.sessions import session
from requests import Response


class Request(object):
    def __init__(self):
        self.session = session()

    def get(self, url: [Union[str, object]], **kwargs) -> Response:
        """
            GET 请求
        :param url: 请求主机端的url地址, str
        :param kwargs:
        :return: <response>
        """
        return self.session.get(url, **kwargs)

    def post(self, url: [Union[str, object]],
             data: Optional[Union[Dict, str]] = None,
             json: Optional[Dict] = None,
             **kwargs) -> Response:
        """
        POST 请求
        :param url: 请求主机端的url地址, str
        :param data:
        :param json:
        :param kwargs:
        :return: <response>
        """
        return self.session.post(url=url, data=data, json=json, **kwargs)

    def put(self, url: [Union[str, object]],
            data: Optional[Union[Dict, str]] = None,
            **kwargs) -> Response:
        """
        PUT
        :param url: 请求主机端的url地址, str
        :param data:
        :param kwargs:
        :return: <response>
        """        
        return self.session.put(url=url, data=json.dumps(data), **kwargs)

    def delete(self, url: [Union[str, object]],
               json: Optional[Dict] = None,
               **kwargs) -> Response:
        """
        DELETE 请求
        :param url: 请求主机端的url地址, str
        :param json:
        :param kwargs:
        :return: <response>
        """
        return self.session.delete(url=url, json=json, **kwargs)

    def __del__(self):
        """关闭会话"""
        self.session.close()


def open_session(url: str):
    with Request() as session:
        session.headers.update({'accept': 'application/json'})


if __name__ == "__main__":
    open_session('http://xxxx')
```




