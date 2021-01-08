---
title: "logging：如何使用 logging 紀錄事件"
categories: Python
tags: logging
lang: en
toc: true
---

>How to use logging in Python

`Log` 是你的好朋友，尤其是當 **User** 說他昨天可以用，但是今天突然就不能用的時候:expressionless:，<br>
`Log` 絕對是你的好朋友:smirk:。

在 Python 裡面就有內建好用的 [logging](https://docs.python.org/3.6/library/logging.html) 的 package，以下紀錄使用過程。

---

## 1. Basic
最簡單的用法就是，`import logging`，然後就可以用了。

```python
import logging


logging.debug('debug')
logging.info('info')
logging.warning('warning')
logging.error('error')
logging.critical('critical')
```

執行後，你就會在 console 裡面你剛剛輸入的 log 訊息。由於預設 log `level` 是 `WARNING` 以上的訊息才會被輸出，所以 `debug` 與 `info` 就不會出現。
```shell
(technical-note) Andrew-MacAir:logging Andrew$ python app-basic.py
WARNING:root:warning
ERROR:root:error
CRITICAL:root:critical
```

雖然這樣就可以簡單的開始寫 log，但是如果你希望制定 log 訊息的格式、把 log 寫入到檔案或是修改 log `level` 時，這樣的作法就沒有彈性了。

`logging` 提供了 [`basicConfig`](https://docs.python.org/3.6/library/logging.html#logging.basicConfig) 的 method 來做一些基本的設定。

```python
import logging


logging.basicConfig(
    filename='app-basic.log',
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)


if __name__ == "__main__":
    logger = logging.getLogger(__name__)
    logger.debug('debug')
    logger.info('info')
    logger.warning('warning')
    logger.error('error')
    logger.critical('critical')
```

這邊稍微說明一下基本的設定：
- `filename`: 可以設定要寫入的檔案路徑，如果沒有設定時，預設會輸出到 `sys.stderr`，也就是 console。
- `level`: 先前提到的 `level` 預設是 `WARNING`，我這邊修改成 `INFO`。
- `format`: 這邊可以制定 log 訊息的格式，預設是用 [printf-style String Formatting](https://docs.python.org/3/library/stdtypes.html#printf-style-string-formatting) 做字串的格式化，Python 的字串格式化可以參考另一篇 [如何漂亮地格式化你的字串]({{ site.baseurl }}{% post_url 2019-07-18-string-formatting %})。
    - `asctime`: 指的是 log 產生的時間，預設格式是 `2003-07-08 16:49:45,896`。
    - `name`: 指的是 `logger` 的名稱，如果沒有指定的話，預設會是 `root`。
    - `levelname`: 指的是寫入的 log 屬於哪一個 `level`。
    - `message`: 指的是你寫入的 log 訊息。
    - 更多詳細的 `format` 變數說明可以參考 [LogRecord attributes](https://docs.python.org/3.6/library/logging.html#logrecord-attributes)。
- `datefmt`: 如果你不喜歡預設的時間格式，可以在這邊做修改，時間格式是依照 [strftime() and strptime() Behavior](https://docs.python.org/3.6/library/datetime.html#strftime-strptime-behavior) 來去設定，請參考另一篇 [datetime 物件：strftime() and strptime()]({{ site.baseurl }}{% post_url 2019-07-17-datetime-format %})。

```shell
(technical-note) Andrew-MacAir:logging Andrew$ python app-basic.py
2019-10-02 00:14:41 - __main__ - INFO - info
2019-10-02 00:14:41 - __main__ - WARNING - warning
2019-10-02 00:14:41 - __main__ - ERROR - error
2019-10-02 00:14:41 - __main__ - CRITICAL - critical
```
設定完後執行，你就可以看到 log 訊息就會變得比用之前的方式更豐富，設定的彈性也更大了。

## 2. Handler
除了 `basicConfig` 的基本設定外，`logging` 還提供多種 [Logging handlers](https://docs.python.org/3.6/library/logging.handlers.html) 選擇，以下紀錄使用方式。

```python
import logging
import logging.handlers


# 1. Create handlers
console_handler = logging.StreamHandler()
file_handler = logging.FileHandler('handler.log')
time_rotating_handler = logging.handlers.TimedRotatingFileHandler('time-rotating-handler.log')

# 2. Create formatters
console_format = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
file_format = logging.Formatter('%(levelname)s - %(message)s')
time_rotating_format = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# 3. Set the format for handlers
console_handler.setFormatter(console_format)
file_handler.setFormatter(file_format)
time_rotating_handler.setFormatter(time_rotating_format)

# 4. Set the log level for handlers
console_handler.setLevel(logging.INFO)
file_handler.setLevel(logging.INFO)
time_rotating_handler.setLevel(logging.INFO)

# 5. Get a logger
logger = logging.getLogger(__name__)

# 6. Add the handlers to the logger
logger.addHandler(console_handler)
logger.addHandler(file_handler)
logger.addHandler(time_rotating_handler)

if __name__ == "__main__":
    logger.info('info')
    logger.warning('warning')
```
這邊來敘述一下流程：
1. Create handlers:
    - `logging.StreamHandler()`: 預設是輸出到 `sys.stderr`。
    - `logging.FileHandler()`: 可將 log 寫入到檔案中。
    - `logging.handlers.TimedRotatingFileHandler()`: 也是將 log 寫入到檔案中，但是可以額外做 `log rotation` 的功能，詳細設定請參考 [TimedRotatingFileHandler](https://docs.python.org/3.6/library/logging.handlers.html#logging.handlers.TimedRotatingFileHandler)。
2. Create formatters: 制定 log 訊息的格式。
3. Set the format for handlers: 將前一項的格式加入 `handler` 中。
4. Set the log level for handlers: 設定 `handler` 的 log `level`。
5. Get a logger: 取得 `logger`。
6. Add the handlers to the logger: 把第 1 項到第 4 項所設定的 `handler` 加入到前一項的 `logger` 中。

接著執行下去，你就可以看到 log 訊息分別以不同的格式輸出到不同的地方啦。

## 3. Logging configuration
第一個方式對於單一功能與單一格式的 log 或許就夠用了，第二個方法雖然可以應付多種不同格式的 log，但是 `set` 來 `set` 去的讓整個程式碼看起來很複雜。

於是 `logging` 提供了更多元的 [Logging configuration](https://docs.python.org/3.6/library/logging.config.html)，這邊主要支援兩種設定方式，`dictConfig()` 以及 `fileConfig()`，以下紀錄 `dictConfig()` 的使用方式。

```python
import logging
import logging.config


LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'normal': {  # the name of formatter
            'format': '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        },
        'simple': {  # the name of formatter
            'format': '%(levelname)s - %(message)s'
        },
    },
    'handlers': {
        'console1': {  # the name of handler
            'class': 'logging.StreamHandler',  # emit to sys.stderr(default)
            'formatter': 'normal',  # use the above "normal" formatter
        },
        'console2': {  # the name of handler
            'class': 'logging.StreamHandler',  # emit to sys.stderr(default)
            'formatter': 'simple',  # use the above "simple" formatter
        },
        'file1': {  # the name of handler
            'class': 'logging.FileHandler',  # emit to disk file
            'filename': 'f1.log',  # the path of the log file
            'formatter': 'normal',  # use the above "normal" formatter
        },
        'file2': {  # the name of handler
            'class': 'logging.FileHandler',  # emit to disk file
            'filename': 'f2.log',  # the path of the log file
            'formatter': 'simple',  # use the above "simple" formatter
        },
        'time-rotating-file': {  # the name of handler
            'class': 'logging.handlers.TimedRotatingFileHandler',  # the log rotation by time interval
            'filename': 'f3.log',  # the path of the log file
            'when': 'midnight',  # time interval
            'formatter': 'normal',  # use the above "simple" formatter
        },
    },
    'loggers': {
        'c': {  # the name of logger
            'handlers': ['console1', 'console2'],  # use the above "console1" and "console2" handler
            'level': 'INFO',  # logging level
        },
        'f1': {  # the name of logger
            'handlers': ['file1'],  # use the above "file1" handler
            'level': 'INFO',  # logging level
            'propagate': True,
        },
        'f2': {  # the name of logger
            'handlers': ['file2'],  # use the above "file2" handler
            'level': 'INFO',  # logging level
            'propagate': True,
        },
        'time-f': {  # the name of logger
            'handlers': ['time-rotating-file'],  # use the above "time-rotating-file" handler
            'level': 'INFO',  # logging level
            'propagate': True,
        },
    },
}

logging.config.dictConfig(config=LOGGING)
```
這邊稍微說明一下 `SETTING` 這個 `dict` 的設定：
- `formatters`: 這裡可以設定多種格式的 log 訊息，裡面 `format` 的設定方式與第一節介紹相同。
- `handlers`: 這裡可以設定多個 `handler`。
    - `class`: 這邊是選擇 `handler` 的型態，同上一節的介紹。
    - `formatter`: 這邊選擇上一項所設定的格式
- `loggers`: 這邊可以設定多個 `logger`。
    - `handlers`: 在同一個 `logger` 裡面可以選擇一個或是多個 `handler`。
    - `level`: 設定這一個 `logger` 的 log `lever`。

```python
if __name__ == "__main__":
    logger_c = logging.getLogger('c')
    logger_c.info('info')

    logger_f1 = logging.getLogger('f1')
    logger_f1.info('info')

    logger_f2 = logging.getLogger('f2')
    logger_f2.info('info')

    logger_time_f = logging.getLogger('time-f')
    logger_time_f.info('info')
```
接著執行下去，你就可以看到在同一個 application 中使用不同型態的 `logger` 的結果啦。

---
完整的 sample code 請參考 [python-logging](https://github.com/orcahmlee/lab-technical-code/tree/master/Python/logging)。
