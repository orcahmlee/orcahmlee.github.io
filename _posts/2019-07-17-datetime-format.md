---
title: "datetime 物件：strftime() and strptime()"
categories: Python
tags: datetime
lang: en
toc: true
---

The datetime objects: strftime() and strptime()
===

從上一篇 [datetime 物件：Naive 與 Aware]({{ site.baseurl }}{% post_url 2019-06-30-datetime-naive-and-aware %}) 知道如何建立 `datetime` 物件後。

```python
datetime.now()
```
```output
datetime.datetime(2019, 7, 17, 11, 41, 40, 175848)
```

```python
datetime.now(tz=pytz.timezone('Asia/Taipei'))
```
```output
datetime.datetime(2019, 7, 17, 11, 41, 40, 202076, tzinfo=<DstTzInfo 'Asia/Taipei' CST+8:00:00 STD>)
```

接著要把它轉換成所需要的字串格式，官方文件有詳細的說明 [strftime() and strptime() Behavior](https://docs.python.org/3.6/library/datetime.html#strftime-strptime-behavior)，下面紀錄一些常用的方式。

## Basic
當建立一個 `datetime` 物件後，你就可以隨心所欲地取得該物件的 `attributes`，例如年、月、日、時、分、秒、微秒等等。
```python
now_taiwan_pytz = datetime.now(tz=pytz.timezone('Asia/Taipei'))
print('Year: {}'.format(now_taiwan_pytz.year))
print('Month: {}'.format(now_taiwan_pytz.month))
print('Day: {}'.format(now_taiwan_pytz.day))
print('Hour: {}'.format(now_taiwan_pytz.hour))
print('Minute: {}'.format(now_taiwan_pytz.minute))
print('Second: {}'.format(now_taiwan_pytz.second))
print('MicroSecond: {}'.format(now_taiwan_pytz.microsecond))
```

如果是要取得 `week number` 和 `weekday` 的資訊時，則可以使用該物件的 `method`，`datetime.datetime.isocalendar()` 會回傳一個含有 (ISO year, ISO week number, ISO weekday) 的 `tuple`，接著在個別 slicing 出來即可。
```python
print(now_taiwan_pytz.isocalendar())
print('ISO Year: {}'.format(now_taiwan_pytz.isocalendar()[0]))
print('ISO Week Number: {}'.format(now_taiwan_pytz.isocalendar()[1]))
print('ISO Weekday: {}'.format(now_taiwan_pytz.isocalendar()[2]))
```

這邊要注意的是，weekday 和斯斯一樣有兩種：
- [`datetime.datetime.weekday()`](https://docs.python.org/3/library/datetime.html#datetime.datetime.weekday)：Monday is 0 and Sunday is 6.
- [`datetime.datetime.isoweekday()`](https://docs.python.org/3/library/datetime.html#datetime.datetime.isoweekday)：Monday is 1 and Sunday is 7.

最後就是時間戳記(`timestamp`)，它有超多名字 `Unix time`、`UNIX epoch` 、`Unix timestamp`、`POSIX time`、`POSIX timestamp`，總之它就是定義從 `1970-01-01 00:00:00+00:00` 開始的累積秒數，你可以使用 `timestamp()` 獲得。

```python
print('Timestamp: {}'.format(now_taiwan_pytz.timestamp()))
```

## strftime()
> get the string from a datetime object

首先可以用 [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) 的格式，
```python
now_taiwan_pytz.isoformat()
```

如果你不想用 `T` 作為 `日期` 與 `時間` 的分隔時，你還可以指定分隔字串，注意字串長度只能為 1。
```python
now_taiwan_pytz.isoformat(sep=' ')
```

如果你想要自訂義格式的話，你也可以利用 `strftime()` 自由組成需要的格式，例如這個噁心的格式。
```python
now_taiwan_pytz.strftime('%Y/%m/%d 我是分隔 %H:%M:%S 我是分隔 %A 我是分隔 %z')
```

這邊還是要注意的是，weekday 又有兩種:confused:：
- `%w`: 0 is Sunday and 6 is Saturday.
- `%u`: 1 isMonday and 7 isSunday.

## strptime()
> string parse to a new datetime object

除了用 `datetime.datetime.now()` 來建立 `datetime` 物件，你還可以用 `timestamp` 及字串來建立。
```python
ts = 1563334900.217434
datetime.fromtimestamp(ts)

datetime.strptime('2019-07-17T11:30:41.918288', '%Y-%m-%dT%H:%M:%S.%f')
```

---
完整的 sample code 請參考 [02-datetime-format](https://github.com/orcahmlee/lab-technical-code/blob/master/Python/datetime/02-datetime-format.ipynb)
