---
title: "視覺化：雷達圖"
layout: post
categories: Data-Science
tags: python matplotlib radar
lang: en
---

Data Visualization: Radar chart
===

雷達圖適合拿來說明一個個體的組成，例如一個學生的各科分數、一個籃球員的各項表現、一個食物的各種成分比例⋯⋯等等，資料必須為數值型(Numeric)且不適合太多項，一般來說，五、六項差不多可以讓人類一眼分辨又不會覺得眼花撩亂。

常見的 Python 套件並沒有可以直接畫出 `method`，除了 [Plotly](https://plot.ly/python/radar-chart/) 外；雖然在 [matplotlib](https://matplotlib.org/examples/api/radar_chart.html) 裡面有示範如何畫出雷達圖，但是實在是有點長⋯⋯。

所以，以下紀錄手刻的過程。

首先，取得資料或是建立 toy data。
```python
# Set data
df = pd.DataFrame(
    data = {
        'Student': ['A', 'B', 'C', 'D'],
        'Math': np.random.randint(60, 100, 4),
        'Physics': np.random.randint(60, 100, 4),
        'Chemistry': np.random.randint(60, 100, 4),
        'History': np.random.randint(60, 100, 4),
        'Geography': np.random.randint(60, 100, 4)
    }
)
```

取得雷達圖上要標的名稱，這邊是 `各科名稱`。
```python
labels = df.columns.to_numpy()[1:]  # get the labels for radar
```

取得雷達圖上要標的數值，這邊只取某一個學生的 `各科分數`。
```python
stats = df.loc[df.Student == 'A', labels].values[0]  # get the values for radar
```

根據科目數量，計算每個科目之間的夾角；注意，這邊的角度是 ***弧度(Radian)***。
```python
angles = np.linspace(0, 2 * np.pi, len(labels), endpoint=False)  # get the angles for each group
```

接著把分數和夾角這兩個 `array` 的頭尾接起來，如果沒接的話，最後畫成雷達圖的時候會破口。
```python
# close the plot
stats = np.concatenate((stats, [stats[0]]))
angles = np.concatenate((angles, [angles[0]]))
```

資料準備好之後就可以來畫圖啦，這邊可以用兩個方式來產生 `class matplotlib.axes.Axes`：
1. `matplotlib.pyplot.figure().add_subplot()`:
```python
# method1: Figure.add_subplot
fig = plt.figure()
ax = fig.add_subplot(111, polar=True)
```

2. `matplotlib.pyplot.subplot()`:
```python
# method2: plt.subplot
ax = plt.subplot(111, polar=True)
```

不管選用哪個方法，重要的是在 `polar` 這個參數要設定為 `True`，這樣畫出來的圖才會是圓形。
```python
ax = plt.subplot(1, 1, 1, polar=True)  # subplot the polar plot
```

有了 `Axes` 物件之後，就可以把要畫的東西一個一個加進去，**這邊建議加一行 code 就看一次圖，就會知道每一行 code 所代表的功能為何**。

第一步，先畫線，`x` 放角度(Radian)，`y` 放各科分數。
```python
ax.plot(angles, stats, 'o-', linewidth=2)  # plot: x=angles, y=stats
ax.set_ylim(top=100)
```
![radar-chart-1.png](/assets/images/2019-08-07-radar-chart/radar-chart-1.png)

上一步畫完後，雷達圖的雛形就出來了，但是你會發現外圍的 `labels` 是角度(Degrees)，而不是原本想要看到的 `各科名稱`；<br>
所以這邊要使用 [`set_thetagrids()`](https://matplotlib.org/3.1.0/api/_as_gen/matplotlib.pyplot.thetagrids.html#matplotlib.pyplot.thetagrids) 把 `各科名稱` 依照它應該在的角度放上去，***注意*** 這裡的角度是 ***Degrees***。

$$ 2 * \pi = 360^o $$

$$ rad = deg * \frac{\pi}{180^o} $$

$$ deg = rad * \frac{180^o}{\pi} $$

```python
ax.set_thetagrids(angles * 180 / np.pi, labels)  # set the labels by the angles
```
![radar-chart-2.png](/assets/images/2019-08-07-radar-chart/radar-chart-2.png)

基本上雷達圖就已經完成了，接著就是為了美觀在圍起來的區域塗滿透明色。
```python
ax.fill(angles, stats, alpha=0.25)  # fill the polygon
```
![radar-chart-3.png](/assets/images/2019-08-07-radar-chart/radar-chart-3.png)

最後就是補上說明文字，例如 `title`、`legend`、`ticks`⋯⋯等等。
```python
ax.set_title('Student A')
```
![radar-chart-4.png](/assets/images/2019-08-07-radar-chart/radar-chart-4.png)

---
其他的畫法，例如把多個學生畫在同一個雷達圖上、把多個雷達畫在同一張圖上，都是基於上述的方法做一些修改。<br>
完整的 sample code 請參考 [radar-charts](https://github.com/orcahmlee/lab-technical-note/blob/master/plotting/radar-chart/radar-charts.ipynb)
