---
title: "argparse：如何使用 argparse 製作 CLI"
categories: Python
tags: cli argparse
lang: en
toc: true
---

How to use argparse creating CLI in Python
===

Command-Line Interface(CLI)，一種透過文字來跟你的程式溝通的介面，在 Linux 或 MacOS 介面中一定會用到 CLI 來達到你想要的目的，而且身為一個潮潮的工程師一定要會打一些指令:smirk:。

由於工作上的需求，需要透過 CLI 下參數來呼叫建立好的 model。常常用別人建立的 CLI，要自己製作 CLI 還是第一次，幸好在 Python 裡面就有內建好用的 [argparse](https://docs.python.org/3.6/library/argparse.html) 的 package，以下利用一個簡單計算日期的小程式來紀錄使用過程。

---
## 1. Calculator
```python
# date_diff_bad.py
import sys
from datetime import datetime


def calc(start_date: str, end_date: str, include: bool = False) -> int:
    date_format = '%Y-%m-%d'
    start_d = datetime.strptime(start_date, date_format)
    end_d = datetime.strptime(end_date, date_format)
    diff = end_d - start_d

    if include:
        return diff.days + 1
    else:
        return diff.days


if __name__ == "__main__":
    res = calc('2019-01-01', '2019-01-03')
    print(f'{res} days.')

```
這邊是一段小程式，用來計算兩個日期之間一共有幾天；預設是不加上第一天，如果要把第一天也算進去的話，只要傳入 `include=True` 即可。

寫完之後，該怎麼使用呢？

```shell
(technical-note) Andrew-MacAir:argparse Andrew$ python date_diff_bad.py
2 days.
```

這樣就可以算出兩個日期中間，間隔了 2 天，是不是很簡單！但是，哪裡怪怪的......

這樣的作法一點都不方便，每當我想要換日期的時候，我都必須編輯 `date_diff_bad.py` 這個檔案，然後再執行它才能獲得結果。

## 2. sys.argv
那我們試著來傳入參數試試看吧，最簡單的傳參數方法是用 `sys.argv`，利用下面的程式來看看它怎麼運作。

```python
# simple.py
import sys


if __name__ == "__main__":
    for i in range(len(sys.argv)):
        print(sys.argv[i])

```

```shell
(technical-note) Andrew-MacAir:argparse Andrew$ python simple.py Hi I am Andrew
simple.py
Hi
I
am
Andrew
```

呦，這樣我們可以觀察到 `sys.argv[0]` 其實就是檔案名稱，接下來的 `sys.argv[1]`, `sys.argv[2]`,...依序分別就是你傳入的參數。

所以其實只要把上一小節中呼叫 `calc()` 的部分換掉，
```python
......
if __name__ == "__main__":
    res = calc('sys.argv[1]', 'sys.argv[2]')
    print(f'{res} days.')
```

就可以用傳參數的方式，使用 `calc()` 這個 function。
```shell
(technical-note) Andrew-MacAir:argparse Andrew$ python date_diff_bad.py 2019-01-01 2019-01-03
2 days.
```

大致上，CLI 看起來好像完成了，但是是不是少了些什麼:worried:？

如果其他人看不懂需要 `--help` 呢？如果我想要指定參數呢？
## 3. argparse-positional
[argparse](https://docs.python.org/3.6/library/argparse.html) 裡面有很多的用法，這一小節先來介紹如何使用 `positional` 的方式。

```python
# date_diff_pos.py
from datetime import datetime
from argparse import ArgumentParser


def calc(start_date: str, end_date: str, include: bool = False) -> int:
    date_format = '%Y-%m-%d'
    start_d = datetime.strptime(start_date, date_format)
    end_d = datetime.strptime(end_date, date_format)
    diff = end_d - start_d

    if include:
        return diff.days + 1
    else:
        return diff.days


if __name__ == "__main__":
    parser = ArgumentParser()
    parser.add_argument(
        'start_date', type=str,
        help='The start date'
    )
    parser.add_argument(
        'end_date', type=str,
        help='The end date'
    )
    parser.add_argument(
        '-i', '--include', action='store_true',
        dest='include',
        help='Include end date in calculation (1 day is added)'
    )
    args = parser.parse_args()

    res = calc(args.start_date, args.end_date, args.include)
    print(f'The duration between {args.start_date} and {args.end_date} is {res} days.')
```

這邊來說明一下 `add_argument()` 裡面的參數：
- 第一個參數可以是參數名稱(`a name`)或是多個選項的 `flag`(`a list of option strings`)。
- `type`：CLI 傳進來的參數都是 `str`，在這邊可以設定傳進來後要轉換成 `int` 或是 `bool`等等。
- `dest`：當第一個是 `flag` 時，可以用 `dest` 這個參數將這個值指定成 `parser` 物件的屬性名稱(`attribute`)。
- `action`：預設是 `store`；在 `-i` 這個參數這邊，我把它設定成 `store_ture`，意思就是當有這個 `flag` 時，`include` 這個屬性為
 `True`，否則就是 `False`。

設定之後，我們就來用 `-h` 試試看新的 CLI 吧。

這邊可以清楚地看到哪些是 `positional arguments` 哪些是 `optional arguments`，以及 `help` 的說明文字。
```shell
(technical-note) Andrew-MacAir:argparse Andrew$ python date_diff_pos.py -h
usage: date_diff_pos.py [-h] [-i] start_date end_date

positional arguments:
  start_date     The start date
  end_date       The end date

optional arguments:
  -h, --help     show this help message and exit
  -i, --include  Include end date in calculation (1 day is added)
```

接著傳日期進去試試看
```shell
(technical-note) Andrew-MacAir:argparse Andrew$ python date_diff_pos.py 2019-01-01 2019-01-03
The duration between 2019-01-01 and 2019-01-03 is 2 days.
(technical-note) Andrew-MacAir:argparse Andrew$ python date_diff_pos.py 2019-01-01 2019-01-03 -i
The duration between 2019-01-01 and 2019-01-03 is 3 days.
```

第一個沒有 `-i`，所以兩個日期中間的間隔是 2 天；加上 `-i` 後，就是把第一天也加上去啦。

## 4. argparse-optional
由於前一小節用的是 `positional` 的方式，所以是依照傳入參數的位置來決定；那如果我很龜毛的想要指定參數呢？

前面 `calc()` function 的部分都是一樣的，我就省略了，主要是修改程式進入點的部分。
```python
# date_diff_opt.py
......
if __name__ == "__main__":
    parser = ArgumentParser()
    parser.add_argument(
        '-s', '--start_date', type=str,
        dest='start', required=True,
        help='The start date'
    )
    parser.add_argument(
        '-e', '--end_date', type=str,
        dest='end', required=True,
        help='The end date'
    )
    parser.add_argument(
        '-i', '--include', action='store_true',
        dest='include',
        help='Include end date in calculation (1 day is added)'
    )
    args = parser.parse_args()
    res = calc(args.start, args.end, args.include)
    print(f'The duration between {args.start} and {args.end} is {res} days.')
```

多的一個 `required`，顧名思義就是要你一定要傳啦。

這時候的 `--help` 就不太一樣囉，全部都是 `optional arguments`。

```shell
(technical-note) Andrew-MacAir:argparse Andrew$ python date_diff_opt.py -h
usage: date_diff_opt.py [-h] -s START -e END [-i]

optional arguments:
  -h, --help            show this help message and exit
  -s START, --start_date START
                        The start date
  -e END, --end_date END
                        The end date
  -i, --include         Include end date in calculation (1 day is added)
```

這樣的 CLI 是用 `flag` 的方式傳入參數，所以就算我把兩個日期對調位置也不會影響結果啦。
```shell
(technical-note) Andrew-MacAir:argparse Andrew$ python date_diff_opt.py -s 2019-01-01 -e 2019-01-03
The duration between 2019-01-01 and 2019-01-03 is 2 days.
(technical-note) Andrew-MacAir:argparse Andrew$ python date_diff_opt.py -e 2019-01-03 -s 2019-01-01 -i
The duration between 2019-01-01 and 2019-01-03 is 3 days.
```

---
完整的 sample code 請參考 [python-cli](https://github.com/orcahmlee/lab-technical-code/tree/master/Python/cli/argparse)。
