---
title: "DevOps：如何將 Nginx + uWSGI + Django 的服務部署在一台主機的根目錄"
layout: post
categories: DevOps
tags: nginx uwsgi django
lang: en
---

How to deploy the Django application at the root on a host machine using Nginx + uWSGI + Django
===

部署的設定千百種，坑也是千百種，所以趁著還有記憶的時候趕快紀錄一下。完整的 sample code 請參考 [nginx-uwsgi-django-depoly-at-root](https://github.com/orcahmlee/lab-technical-code/tree/master/DevOps/nginx-uwsgi-django/depoly-at-root)。


以下的設定皆是在 `Mac` 上進行測試，若是在 `CentOS 7` 或是 `Ubuntu` 上的話，`Nginx` 的部分可能需要微調。

OS Information and packages version

```shell
macOS Catalina, Version 10.15.6
conda 4.8.5
django 3.1.2
uwsgi 2.0.19.1
nginx version: nginx/1.19.3
```

---
## 1. Build a simple Django project

本文不對 `Django` 做太多說明，詳細的教學與設定可以參考官方文件 [Get started with Django](https://www.djangoproject.com/start/)。如果懶得看的話，照著以下的步驟就可以建立出一個簡易的 Django Application。

建立專案並移動到專案目錄下。

```shell
django-admin startproject dj3

cd dj3
```

在 `dj3` 專案中建立第一個名為 `api` 的 `app`，並且在這個 `api` 中新增一個 `urls.py` 的檔案。`project` 與 `app` 的差異請參考 [官方文件](https://docs.djangoproject.com/en/3.1/intro/tutorial01/#creating-the-polls-app)。

```shell
django-admin startapp api

touch api/urls.py
```

將剛剛建立的名為 `api` 的 `app` 註冊到 `dj3` 這個 `project` 中，將 `dj3/settings.py` 的檔案修改如下，

```python
# dj3/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'api',
]
```

並且在最下方新增 [`STATIC_ROOT` ](https://docs.djangoproject.com/en/3.1/ref/settings/#static-root) 的路徑。

這個路徑是部署時將靜態檔案收集到同一個路徑下，方便 `Nginx` 讀取用。

```python
# dj3/settings.py
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'static'
```

接著修改一些安全性設定，首先將 `SECRET_KEY` 改成吃環境變數的方式，避免直接暴露於版本控制中。

P.S.: 本文為了 Demo 方便，將預設產生的 `SECRET_KEY` 值設定成 `default`，請不要使用於正式環境。

```python
# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.getenv('SECRET_KEY', '!^-p96ces65=nkgedrbvc2v5^y6asmm5#y#-3+&u7edzju16pu')
```

再來則是 Django 自帶的 Debug 模式，由於 Debug 模式會呈現很多資訊，如路徑、變數名稱、變數內容⋯⋯等等，請不要使用於正式環境。

```python
# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = os.getenv('DEBUG', True)
```

為了讓 `api` 的內容可以被讀取到，需要修改 `dj3` 的路由設定，將 `dj3/urls.py` 的檔案修改如下。

```python
# dj3/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('api.urls')),
]
```

在剛剛新增的 `api/urls.py` 檔案中，增加以下的程式碼。

```python
# api/urls.py
from django.urls import path
from . import views


urlpatterns = [
    path('', views.index, name='index'),
]
```

將 `api/views.py` 的檔案修改如下。

```python
# api/views.py
from django.shortcuts import render
from django.http import HttpResponse


def index(request):
    return HttpResponse("I am Django 3.1 !!!")
```

好啦！一個簡易的 Django application 就完成了。

最後啟動 Django 自帶的開發伺服器。

```python
python manage.py runserver
```

啟動後如果沒有出現錯誤，應該就會在 terminal 中看到

```shell
Django version 3.1.2, using settings 'dj3.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

在瀏覽器中開啟 [http://localhost:8000/api/](http://localhost:8000/api/)，就可以看到 `I am Django 3.1 !!!` 的文字；

而開啟 [http://localhost:8000/admin/](http://localhost:8000/admin/) 的話，就可以看到 Django 自帶的管理介面啦。

## 2. Run the Django project using uWSGI

Django 自帶的開發伺服器，其功能只是讓開發者在開發過程中可以方便的看到變化，其效能並不適合拿到正式環境中使用。

Django 常見的 Web Application Server 有 [uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/#)、[gunicorn](https://gunicorn.org/)。

這邊採用 `uWSGI`，而詳細的設定可以參考 [官方文件](https://uwsgi-docs.readthedocs.io/en/latest/Options.html?highlight=options)

確認在 `dj3` 專案的路徑下後，新增一個 `uWSGI` 的設定檔，並新增以下的設定。

```shell
touch uwsgi.ini
```

```python
# dj3/uwsgi.ini
[uwsgi]
http = :8003
module = dj3.wsgi:application
master = True
processes = 1
threads = 1
vacuum = True
pidfile = /tmp/dj3-master.pid
```

以下簡易說明各項設定：

- `http`: 使用 HTTP protocal 將服務跑在 `127.0.0.1:8003`。
- `module`: 設定 uWSGI 的 modele，也就是 `dj3` 專案中的 `wsgi.py` 中的 `application` 物件。
- `pidfile`: 設定一個 pidfile，否則 `ctrl + C` 會關不掉。

最後，將啟動 uWSGI 伺服器

```shell
uwsgi --ini uwsgi.ini
```

接著在瀏覽器中開啟 [http://localhost:8003/api/](http://localhost:8003/api/)，就可以看到 `I am Django 3.1 !!!` 的文字；

而開啟 [http://localhost:8003/admin/](http://localhost:8003/admin/) 的話，就可以看到 Django 自帶的管理介面啦。

***注意是 `8003` 而不是先前的 `8000` 喔！***

你應該會注意到當你開啟 [http://localhost:8003/admin/](http://localhost:8003/admin/) 可以看到輸入帳號密碼的欄位，但是原本的 `CSS` 樣式卻都消失了。

這是因為我們沒有在 `uwsgi.ini` 中設定靜態檔案的路徑，如果你要用 `uWSGI` 伺服器來取用靜態檔案的話，你可以在設定檔中新增 [`static-map`](https://uwsgi-docs.readthedocs.io/en/latest/Options.html#static-map) 的設定，不過在本文中，我將用 `Nginx` 來取用靜態檔案。

## 3. The preparation of Nginx

[Nginx](https://www.nginx.com/) 是一個高效且功能五花八門的 Web Server，同時它還具有 Reverse proxy、Load balancer 與 HTTP cache⋯⋯等功能，而在這邊我將利用 `Nginx` 作為 Reverse proxy 及取用靜態檔案。

`Nginx` 的兩大功能：

- `Reverse proxy`: 監聽 `80` port 並將其 Request 導向 uWSGI 伺服器(`8003` port)，最後再將 Respones 回傳給 Nginx 伺服器。
    - `Nginx` 作為 `Reverse proxy` 所支援的 protocal 除了 `HTTP` 以外，也包含 `FastCGI`、`uwsgi`、`SCGI` 與 `memcached`。
- `Serve static files`: 由於靜態檔案(如 `CSS`、`JavaScript`、`png` 等) 並不會隨著 Request 而更動內容，所以 Nginx 可以直接取用靜態檔案而不需要經過 uWSGI 伺服器。

其架構大致如下：

```shell
# Dynamic respones
browser <-> 80 port <-> nginx <-> 8003 port <-> uwsgi <-> django
# Static files
browser <-> 80 port <-> nginx <-> static files
```

首先，確認你已經安裝 Nginx，並且移動到該路徑下

- macOS: `/usr/local/etc/nginx`
- CentOS 7: `/etc/nginx`
- Ubuntu: `/usr/local/nginx`

修改 `nginx.conf`，將預設的 `conf.d/*.conf` 註解掉，並新增 `include /.../.../nginx/sites-enabled/*` 的設定。

macOS
```shell
    # include /usr/local/etc/nginx/conf.d/*.conf;
    include /usr/local/etc/nginx/sites-enabled/*;
```

CentOS 7
```shell
    # include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
```

接著建立兩個資料夾

```shell
mkdir sites-available
mkdir sites-enabled
```

在 `sites-available` 中建立設定檔

```shell
touch sites-available/deploy-at-root-proxy-pass.conf
```

設定檔會放置在 `sites-available` 資料夾中，而當確定要使用某個設定檔的時候，再利用 `symbolic link(soft link)` 將該設定檔連結到 `sites-enabled` 中，避免直接在 `nginx.conf` 或 `conf.d` 中修改設定。

這樣做的好處是，你可以在 `sites-available` 存放多個不同服務的設定檔，但只有將要啟動的設定檔連結到 `sites-enabled`，如果你要轉換到另一個設定檔的時候，只要單純在 `sites-enabled` 將不要的 `soft link` 刪除，並且重新連結新的設定檔到 `sites-enabled` 即可。

## 4. Run the Django project using uWSGI + Nginx with proxy_pass

這節先使用 `proxy_pass` 作為設定。

修改先前所建立的 `deploy-at-root-proxy-pass.conf`

```
# sites-available/deploy-at-root-proxy-pass.conf

server {
    # the port your site will be served on
    listen 80;
    # the domain name it will serve for
    # substitute your machine's IP address or FQDN
    server_name _;
    charset utf-8;

    client_max_body_size 75M;   # adjust to taste

    location / {
        proxy_pass http://127.0.0.1:8003/;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header  X-Forwarded-Proto $scheme;
    }

    location /static/ {
        alias /your/path/dj3/static/;
    }

}
```

注意！部署時，記得切換到 `dj3` 專案目錄後利用 [`collectstatic` 指令](https://docs.djangoproject.com/en/3.1/howto/static-files/deployment/#deploying-static-files)，將所有靜態檔案都收集到 `STATIC_ROOT` 中。

```shell
cd dj3
python manage.py collectstatic
```

利用 `symbolic link` 將 `sites-available/deploy-at-root-proxy-pass.conf` 連結至 `sites-enabled/` 路徑下(***注意：要用絕對路徑！***)。

```shell
# macOS
ln -s /usr/local/etc/nginx/sites-available/deploy-at-root-proxy-pass.conf /usr/local/etc/nginx/sites-enabled/

# CentOS 7
ln -s /etc/nginx/sites-available/deploy-at-root-proxy-pass.conf /etc/nginx/sites-enabled/
```

檢查設定檔是否正確

```shell
nginx -t
```

重新啟動 Nginx

```
# macOS
brew services restart nginx
# CentOS 7
systemctl restart nginx
```

`Nginx` 重啟後，開啟 [http://localhost/admin/](http://localhost/admin/) 的話，就可以看到 Django 自帶的管理介面，而且 CSS 樣式也可以正常顯示。同時，開啟 [http://localhost:8003/admin/](http://localhost:8003/admin/) 管理介面時，仍舊沒有 CSS 樣式，這表示靜態檔案確實地是由 `Nginx` 所取用而不是由 `uWSGI`。

最後，這個簡易的 Django application 就已經成功地被部署到主機的根目錄啦～～～

## 5. Run the Django project using uWSGI + Nginx with uwsgi_pass

本節使用 `uwsgi_pass` 作為設定。

首先，將 `uwsgi.ini` 中的 `http` protocal 修改成 `socket`，`socket` 預設使用 UNIX protocal。

```python
# dj3/uwsgi.ini
[uwsgi]
# http = :8003
socket = :8003
```

接著在 `sites-available` 中建立設定檔

```shell
touch sites-available/deploy-at-root-uwsgi-pass.conf
```

修改 `deploy-at-root-uwsgi-pass.conf`

```
# sites-available/deploy-at-root-uwsgi-pass.conf

server {
    # the port your site will be served on
    listen 80;
    # the domain name it will serve for
    # substitute your machine's IP address or FQDN
    server_name _;
    charset utf-8;

    client_max_body_size 75M;   # adjust to taste

    location / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8003;
    }

    location /static/ {
        alias /your/path/dj3/static/;
    }

}
```

利用 `symbolic link` 將 `sites-available/deploy-at-root-uwsgi-pass.conf` 連結至 `sites-enabled/` 路徑下(***注意：要用絕對路徑！***)。

```shell
# macOS
ln -s /usr/local/etc/nginx/sites-available/deploy-at-root-uwsgi-pass.conf /usr/local/etc/nginx/sites-enabled/

# CentOS 7
ln -s /etc/nginx/sites-available/deploy-at-root-uwsgi-pass.conf /etc/nginx/sites-enabled/
```

檢查設定檔是否正確

```shell
nginx -t
```

重新啟動 Nginx

```
# macOS
brew services restart nginx
# CentOS 7
systemctl restart nginx
```

`Nginx` 重啟後，開啟 [http://localhost/admin/](http://localhost/admin/) 的話，就可以看到 Django 自帶的管理介面，而且 CSS 樣式也可以正常顯示。

如果你像前一節一樣用瀏覽器開啟 [http://localhost:8003/admin/](http://localhost:8003/admin/) 管理介面時，會發現無法開啟；同時，檢查 `uWSGI` 的 terminal 時會發現這樣的訊息。

```shell
invalid request block size: 21573 (max 4096)...skip
```

這是因為我們在設定 `uwsgi_pass` 之前，有先將 `uwsgi.ini` 的設定檔改成 `socket = :8003`，這表示我們的 Django 服務是使用 UNIX socket 跑在 `127.0.0.1:8003` 上，但是並不是使用 HTTP protocal，所以無法用瀏覽器打開。

最後，這個簡易的 Django application 也成功地被部署到主機的根目錄啦～～～
