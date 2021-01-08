---
title: "如何漂亮地格式化你的字串"
categories: Python
tags: string
lang: en
toc: true
---

How to format your string?
===

在這篇 [datetime 物件：strftime() and strptime()]({{ site.baseurl }}{% post_url 2019-07-17-datetime-format %}) 中有看到一些格式化字串的方法。

如果我想要產生一段如下的文字，而且文字要根據物件的內容動態的改變，
> Hi there, I am Andrew and I'm from Taiwan!<br>
> The coordinates of Taiwan are 23.6978 N, 120.9605 E.<br>
> The population of Taiwan is approximately 23 million.

下面紀錄一些常用的方式。

首先，先定義物件內容
```python
name = 'Andrew'
nationality = 'Taiwan'
lat = 23.6978
lon = 120.9605
population = 23.57

info = {
    'name': 'Andrew',
    'nationality': 'Taiwan'
}
coordinates = {
    'nationality': 'Taiwan',
    'lat': 23.6978,
    'lon': 120.9605
}
```

## C-style
[printf-style String Formatting](https://docs.python.org/3/library/stdtypes.html#printf-style-string-formatting) 類似 C 語言的 `printf()` 使用 `%` 的符號作為字串插入位置，例如：
```python
>>> print("Hi there, I am %s and I'm from %s!" % (name, nationality))
>>> print('The coordinates of %s are %.4f N, %.4f E.' % (nationality, lat, lon))
>>> print('The population of %s is approximately %d million.' % (nationality, population))

Hi there, I am Andrew and I'm from Taiwan!
The coordinates of Taiwan are 23.6978 N, 120.9605 E.
The population of Taiwan is approximately 23 million.
```

引號內的 `%s` 代表的就是待插入的字串位置，而引號後的 `tuple`代表要插入的物件，兩者中間用 `%` 做為分隔。除了 `%s` 以外，還可以用 `%d` 代表整數(Integer)；`%.4f` 代表浮點數(Float)，其中 `.f` 代表小數點後四位數。

如果嫌 `tuple` 內要放很多物件麻煩的話，你還可以把字串整理成 `dict` 物件，然後再放到 `%` 的後面去，例如：
```python
>>> print("Hi there, I am %(name)s and I'm from %(nationality)s!" % info)
>>> print('The coordinates of %(nationality)s are %(lat).4f N, %(lon).4f E.' % (coordinates))
>>> print('The population of %s is approximately %d million.' % (nationality, population))
```

引號中的 `%(name)s`、`%(nationality)s` 就是 `info` 物件中的 `key`(`name`、`nationality`)。有沒有發現還是麻煩，因為你只是把物件名稱換到字串裡面而已:neutral_face:。

這樣的格式化方式其實並不好用，而且也容易造成程式中的臭蟲:bug:，就連官方也建議你不要用這個方式。

>Note The formatting operations described here exhibit a variety of quirks that lead to a number of common errors (such as failing to display tuples and dictionaries correctly). Using the newer formatted string literals, the str.format() interface, or template strings may help avoid these errors. Each of these alternatives provides their own trade-offs and benefits of simplicity, flexibility, and/or extensibility.

## str.format()
我們來看看官方建議的其中一種方式 [str.format()](https://docs.python.org/3/library/stdtypes.html#str.format)，

```python
>>> print("Hi there, I am {} and I'm from {}!".format(name, nationality))
>>> print('The coordinates of {} are {} N, {} E.'.format(nationality, lat, lon))
>>> print('The population of {} is approximately {} million.'.format(nationality, population))

Hi there, I am Andrew and I'm from Taiwan!
The coordinates of Taiwan are 23.6978 N, 120.9605 E.
The population of Taiwan is approximately 23.57 million.
```

引號中的字串插入位置換成 `{}`，而物件內容則是依照 `.format()` 中的參數決定；
- 如果 `{}` 中沒有參數位置時，就依照參數的位置順序。
- 若是 `{0}` 則代表第 1 個參數，`{1}` 代表第 2 個參數，以此類推。
- 而 `{}` 中的 `:.2f` 表示該字串要呈現小數點的位數。

```python
>>> print("Hi there, I am {0} and I'm from {1}!".format(name, nationality))
>>> print('The coordinates of {0} are {1:.2f} N, {2:.2f} E.'.format(nationality, lat, lon))
>>> print('The population of {0} is approximately {1:.0f} million.'.format(nationality, population))

Hi there, I am Andrew and I'm from Taiwan!
The coordinates of Taiwan are 23.70 N, 120.96 E.
The population of Taiwan is approximately 24 million.
```

除了指定參數位置外，`str.format()` 還可以指定要插入的物件名稱，例如：
```python
print("Hi there, I am {name} and I'm from {nationality}!".format(name=name, nationality=nationality))
print("Hi there, I am {name} and I'm from {nationality}!".format(name=info['name'], nationality=info['nationality']))
```

你還可以直接利用 `**kwargs` 的方式，將 `dict` 直接傳入 `.format()` 裡面
```python
print("Hi there, I am {name} and I'm from {nationality}!".format(**info))
```

## f-strings
[PEP 498: Formatted string literals](https://docs.python.org/3/whatsnew/3.6.html#whatsnew36-pep498) 是在 version 3.6 的時候被加進來的新功能。`f-strings` 是在字串前面加入 `f` 或 `F` 前綴字，字串插入位置仍然是用 `{}`，使用方法和 `str.format()` 很相似，例如：
```python
>>> print(f"Hi there, I am {name} and I'm from {nationality}!")
>>> print(f'The coordinates of {nationality} are {lat:.4f} N, {lon:.4f} E.')
>>> print(f'The population of {nationality} is approximately {population:.0f} million.')

Hi there, I am Andrew and I'm from Taiwan!
The coordinates of Taiwan are 23.6978 N, 120.9605 E.
The population of Taiwan is approximately 24 million.
```

也可以直接使用 `dict` 物件
```python
>>> print(f"Hi there, I am {info['name']} and I'm from {info['nationality']}!")
Hi there, I am Andrew and I'm from Taiwan!
```

最妙的是 `f-strings` 還可以在字串中進行運算與呼叫 `method`
```python
>>> print(f'{2 ** 10}')
1024

>>> print(f"I'm from {nationality.upper()}!")
I'm from TAIWAN!
```

---
完整的 sample code 請參考 [string-formatting](https://github.com/orcahmlee/lab-technical-code/blob/master/Python/string/string-formatting.ipynb)
