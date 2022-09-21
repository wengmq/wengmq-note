# python 日志处理函数封装

```
import logging
import os
def init_log(log_path, level=logging.DEBUG, when="D", backup=7,
             format="%(asctime)s - line:%(lineno)d - [%(levelname)s] : %(message)s",
             datefmt="%Y-%m-%d %H:%M:%S"):
    formatter = logging.Formatter(format, datefmt)
    logger = logging.getLogger()
    logger.setLevel(logging.DEBUG)
    dir = os.path.dirname(log_path)
    if not os.path.isdir(dir):
        os.makedirs(dir)
    from logging.handlers import TimedRotatingFileHandler
    handler = logging.handlers.TimedRotatingFileHandler(log_path, when=when, backupCount=backup)
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    handler.setFormatter(formatter)
    handler.setLevel(level)
    logger.addHandler(handler)

init_log("/tmp/wengmq.log")
logging.info('-' * 80)
logging.info('hhhhh')
logging.debug('aaaa')
logging.error('aaaa')
```

```
$ tail -f /tmp/wengmq.log
2022-04-28 17:06:33 - line:26 - [INFO] : --------------------------------------------------------------------------------
2022-04-28 17:06:33 - line:27 - [INFO] : hhhhh
2022-04-28 17:06:33 - line:28 - [DEBUG] : aaaa
2022-04-28 17:24:59 - line:21 - [INFO] : --------------------------------------------------------------------------------
2022-04-28 17:24:59 - line:22 - [INFO] : hhhhh
2022-04-28 17:24:59 - line:23 - [DEBUG] : aaaa
2022-04-28 17:25:37 - line:21 - [INFO] : --------------------------------------------------------------------------------
2022-04-28 17:25:37 - line:22 - [INFO] : hhhhh
2022-04-28 17:25:37 - line:23 - [DEBUG] : aaaa
2022-04-28 17:25:37 - line:24 - [ERROR] : aaaa
```
