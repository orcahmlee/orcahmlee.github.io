---
title: "datetime 物件：Naive 與 Aware"
categories: Python
tags: datetime
lang: en
toc: true
---

The datetime objects: Naive and Aware
===

處理資料與分析資料的過程中，很常會遇到 `時間`，包括 `日期(Date)` 與 `時間(Time)`，這邊紀錄 Python 的 [datetime](https://docs.python.org/3.6/library/datetime.html) 使用方式。

---
> There are two kinds of date and time objects: “naive” and “aware”.
>
> An aware object has sufficient knowledge of applicable algorithmic and political time adjustments, such as time zone and daylight saving time information, to locate itself relative to other aware objects. An aware object is used to represent a specific moment in time that is not open to interpretation.
>
> A naive object does not contain enough information to unambiguously locate itself relative to other date/time objects. Whether a naive object represents Coordinated Universal Time (UTC), local time, or time in some other timezone is purely up to the program, just like it is up to the program whether a particular number represents metres, miles, or mass. Naive objects are easy to understand and to work with, at the cost of ignoring some aspects of reality.

官方文件一開頭就點到 `datetime` 物件有兩個種類 `Aware` 與 `Naive` 。簡單來說，`Aware` 物件包含足夠的資訊可以讓使用者一目了然，不需要再額外的解釋，如時區、日光節約時間等；而 `Naive` 則是單純的呈現日期與時間而不包含時區的資訊，時區取決於你所執行程式的機器設定，`Naive` 物件雖然簡單易懂，但所換取的代價是忽略了一些實際的細節。

## Naive
直接使用 `datetime.datetime.now()` 可以回傳一個 `datetime.datetime` 物件，其中包含當下的 `年`、`月`、`日`、`時`、`分`、`秒` 及 `毫秒` 等資訊；另外 `datetime.datetime.utcnow()` 則會回傳當下 `UTC` 時間的 `年`、`月`、`日`、`時`、`分`、`秒` 及 `毫秒` 等資訊，你可以檢查這兩個物件的 `tzinfo`，可以發現都是回傳 `None`。雖然 `Naive` 物件的使用方式簡單，但是卻需要額外的解釋或文件告知使用者(或是未來的健忘的自己:sweat_smile:)，你使用的是 `UTC` 還是當地時區。

```python
now = datetime.now()
now_utc = datetime.utcnow()

print(now.tzinfo)
print(now_utc.tzinfo)
```

## Aware
若你想在獲得 `datetime` 物件時，同時知道 `timezone` 的話，你可以在使用 `datetime.datetime.now(tz=None)` 時，傳入 `tz` 的資訊，在這邊你可以使用 `datetime` Library 中的 `datetime.timzone` Class。

```python
now_utc_tz = datetime.now(tz=timezone.utc)
now_taiwan_tz = datetime.now(tz=timezone(timedelta(hours=8)))
now_la_tz = datetime.now(tz=timezone(-timedelta(hours=7)))

print(now_utc_tz.tzinfo)
print(now_taiwan_tz.tzinfo)
print(now_la_tz.tzinfo)
```

### Aware - pytz
上面其中的一個例子是 Los Angeles，而當下剛好是日光節約時間，所以是 `UTC -07:00`，問題是我哪記得住何時是正常時間，何時又是日光節約時間呢！？

在這邊就使用 [pytz](https://pythonhosted.org/pytz/) 來迅速又清楚地設定 `timezone` 吧！

```python
now_utc_pytz = datetime.now(tz=pytz.UTC)
now_taiwan_pytz = datetime.now(tz=pytz.timezone('Asia/Taipei'))
now_la_pytz = datetime.now(tz=pytz.timezone('America/Los_Angeles'))

print(now_utc_pytz.tzinfo)
print(now_taiwan_pytz.tzinfo)
print(now_la_pytz.tzinfo)
```

---
完整的 sample code 請參考 [01-naive-and-aware](https://github.com/orcahmlee/lab-technical-code/blob/master/Python/datetime/01-naive-and-aware.ipynb)