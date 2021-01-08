---
title: "DevOps：如何將容器化 (Docker + Nginx + uWSGI + Django + PostgreSQL) 的服務部署在一台主機的根目錄"
categories: DevOps
tags: docker docker-compose
lang: en
toc: true
---

>How to deploy the containerized web application at the root on a host machine using Docker + Nginx + uWSGI + Django + PostgreSQL

從上一篇 [DevOps：如何將 Nginx + uWSGI + Django 的服務部署在一台主機的根目錄]({{ site.baseurl }}{% post_url 2020-11-07-nginx-uwsgi-django-root %}) 中已經知道如何建立並部署一個 Django 專案。

從這篇開始紀錄如何將整個專案容器化後，並部署到主機的根目錄上。本文不會著重在 Docker 的細節，相關細節可以參考 [Dockerfile](https://docs.docker.com/engine/reference/builder/) 與 [Docker Compose](https://docs.docker.com/compose/)。完整的 sample code 請參考 [docker-compose-deploy-at-root](https://github.com/orcahmlee/lab-technical-code/tree/master/DevOps/docker-compose-nginx-uwsgi-django-postgres/docker-compose-deploy-at-root)。


以下的設定皆是在 `Mac` 上進行測試，若是在 `CentOS 7` 或是 `Ubuntu` 上的話，`Nginx` 的部分可能需要微調。

OS Information and packages version

```shell
macOS Catalina, Version 10.15.6
Docker version 19.03.13
docker-compose version 1.27.4
conda 4.8.5
django 3.1.2
uwsgi 2.0.19.1
nginx version: nginx/1.19.3
```

---
## 1. Containerizing the simple Django project using Dockerfile

同樣的，本文不對 `Django` 做太多說明，詳細的教學與設定可以參考官方文件 [Get started with Django](https://www.djangoproject.com/start/)。

這邊直接複製上一篇 [DevOps：如何將 Nginx + uWSGI + Django 的服務部署在一台主機的根目錄]({{ site.baseurl }}{% post_url 2020-11-07-nginx-uwsgi-django-root %}) 的 Django 專案 `dj3`，把整個 `dj3` 資料夾複製過來並放在 `web` 的資料夾中，最後的檔案結構會像這樣(先忽略其他東西)。

```shell
.
├── docker-compose.yml
├── nginx
│   ├── Dockerfile
│   ├── default.conf
│   ├── docker-nginx-dj3.conf
│   ├── nginx-origin.conf
│   └── nginx.conf
└── web
    ├── Dockerfile
    ├── dj3
    ├── requirements.txt
    └── wait-for-it.sh
```

複製完後，接著在 `web` 資料夾中開始編輯 `Dockerfile`，內容如下

```docker
FROM python:3.7
LABEL maintainer="orcahmlee@gmail.com"


WORKDIR /web
COPY . /web/

RUN pip install -r requirements.txt

WORKDIR /web/dj3

VOLUME /web
EXPOSE 8000

ENTRYPOINT [ "/bin/bash", "docker-entrypoint.sh" ]
CMD python manage.py runserver 0.0.0.0:8000
```

以下簡易說明各項設定：

- `WORKDIR`: 設定工作目錄，如果該目錄不存在的話會自動建立。
- `COPY`: 將所需要的檔案從 host 複製到 container 內部，這裡的寫法是將 host 上 `web` 內所有的檔案複製到 container 中的 `/web/`。
- `RUN`: 執行指令，這裡的寫法是安裝 `requirements.txt` 所列出的套件。
- `VOLUME`: 列出需要分享的路徑或需要持久化的資料路徑，供 `docker run` 時使用。
- `EXPOSE`: 列出需要導出的 port，供 `docker run` 時使用。
- `ENTRYPOINT`: 可以將 container 每次都需要執行的指令整理成 `docker-entrypoint.sh`；同時 `ENTRYPOINT` 還可以接受 `CMD` 的指令作為 `docker-entrypoint.sh` 內部的參數使用。
- `CMD`: 希望 container 所執行的預設指令。

`ENTRYPOINT` 與 `CMD` 很容易搞混，但是非常實用；`ENTRYPOINT` 與 `CMD` 彼此的相互關係可參考 [Understand how CMD and ENTRYPOINT interact](https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact)。

做到這一步驟，基本上就已經把先前的 `dj3` 專案給容器化了，接著來測試看看。

先移動到 `Dockerfile` 所在的路徑後，接著 build image。

```shell
$ cd web

$ docker build -t andrew/dj3 .
```

完成後，檢查 image 是否有成功被建立。

```shell
$ docker images
REPOSITORY                            TAG                 IMAGE ID            CREATED             SIZE
andrew/dj3                            latest              f5d1953ca451        8 seconds ago       924MB
```

確定剛剛的 image 有被建立之後，就可以利用這個 image 來建立並啟動 container。以下的指令在啟動 container 時，會執行 `docker-entrypoint.sh` 內的指令

```shell
# docker-entrypoint.sh
#!/bin/bash

# Collect static files
echo "Collect static files"
python manage.py collectstatic --noinput

# Make migrations
echo "Make migrations"
python manage.py makemigrations

# Apply database migrations
echo "Apply database migrations"
python manage.py migrate

exec "$@"
```

分別是：

1. 收集所有靜態檔案到部署時所指定的路徑
2. 建立 model 所變動的 migrations
3. 將 migrations 套用到資料庫中
4. 最後，再執行 `CMD` 所傳入的指令

```shell
$ docker run --rm -it -p 8000:8000 -v $(PWD):/app andrew/dj3
Collect static files

132 static files copied to '/web/dj3/static'.
Make migrations
No changes detected
Apply database migrations
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying sessions.0001_initial... OK
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
December 30, 2020 - 16:08:07
Django version 3.1.4, using settings 'dj3.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```

從 terminal 輸出的結果中，可以看到這個 container 在啟動過程中執行 `docker-entrypoint.sh` 的指令，並且最後成功啟動 Django 自帶的開發伺服器。由於在 `docker run` 時，有將 host 與 container 的 `8000` port 打通，所以這時候在瀏覽器中開啟 [http://localhost:8000/api/](http://localhost:8000/api/)，就可以看到 `I am Django 3.1 !!!` 的文字。

## 2. Adding the Nginx's Dockerfile and modify the configurations

畫面切到 `nginx` 的資料夾，開始編輯 `Dockerfile`，內容如下

```docker
FROM nginx:latest
LABEL maintainer="orcahmlee@gmail.com"


COPY nginx.conf /etc/nginx/nginx.conf
COPY docker-nginx-dj3.conf /etc/nginx/sites-available/

RUN mkdir -p /etc/nginx/sites-enabled/ && \
    ln -s /etc/nginx/sites-available/docker-nginx-dj3.conf /etc/nginx/sites-enabled/

CMD ["nginx", "-g", "daemon off;"]
```

以下簡易說明各項設定：

1) 利用修改過的 `nginx.conf` 取代預設的設定，修改的部分有兩處。第一個是將 `user nginx` 修改成 `user root`，第二個則是原先的 `include` 註解掉並修改如下

```shell
    # include /etc/nginx/conf.d/*.conf;  # comment this line
    include /etc/nginx/sites-available/*;  # then using this line
```

2) 複製 `dj3` 專案專屬的設定 `docker-nginx-dj3.conf` 到 `/etc/nginx/sites-available/` 這個路徑是 `Nginx` 存放各種不同設定檔的位置。

3) 建立新路徑 `mkdir -p /etc/nginx/sites-enabled/`，而這個路徑是存放 ***需要啟動的設定檔*** 的位置，接著再利用 `symbolic link` 將 `docker-nginx-dj3.conf` 連結至 `sites-enabled` 路徑下。

