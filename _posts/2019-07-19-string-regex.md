---
title: "Python 正規表示式"
layout: post
categories: Python
tags: string regex
lang: en
---

How to use regex in Python?
===

正規表示式(Regular expression)的水實在是太深，以下單純紀錄在 Python 使用過的方式。

|Special character|Description|
|---|---|
|^     |Start with|
|$     |End with|
|\|    |Either or|
|.     |Match any character(except a newline)|
|X*    |Match zero times or more than once|
|X+    |Match once or more than once(at least once)|
|X?    |Match either zero times or once|
|X{n}  |Exactly reapt n times|
|X{m,n}|Exactly reapt at least m times but no more than n times|
|X{0,} |Equals to X*|
|X{1,} |Equals to X+|
|X{0,1}|Equals to X?|
|[]    |Used to indicate a set of characters|
|()    |Capture and group|

上面這張表是正規表示式所使用符號及代表的意義，例如：
- `^Python`: 開頭為 `Python` 的字串。
- `^Python$`: 完全符合 `Python` 這樣的字串，因為頭尾各用 `^`、`$` 包起來。
- `p+`: `p` 至少出現過一次。
- `p*`: `p` 出現過一次以上，或是沒有出現過。
- `[abc]`: 只要方括號內的文字個別出現都算，如：`a`、`b`、`c` 。
- `(abc)`: 圓括號內的文字作為一個群組，整個群組都符合才算。

|Special squence|Description|Equals to|
|---|---|---|
|\w|Matches any word character|[a-zA-Z0-9_]|
|\W|Matches any character which is NOT word character|[^\w]|
|\d|Matches any decimal digit|[0-9]|
|\D|Matches any character which is NOT decimal digit|[^\d]|
|\s|Matches any whitespace character|[ \t\n\r\f\v]|
|\S|Matches any character which is NOT whitespace character|[^\s]|

上面這張表則是常用的縮寫代號。

## re.findall()
[re.findall()](https://docs.python.org/3/library/re.html#re.findall)

## re.search()
[re.search()](https://docs.python.org/3/library/re.html#re.search)

## re.match()
[re.match()](https://docs.python.org/3/library/re.html#re.match)