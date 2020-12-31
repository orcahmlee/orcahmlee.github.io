---
title: "Cython：加速你的 Python code"
layout: post
categories: Python
tags: cython
lang: en
---

Cython: Speeding up your Python code
===

Python 是一種直譯式的動態語言，它的優點是可讀性高、彈性大、開發快⋯⋯等很多優點啦(***This is why I like it!***)。但是，身為直譯式語言(Interpreted language)的一員，在執行速度上當然會比編譯式語言(Compiled language)慢一點啦。

試想一種情境，當你用 Python 快速開發出一個 prototype 後，卻發現執行速度有點慢，如果你希望它跑的和 C/C++ 一樣快的時候，應該怎麼做勒？

這時候 [Cython](https://cython.org/) 就可以幫你啦，Cython 可以讓你稍微修改原始的 Python code 後，就可以讓你從機車變成高鐵。

以下參考官方的教學 [Basic Tutorial](http://docs.cython.org/en/latest/src/tutorial/cython_tutorial.html#basic-tutorial)，紀錄使用過程。

OS Information:
由於編譯過程會與系統相依，所以這邊顯示我使用的系統資訊。

```shell
macOS Mojave Version 10.14.3
Python 3.6
```

## 1. Write a simple Python code
這邊先寫一個簡單的階乘 function，檔名叫做 `run_as_python.py`。
```python
def factorial(n):
    ans = 1
    for i in range(1, n + 1):
        ans *= i
    return ans
```

## 2. Modify your Python code 
然後再新增一個檔案，檔名叫做 `run_as_cython.pyx`，然後稍微修改前一個階乘 function。
```python
cpdef int factorial(int n):
    cdef int ans
    ans = 1
    cdef int i
    for i in range(1, n + 1):
        ans *= i
    return ans
```

所謂的稍微，也就是將原本的 Python code 裡面的物件都宣告成 C 的型態 ([Cython type reference](https://cython.readthedocs.io/en/latest/src/userguide/language_basics.html#automatic-type-conversions))。

這邊把 `ans` `i` `n` 都宣告成 `cdef int`，由於計算完的結果也是 `int`，所以在 `factorial()` 前面宣告 `cpdef int`。
`cdef` 和 `cpdef` 的差異可以 [參考](https://notes-on-cython.readthedocs.io/en/latest/function_declarations.html#cython-function-declarations)，如果要讓編譯後的 function 可以在 Python 裡面使用的話，就必須要將該 function 宣告成 `cpdef`。

## 3. Setup the setup.py
接著建立 `setup.py`，然後在裡面寫
```python
from distutils.core import setup
from Cython.Build import cythonize


setup(ext_modules = cythonize('run_as_cython.pyx'))
```

## 4. Compile
經過前面的步驟後，你的檔案結構應該會長這樣。
```shell
.
├── run_as_cython.pyx
├── run_as_python.py
└── setup.py
```

最後，在 terminal 裡面下 
```shell
python setup.py build_ext --inplace
```

如果編譯成功的話，你的檔案結構應該會變成這樣。
```shell
.
├── run_as_cython.c
├── run_as_cython.cpython-36m-darwin.so
├── run_as_cython.pyx
├── run_as_python.py
└── setup.py
```

如果你的系統是 Unix 會產生 `.so` 檔，如果是 Windows 則會產生 `.pyd`。

使用方式就只要簡單的 `import` 該檔案就好啦。

## 5. Try it
萬事俱備，直接在 Jupyter Notebook 中測試一下速度是不是真的從機車變成高鐵吧。

```python
import run_as_python
import run_as_cython

%%timeit
run_as_python.factorial(5)
# output
# 629 ns ± 32.2 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)

%%timeit
run_as_cython.factorial(5)
# output
# 79.4 ns ± 1.46 ns per loop (mean ± std. dev. of 7 runs, 10000000 loops each)
```

利用 Jupyter 的 `%%timeit` 測試，可以發現一樣的程式碼，速度提升接近 8 倍。

---
完整的 sample code 請參考 [cython-basic](https://github.com/orcahmlee/lab-technical-code/tree/master/Python/cython)。

如果你要直接下載執行的話，由於機器的不同，可能需要重新編譯。