4) 在 `Nginx` 的服務中，`nginx` 的指令預設會將該服務跑在背景模式，接著 return `Exit code 0`，然後 Docker 就會將這個 container 關閉。為了讓 `Nginx` 這個 container 一直存活著，這邊利用 `CMD` 的指令 `daemon off` 讓服務可以一直跑在前景模式。


完成 `Dockerfile` 後，接著說明 `docker-nginx-dj3.conf` 的設定檔。

```shell
upstream uwsgi {
    # server unix:/web/dj3/web.sock; # using a file socket
    server web:8003;  # using the docker network
}
```

`upstream` 可以設定 server 叢集(a group of servers)，`uwsgi` 是叢集名稱，也可以設定 Load balancing，更多細節可以參考 [官方文件 #upstream](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream)。這邊簡單的設定單一 server，也就是 `dj3` 服務所啟動的位置，`web` 代表 `Docker Network` 中的 `web` 服務(後面會提到)。

```shell
server {
    location /static/ {
        alias /web/dj3/static/; # your Django project's static files - amend as required
    }
}
```

`location /static/` 將靜態資源的請求導向至 Django 專案中的 `static` 資料夾，也就是 `python manage.py collectstatic` 收集後所放置的路徑。

```shell
server {
    location / {
        # uwsgi_pass uwsgi;
        # include /etc/nginx/uwsgi_params; # the uwsgi_params file you installed
        proxy_pass http://uwsgi;  # using http protocal
    }
}
```

`location /` 將根目錄的請求導向至 `uwsgi` 這個 server 叢集，注意這邊是使用 `proxy_pass` 所以需要加上 `http` protocol 的前綴，更多細節可以參考 [官方文件 #proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)。

## 3. Using docker-compose to combine the Nginx, uWSGI and Django

成功將 `dj3` 和 `Nginx` 的服務容器化之後，接著要利用 `docker-compose.yml` 將 `dj3` 與 `Nginx` 這兩個容器串連起來，這樣的好處是可以同時管理多個容器，且各個容器可以在同一個[Compose 網路環境](https://docs.docker.com/compose/networking/)中互相溝通。

```shell
.
├── docker-compose.yml
├── nginx
│   ├── Dockerfile
│   ├── default.conf
│   ├── docker-nginx-dj3.conf
│   ├── nginx-origin.conf
│   └── nginx.conf
└── web
    ├── Dockerfile
    ├── dj3
    ├── requirements.txt
    └── wait-for-it.sh
```

回到根目錄中開始編輯 `docker-compose.yml`

```shell
version: '3.8'

services: 
    web:
        build: ./web
        container_name: dj3_web
        restart: always
        command: [ "/bin/bash", "-c", "uwsgi --ini uwsgi.ini" ]
        volumes:
            # Using for production that could share the named volume for other services.
            - web_data:/web/dj3
        environment:
            - PYTHONUNBUFFERED=TURE
    nginx:
        build: ./nginx
        container_name: dj3_nginx
        restart: always
        volumes:
            # Using the named volume from the Django project.
            - web_data:/web/dj3
        ports:
            - "127.0.0.1:8000:80"
        depends_on:
            - web

volumes:
    web_data:
```

接著利用以下指令，就可以啟動 `dj3` 與 `Nginx` 這兩個 container，同時 `docker-compose` 會建立 `network` 與 `volume`，在相同的 `network` 與 `volume` 中 container 彼此之間可以互相溝通也可以共享資料。這時候再去瀏覽器中開啟 [http://localhost:8000/api/](http://localhost:8000/api/)，就可以看到 `I am Django 3.1 !!!` 的文字。

```shell
$ docker-compose up 
Creating network "docker-compose-deploy-at-root_default" with the default driver
Creating volume "docker-compose-deploy-at-root_web_data" with default driver
......以下省略
```

以下簡易說明各項設定：

1) 在 `services` directive 中，定義兩個服務，分別是 `web` 與 `nginx`，`web` 就是 `dj3` 這個 Django 專案，而 `nginx` 就是 `Nginx` 的服務；`web` 與 `nginx` 除了是服務名稱外同時也是 IP 位置，這就是為什麼先前的 Nginx 設定檔中要寫成 `server web:8003;`。

2) `build` 指的是要將哪個路徑下的 `Dockerfile` 建立成 `image`。

3) `web` container 中的 `command` directive 相當於 `Dockerfile` 內的 `CMD`，這邊使用 `uwsgi --ini uwsgi.ini` 去覆蓋 `Dockerfile` 內的 `python manage.py runserver 0.0.0.0:8000`。

4) `volumes` 定義 container 內的哪些資料需要 `volume` 或 `bind mount` 出來，請參考 [Use volumes](https://docs.docker.com/storage/volumes/)。

5) `depends_on` 定義 container 的啟動順序。

