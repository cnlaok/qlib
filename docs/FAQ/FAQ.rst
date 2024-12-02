 ### Qlib 常见问题解答

#### Qlib 常见问题

**目录**

- [1. RuntimeError: An attempt has been made to start a new process before the current process has finished its bootstrapping phase...](#1-runtimeerror-an-attempt-has-been-made-to-start-a-new-process-before-the-current-process-has-finished-its-bootstrapping-phase)
- [2. qlib.data.cache.QlibCacheException: It sees the key(...) of the redis lock has existed in your redis db now.](#2-qlibdatacacheqlibcacheexception-it-sees-the-key-of-the-redis-lock-has-existed-in-your-redis-db-now)
- [3. ModuleNotFoundError: No module named 'qlib.data._libs.rolling'](#3-modulenotfounderror-no-module-named-qlibdata_libsrolling)
- [4. BadNamespaceError: / is not a connected namespace](#4-badnamespaceerror-is-not-a-connected-namespace)
- [5. TypeError: send() got an unexpected keyword argument 'binary'](#5-typeerror-send-got-an-unexpected-keyword-argument-binary)

------

#### 1. RuntimeError: An attempt has been made to start a new process before the current process has finished its bootstrapping phase...

**错误信息**：

```console
RuntimeError:
        An attempt has been made to start a new process before the
        current process has finished its bootstrapping phase.

        This probably means that you are not using fork to start your
        child processes and you have forgotten to use the proper idiom
        in the main module:

            if __name__ == '__main__':
                freeze_support()
                ...

        The "freeze_support()" line can be omitted if the program
        is not going to be frozen to produce an executable.
```

这是由于 Windows 操作系统下多进程的限制引起的。更多信息请参考 [这里](https://stackoverflow.com/a/24374798)。

**解决方案**：在主模块的 `if __name__ == '__main__'` 子句中使用 `D.features` 来选择启动方法。例如：

```python
import qlib
from qlib.data import D

if __name__ == "__main__":
    qlib.init()
    instruments = ["SH600000"]
    fields = ["$close", "$change"]
    df = D.features(instruments, fields, start_time='2010-01-01', end_time='2012-12-31')
    print(df.head())
```

#### 2. qlib.data.cache.QlibCacheException: It sees the key(...) of the redis lock has existed in your redis db now.

**错误信息**：

```console
qlib.data.cache.QlibCacheException: It sees the key of the redis lock has existed in your redis db now.
```

**解决方案**：可以使用以下命令清除 Redis 键并重新运行命令：

```bash
redis-cli
> select 1
> flushdb
```

如果问题未解决，使用 `keys *` 查找是否存在多个键。如果是，尝试使用 `flushall` 清除所有键。

**注意**：`qlib.config.redis_task_db` 默认值为 `1`，用户可以使用 `qlib.init(redis_task_db=<other_db>)` 设置。

如果问题仍然存在，请随时在我们的 GitHub 仓库中提交新问题。我们总是仔细检查每个问题，并尽力解决它们。

#### 3. ModuleNotFoundError: No module named 'qlib.data._libs.rolling'

**错误信息**：

```python
#### Do not import qlib package in the repository directory in case of importing qlib from . without compiling #####
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
File "qlib/qlib/__init__.py", line 19, in init
    from .data.cache import H
File "qlib/qlib/data/__init__.py", line 8, in <module>
    from .data import (
File "qlib/qlib/data/data.py", line 20, in <module>
    from .cache import H
File "qlib/qlib/data/cache.py", line 36, in <module>
    from .ops import Operators
File "qlib/qlib/data/ops.py", line 19, in <module>
    from ._libs.rolling import rolling_slope, rolling_rsquare, rolling_resi
ModuleNotFoundError: No module named 'qlib.data._libs.rolling'
```

**解决方案**：

- 如果在使用 `PyCharm` IDE 导入 `qlib` 包时出现此错误，用户可以在项目根文件夹中执行以下命令来编译 Cython 文件并生成可执行文件：

  ```bash
  python setup.py build_ext --inplace
  ```

- 如果在使用命令 `python` 导入 `qlib` 包时出现此错误，用户需要更改运行目录，以确保脚本不在项目目录中运行。

#### 4. BadNamespaceError: / is not a connected namespace

**错误信息**：

```python
File "qlib_online.py", line 35, in <module>
    cal = D.calendar()
File "e:\code\python\microsoft\qlib_latest\qlib\qlib\data\data.py", line 973, in calendar
    return Cal.calendar(start_time, end_time, freq, future=future)
File "e:\code\python\microsoft\qlib_latest\qlib\qlib\data\data.py", line 798, in calendar
    self.conn.send_request(
File "e:\code\python\microsoft\qlib_latest\qlib\qlib\data\client.py", line 101, in send_request
    self.sio.emit(request_type + "_request", request_content)
File "G:\apps\miniconda\envs\qlib\lib\site-packages\python_socketio-5.3.0-py3.8.egg\socketio\client.py", line 369, in emit
    raise exceptions.BadNamespaceError(
BadNamespaceError: / is not a connected namespace.
```

**解决方案**：`qlib` 中的 `python-socketio` 版本需要与 `qlib-server` 中的 `python-socketio` 版本相同：

```bash
pip install -U python-socketio==<qlib-server python-socketio version>
```

#### 5. TypeError: send() got an unexpected keyword argument 'binary'

**错误信息**：

```python
File "qlib_online.py", line 35, in <module>
    cal = D.calendar()
File "e:\code\python\microsoft\qlib_latest\qlib\qlib\data\data.py", line 973, in calendar
    return Cal.calendar(start_time, end_time, freq, future=future)
File "e:\code\python\microsoft\qlib_latest\qlib\qlib\data\data.py", line 798, in calendar
    self.conn.send_request(
File "e:\code\python\microsoft\qlib_latest\qlib\qlib\data\client.py", line 101, in send_request
    self.sio.emit(request_type + "_request", request_content)
File "G:\apps\miniconda\envs\qlib\lib\site-packages\socketio\client.py", line 263, in emit
    self._send_packet(packet.Packet(packet.EVENT, namespace=namespace,
File "G:\apps\miniconda\envs\qlib\lib\site-packages\socketio\client.py", line 339, in _send_packet
    self.eio.send(ep, binary=binary)
TypeError: send() got an unexpected keyword argument 'binary'
```

**解决方案**：`python-engineio` 版本需要与 `python-socketio` 版本兼容，参考：https://github.com/miguelgrinberg/python-socketio#version-compatibility

```bash
pip install -U python-engineio==<compatible python-socketio version>
# 或者
pip install -U python-socketio==3.1.2 python-engineio==3.13.2
```