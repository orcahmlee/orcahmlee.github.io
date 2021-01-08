---
title: "利用 conda 快速建立資料科學的 Python 環境"
categories: Data-Science
tags: python conda
lang: en
toc: true
---

The easiest way to create Python environment for Data Science using conda
===

## Introduction
建立環境是一件很重要卻很煩人的事情，有經驗的開發者會利用 [Homebrew](https://brew.sh/)、[yum](https://en.wikipedia.org/wiki/Yum_(software))、[APT](https://en.wikipedia.org/wiki/APT_(Package_Manager)) 等套件管理工具安裝 [Python](https://www.python.org/about/)，當然也可以到 Python 官網直接下載 [GUI 的安裝檔](https://www.python.org/downloads/)，最後再確定環境變數有正確的設定。

安裝好 Python 後，為了不污染系統本身的 Python2，一般來說都會利用 [venv](https://docs.python.org/3/library/venv.html#module-venv)、[virtualenv](https://pypi.org/project/virtualenv/) 或 [pipenv](https://pypi.org/project/pipenv/) 來建立 Python 的虛擬環境以達到彼此隔離的效果。

當 Python 安裝完成，Python 虛擬環境也搞定之後，就代表後面無事一身輕了嗎？~~代誌不是憨人所想的那麼簡單啦~~；哈哈哈，錯了:flushed:！試想一個情境，當今天你正在使用 Python 3.6 進行系統開發、資料科學的分析或模型的建置，然後同時有一個舊專案必須使用 Python 3.4 維護，再然後你想要很潮的測試最新的 Python 3.7。

嗯⋯⋯，你當然可以安裝很多版本的 Python，同時，你要很清楚自己的機器的環境變數，然後再用類似 `brew switch python 3.x.x` 的方式進行切換。如果像我這樣對環境變數不是很熟，而且需要在不同平台上面跑資料科學專案時不就很煩:weary:。

所以下面就介紹我安裝與使用 [conda](https://docs.conda.io/projects/conda/en/latest/) 的過程。

> ***Package, dependency and environment management for any language---Python, R, Ruby, Lua, Scala, Java, JavaScript, C/ C++, FORTRAN***

在介紹頁中，`conda` 直接的跟你說它有多方便，`conda` 同時是套件管理、相依性管理與環境管理的工具，除了可以跨平台的運行在 Windows、macOS、Linux，也可以輕鬆的切換不同的 Python 版本。`conda` 已經直接包含在 [Anaconda](https://www.anaconda.com/distribution/) 與 [Miniconda](https://docs.conda.io/en/latest/miniconda.html) 中，所以不管安裝哪一種都可以。


Anaconda 在安裝的過程中，會一次幫你下載並安裝 720 以上資料科學常用的套件，安裝完成後就可以直接使用，適合初學者。Miniconda 則是 Anaconda 的最小化版本，使用者安裝後再依照自身需求安裝套件，適合硬碟容量小的使用者。

> - For Miniconda---400 MB disk space.
> - For Anaconda---Minimum 3 GB disk space to download and install.

由於安裝過程大同小異，所以後續使用 Miniconda 做為示範。

---
## Installation on CentOS7
##### OS information:
```shell
# cat /etc/centos-release
CentOS Linux release 7.4.1708 (Core)
```

##### Login your account
```
sudo -i -u <user name>
```

##### Install Miniconda
[Miniconda](https://docs.conda.io/en/latest/miniconda.html)<br>
下載最新的 `bash installer`。

[Miniconda installer archive](https://repo.anaconda.com/miniconda/)<br>
或是歷史資料庫選擇你想要的版本。

使用 `wget` 下載 Linux 版本的 `bash installer`。
```shell
cd /tmp
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

[Miniconda installer archive](https://repo.anaconda.com/miniconda/) 中有提供各檔案的 MD5 雜湊值(hash value) 以確保下載的檔案是正確的。

```shell
$ md5sum Miniconda3-latest-Linux-x86_64.sh
718259965f234088d785cad1fbd7de03 Miniconda3-latest-Linux-x86_64.sh
```

執行該安裝檔

```shell
bash Miniconda3-latest-Linux-x86_64.sh
```

安裝完成後，預設是在登入時或開啟新 terminal 的時候自動啟動 `base` 這個環境。如果你不喜歡可以用以下的方法修改設定值。

```shell
conda config --set auto_activate_base false
```

> If you'd prefer that conda's `base` environment not be activated on startup, set the auto_activate_base parameter to false:

## Installation on macOS
##### OS information:
```shell
macOS Mojave Version 10.14.3
```

##### Install Miniconda
[Miniconda](https://docs.conda.io/en/latest/miniconda.html)<br>
依照平台下載最新的 GUI 安裝檔或是 `bash` 安裝檔，這邊我直接下載 `.pkg` ，然後無腦地一鍵安裝。

[Miniconda installer archive](https://repo.anaconda.com/miniconda/)<br>
或是歷史資料庫選擇你想要的版本。

相同地，安裝完成後，如果你不喜歡自動啟動 `base` 這個環境，你可以用以下的方式修改設定值。

```shell
conda config --set auto_activate_base false
```

安裝完成後，在不同平台的使用方式都是一樣的，所以下面我使用 masOS 做為紀錄。

---
## Managing environments
首先先檢查 `conda` 的路徑與版本。
```shell
Andrew-MacAir:~ Andrew$ which conda
/Users/Andrew/miniconda3/condabin/conda

Andrew-MacAir:~ Andrew$ conda --version
conda 4.6.14
```

`conda info`  這個指令可以檢查相關資訊，這邊可以看到我的 Python 版本、user config file 的路徑⋯⋯等等。
```shell
Andrew-MacAir:~ Andrew$ conda info

     active environment : None
            shell level : 0
       user config file : /Users/Andrew/.condarc
 populated config files : /Users/Andrew/.condarc
          conda version : 4.6.14
    conda-build version : not installed
         python version : 3.7.3.final.0
       base environment : /Users/Andrew/miniconda3  (writable)
           channel URLs : https://repo.anaconda.com/pkgs/main/osx-64
                          https://repo.anaconda.com/pkgs/main/noarch
                          https://repo.anaconda.com/pkgs/free/osx-64
                          https://repo.anaconda.com/pkgs/free/noarch
                          https://repo.anaconda.com/pkgs/r/osx-64
                          https://repo.anaconda.com/pkgs/r/noarch
          package cache : /Users/Andrew/miniconda3/pkgs
                          /Users/Andrew/.conda/pkgs
       envs directories : /Users/Andrew/miniconda3/envs
                          /Users/Andrew/.conda/envs
               platform : osx-64
             user-agent : conda/4.6.14 requests/2.21.0 CPython/3.7.3 Darwin/18.5.0 OSX/10.14.4
                UID:GID : 501:20
             netrc file : None
           offline mode : False
```

### 1. Checking the environments
`conda info --env` 或 `conda env list` 這兩個指令都可以列出機器裡面目前所有擁有的環境，預設安裝完後就會有一個 `base` 的環境。

```shell
Andrew-MacAir:~ Andrew$ conda env list
# conda environments:
#
base                  *  /Users/Andrew/miniconda3
```

### 2. Activating/Deactivating the environments
擁有環境之後，首先要知道如何 `進入/離開` conda 環境；
`conda activate <the name of environment>` 可以啟動你所需要的環境。
```shell
Andrew-MacAir:~ Andrew$ conda activate base
(base) Andrew-MacAir:~ Andrew$
```

這時候你會看到使用者名稱前面會多一個 `(base)`，這就表示你正在使用 `base` 這個 conda 環境。然後檢查 Python 的路徑與版本，確定是使用 `miniconda3` 中的 Python 即可。

```shell
(base) Andrew-MacAir:~ Andrew$ which python
/Users/Andrew/miniconda3/bin/python
(base) Andrew-MacAir:~ Andrew$ python --version
Python 3.7.3
```

成功進入環境之後，`conda deactivate` 可以離開目前所在的 conda 環境。
```shell
(base) Andrew-MacAir:~ Andrew$ conda deactivate
Andrew-MacAir:~ Andrew$ 
```

### 3. Creating the environments
接著就開始建立自己所需要的 Python 環境啦！<br>

##### 3.1 從無到有開始
`conda create -n <env name> python=x.x` 這指令可以建立自己喜歡的環境名稱，並指定 Python 版本。

```shell
Andrew-MacAir:~ Andrew$ conda create -n technical-note python=3.6
Collecting package metadata: done
Solving environment: done

## Package Plan ##

  environment location: /Users/Andrew/miniconda3/envs/technical-note

  added / updated specs:
    - python=3.6


The following NEW packages will be INSTALLED:

  ca-certificates    pkgs/main/osx-64::ca-certificates-2019.5.15-0
  certifi            pkgs/main/osx-64::certifi-2019.3.9-py36_0
  libcxx             pkgs/main/osx-64::libcxx-4.0.1-hcfea43d_1
  libcxxabi          pkgs/main/osx-64::libcxxabi-4.0.1-hcfea43d_1
  libedit            pkgs/main/osx-64::libedit-3.1.20181209-hb402a30_0
  libffi             pkgs/main/osx-64::libffi-3.2.1-h475c297_4
  ncurses            pkgs/main/osx-64::ncurses-6.1-h0a44026_1
  openssl            pkgs/main/osx-64::openssl-1.1.1c-h1de35cc_1
  pip                pkgs/main/osx-64::pip-19.1.1-py36_0
  python             pkgs/main/osx-64::python-3.6.8-haf84260_0
  readline           pkgs/main/osx-64::readline-7.0-h1de35cc_5
  setuptools         pkgs/main/osx-64::setuptools-41.0.1-py36_0
  sqlite             pkgs/main/osx-64::sqlite-3.28.0-ha441bb4_0
  tk                 pkgs/main/osx-64::tk-8.6.8-ha441bb4_0
  wheel              pkgs/main/osx-64::wheel-0.33.4-py36_0
  xz                 pkgs/main/osx-64::xz-5.2.4-h1de35cc_4
  zlib               pkgs/main/osx-64::zlib-1.2.11-h1de35cc_3


Proceed ([y]/n)? 
```

過程中，`conda` 會檢查環境，並詢問你要不要安裝相關的套件，選擇 `y` 後就會開始安裝程序。安裝完成後，`conda` 會提示你如何 `進入/離開` 你剛剛所建立的環境(你剛剛已經會了:stuck_out_tongue:)

```shell
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use
#
#     $ conda activate technical-note
#
# To deactivate an active environment, use
#
#     $ conda deactivate
```

在進入環境之前，先用 `conda env list` 來觀察一下你的環境清單，接著就可以開始隨心所欲地使用你的新環境啦。

```shell
Andrew-MacAir:~ Andrew$ conda env list
# conda environments:
#
base                  *  /Users/Andrew/miniconda3
technical-note           /Users/Andrew/miniconda3/envs/technical-note
```

##### 3.2 利用事先定義好的 yml 檔
`conda env create -f <yml name>` 這指令可以依照檔案內容來建立正確的環境。

```yml
name: technical-note
channels:
  - defaults
  - conda-forge
  - anaconda
dependencies:
  - python=3.6
  - pandas=0.24.2
  - scikit-learn
```

在這個檔案中，你可以指定環境名稱(`name`)、下載的來源(`channels`)以及套件相關資訊(`dependencies`)，在套件資訊裡面你可以指定版本號，或是不指定版本讓 conda 自行尋找符合當下環境且不會衝突的版本。

利用這個方法除了可以方便地控管你的環境外，還可以分享給你的同事，以及在部署到 `production` 環境時，確保大家都是使用相同的版本。

當然，在開發的過程中，有可能會一直新增或刪減套件，當 yml 檔修正後，可以利用 `conda env update -f <yml name>` 指令來更新你的環境。

### 4. Removing the environments
##### 4.1 在當下的環境中，移除某一個 package
`conda remove <package name>`

```shell
(technical-note) Andrew-MacAir:lab-python Andrew$ conda remove numpy
Collecting package metadata: done
Solving environment: done

## Package Plan ##

  environment location: /Users/Andrew/miniconda3/envs/technical-note

  removed specs:
    - numpy


The following packages will be REMOVED:

  mkl_fft-1.0.12-py36h5e564d8_0
  numpy-1.16.4-py36hacdab7b_0


Proceed ([y]/n)?
```

##### 4.2 在不進入環境的狀況下，移除某一個環境中的某一個 package
`conda remove -n <env name> <package name>`

```shell
Andrew-MacAir:lab-python Andrew$ conda remove -n technical-note numpy
Collecting package metadata: done
Solving environment: done

## Package Plan ##

  environment location: /Users/Andrew/miniconda3/envs/technical-note

  removed specs:
    - numpy


The following packages will be REMOVED:

  mkl_fft-1.0.12-py36h5e564d8_0
  numpy-1.16.4-py36hacdab7b_0


Proceed ([y]/n)? 
```

##### 4.3 最後，當你不想要某一個環境，或是某一個環境被玩爛的時候，你可以把整個環境移除
`conda remove -n <env name> --all` 或是 `conda env remove -n <env name>`

```shell
Andrew-MacAir:~ Andrew$ conda remove -n technical-note --all

Remove all packages in environment /Users/Andrew/miniconda3/envs/technical-note:


## Package Plan ##

  environment location: /Users/Andrew/miniconda3/envs/technical-note


The following packages will be REMOVED:

  ca-certificates-2019.5.15-0
  certifi-2019.3.9-py36_0
  libcxx-4.0.1-hcfea43d_1
  libcxxabi-4.0.1-hcfea43d_1
  libedit-3.1.20181209-hb402a30_0
  libffi-3.2.1-h475c297_4
  ncurses-6.1-h0a44026_1
  openssl-1.1.1c-h1de35cc_1
  pip-19.1.1-py36_0
  python-3.6.8-haf84260_0
  readline-7.0-h1de35cc_5
  setuptools-41.0.1-py36_0
  sqlite-3.28.0-ha441bb4_0
  tk-8.6.8-ha441bb4_0
  wheel-0.33.4-py36_0
  xz-5.2.4-h1de35cc_4
  zlib-1.2.11-h1de35cc_3


Proceed ([y]/n)? y

Preparing transaction: done
Verifying transaction: done
Executing transaction: done
```

```shell
Andrew-MacAir:~ Andrew$ conda env list
# conda environments:
#
base                  *  /Users/Andrew/miniconda3
```