## 4. Adding the PostgreSQL container to docker-compose

做到這邊之後，已經可以將 `Nginx` 與 `Django` 容器化並利用 `docker-compose` 來管理了，但是正式的服務並不會用 `SQLite`，於是最後這邊我將資料庫改成使用 `PostgreSQL`，需要修改的檔案只有兩個 `settings.py` 與 `docker-compose.yml`。

首先修改 `settings.py` 中的資料庫設定

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('DATABASE_NAME'),
        'USER': os.getenv('DATABASE_USE'),
        'HOST': os.getenv('DB_HOST', ''),
        'PORT': os.getenv('DB_PORT', 5432),
    }
}
```

接著修改 `docker-compose.yml` 如下

```shell
version: '3.8'

services: 
    db:
        image: postgres:12
        container_name: dj3_db
        restart: always
        volumes:
            - pg_data:/var/lib/postgresql/data
        environment:
            - POSTGRES_USER=dj3
            - POSTGRES_DB=dj3
            - POSTGRES_PASSWORD=dj3
    web:
        build: ./web
        container_name: dj3_web
        restart: always
        command: [ "../wait-for-it.sh", "db:5432", "--", "/bin/bash", "-c", "uwsgi --ini uwsgi.ini" ]
        volumes:
            # Using for production that could share the named volume for other services.
            - web_data:/web/dj3
        environment:
            - PYTHONUNBUFFERED=TURE
            - DATABASE_NAME=dj3
            - DATABASE_USER=dj3
            - DB_HOST=db
    nginx:
        build: ./nginx
        container_name: dj3_nginx
        restart: always
        volumes:
            # Using the named volume from the Django project.
            - web_data:/web/dj3
            # Bind the nginx's log into host machine.
            - ./nginx/logs:/var/log/nginx
        ports:
            - "127.0.0.1:8000:80"
        depends_on:
            - web

volumes:
    pg_data:
    web_data:
```

增加的部分有，新增 `db` 服務，這個 container 直接使用 [PostgreSQL Official Images](https://hub.docker.com/_/postgres)，並且新增三個環境變數 `POSTGRES_USER`、`POSTGRES_DB`、`POSTGRES_PASSWORD`。接著在 `web` 服務中新增三個環境變數 `POSTGRES_USER`、`POSTGRES_DB` 與 `DB_HOST`，其中 `DB_HOST` 設定成 `db` 這個服務名稱同時也是 IP 位置(因為這些 container 存活在相同的 `network` 中)。

```shell
    db:
        image: postgres:12
        container_name: dj3_db
        restart: always
        volumes:
            - pg_data:/var/lib/postgresql/data
        environment:
            - POSTGRES_USER=dj3
            - POSTGRES_DB=dj3
            - POSTGRES_PASSWORD=dj3
    web:
        environment:
            - PYTHONUNBUFFERED=TURE
            - DATABASE_NAME=dj3
            - DATABASE_USER=dj3
            - DB_HOST=db
```

修改的部分，則是修改 `web` 服務中的 `command`，讓啟動 `uwsgi` 啟動之前，利用 [wait-for-it.sh](https://github.com/vishnubob/wait-for-it) 先去檢測 `db` 中的 PostgreSQL 是否已經啟動完成。

```shell
    web:
        command: [ "../wait-for-it.sh", "db:5432", "--", "/bin/bash", "-c", "uwsgi --ini uwsgi.ini" ]
```

最後，還是接著利用以下指令，啟動 `dj3`、`db` 與 `Nginx` 這三個 container，接著再去瀏覽器中開啟 [http://localhost:8000/api/](http://localhost:8000/api/)，就可以看到 `I am Django 3.1 !!!` 的文字。

```shell
$ docker-compose up 
```