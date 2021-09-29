---
title: "[Study] Docker install in Linux & create, upload"
categories:
  - Docker
tags:
  - Docker
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---

# 1. 설치

도커에서 다양한 리눅스 배포판에 대해 알아서 인식하고 도커 패키지를 설치해주는 아래와 같은 스크립트를 제공함. 설치 후 docker version으로 설치가 제대로 됬는지 확인 가능.

- docker 사용을 위한 kernel 버전은 3.10.x 이상 (ubuntu 14.04 이상이면 가능)

```
sudo wget -qO- http://get.docker.com/ | sh
docker version
```

‌

docker는 기본적으로 sudo 권한을 필요로 하기 때문에 아래와 같이 권한을 주어 sudo 없이 사용이 가능함.

```
sudo usermod -aG docker $USER # 현재 접속중인 사용자에게 권한주기
sudo usermod -aG docker your-user # your-user 사용자에게 권한주기
```

# 2. Run 명령‌

## 2.1. 기본 실행법

다음과 같이 명령어를 입력하면, 해당 이미지가 저장되어 있는지 확인 후 없다면 이미지를 다운로드한다. 저장된 이미지가 있다면 컨테이너 생성, 시작 순으로 동작이 수행된다.

- docker run ubuntu:16.04

```
# 기본 도커 실행 명령어
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG....]‌
```

- OPTIONS : run 명령어에 대한 옵션들
    - -d : detached mode (백그라운드 모드)
    - -p : forwarding (Host와 Container의 port 연결)
    - -v : mount (Host와 Container의 directory 연결)
    - -e : Container 내 사용할 환경변수 설정
    - -name : Container 이름 설정
    - -rm : 프로세스 종료와 동시에 컨테이너 제거
    - -it : 명령어 줄 때 사용
    - 기타 등등
- TAG : Version 정보
    - ex) ubuntu:16.04
- COMMAND : /bin/bash와 같은 Container 내부적으로 수행할 명령

‌

## 2.2. 다양한 컨테이너 실행 예

컨테이너 실행 시 기본적으로 무엇을 할지 명령어를 전달 주지 않으면 실행되자마자 프로세스가 끝나버린다. (컨테이너는 프로세스 단위) 그래서, 컨테이너마다 초기에 필요한 명령어를 전달해주는 것이 필요하다. 예제들을 통해 알아보면,

### 2.2.1. Ubuntu‌

우분투를 컨테이너로 실행시키려면 쉘 명령어를 쓸 수 있는 환경이 되도록 해주어야 한다. 그래서 명령어로 '/bin/bash' 를 넘겨주어 초기 쉘 환경을 만들어준다. 우분투의 경우 실행 후 사용자가 계속해서 명령어를 입력할 것이기 때문에 detach 모드로 실행하지 않는다.

```
docker run --rm -it ubuntu:16.04 /bin/bash
# in container
$ cat /etc/issue
Ubuntu 16.04.1 LTS \n \l
$ ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
```

### 2.2.2. Redis

Redis는 메모리 기반의 스토리지이므로 실행 자체에 전달할 명령어는 없다. 다만, 포트를 설정해주어 실행 후 telnet 명령어로 접근하여 다양한 기능을 이용해 볼 수 있다.

```
docker run -d -p 1234:6379 redis
# redis test
$ telnet localhost 1234
set mykey hello
+OK
get mykey
$5
hello
```

‌

### 2.2.3. MySQL

이번에는 환경변수와 이름 옵션을 사용하여 MySQL를 동작시킨다. 'MYSQL_ALLOW_EMPTY_PASSWORD' 라는 환경변수 설정으로 패스워드 없이 root 계정을 만들게 하고, name에 mysql을 주어 컨테이너 식별을 쉽게 한다.

```
docker run -d -p 3306:3306 \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --name mysql \
  mysql:5.7
# MySQL test
$ mysql -h127.0.0.1 -uroot
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
mysql> quit
```

### 2.2.4. WordPress

블로그나 웹으로 많이 사용되는 엔진으로 앞 예제들보단 실행법이 살짝 복잡하다. 우선, 데이터베이스가 필요하니 MySQL로 데이터베이스를 만들어 준다. 그 후, --link 옵션을 통해 MySQL 데이터베이스를 wordpress에 연결시켜준다. link 옵션은 연결한 컨테이너의 IP와 환경변수 정보를 호스트에게 공유해주기 때문에 wordpress에서 자연스럽게 MySQL 데이터베이스 정보를 알 수 있다.

> --link 옵션은 deprecated 되었기 때문에, 실제로는 docker network 기능을 사용하여야 함.
> 

```
# create mysql database
$ mysql -h127.0.0.1 -uroot
create database wp CHARACTER SET utf8;
grant all privileges on wp.* to wp@'%' identified by 'wp';
flush privileges;
quit
# run wordpress container
docker run -d -p 8080:80 \
  --link mysql:mysql \
  -e WORDPRESS_DB_HOST=mysql \
  -e WORDPRESS_DB_NAME=wp \
  -e WORDPRESS_DB_USER=wp \
  -e WORDPRESS_DB_PASSWORD=wp \
  wordpress
```

### 2.2.5. Tensorflow

Tensorflow를 docker 통해서도 쉽게 환경구성이 가능하다. tensorflow 뿐 아니라 연관되어 필요한 아래의 리스트를 모두 같이 제공한다.

- numpy, scipy, pandas, jupyter, scikit-learn, gensim, BeautifulSoup4

```
docker run -d -p 8888:8888 -p 6006:6006 teamlab/pydata-tensorflow:0.1
```

‌

# 3. 자주 사용할 도커 명령어

## 3.1. ps‌

ps는 실행 중인 컨테이너 목록을 확인하는 명령어이다. 여기에 -a 또는 --all을 추가하면, 실행 중이지 않은 컨테이너도 확인된다. 아래에서 우분투 컨테이너에는 Exited(O)가 표기되어 있는데, 이는 컨테이너가 종료되어 있다는 뜻이며 다시 실행이 가능하다.

```
docker ps [OPTIONS]
docker ps
CONTAINER ID        IMAGE                           COMMAND                  CREATED              STATUS              PORTS                                                    NAMES
6a1d027b604f        teamlab/pydata-tensorflow:0.1   "/opt/start"             About a minute ago   Up About a minute   0.0.0.0:6006->6006/tcp, 22/tcp, 0.0.0.0:8888->8888/tcp   desperate_keller
52a516f87ceb        wordpress                       "docker-entrypoint.sh"   8 minutes ago        Up 8 minutes        0.0.0.0:8080->80/tcp                                     happy_curran
2e2c569115b9        mysql:5.7                       "docker-entrypoint.sh"   9 minutes ago        Up 9 minutes        0.0.0.0:3306->3306/tcp                                   mysql
56341072b515        redis                           "docker-entrypoint.sh"   16 minutes ago       Up 9 minutes        0.0.0.0:1234->6379/tcp                                   furious_tesla
docker ps -a
CONTAINER ID        IMAGE                           COMMAND                  CREATED             STATUS                      PORTS                                                    NAMES
6a1d027b604f        teamlab/pydata-tensorflow:0.1   "/opt/start"             2 minutes ago       Up 2 minutes                0.0.0.0:6006->6006/tcp, 22/tcp, 0.0.0.0:8888->8888/tcp   desperate_keller
52a516f87ceb        wordpress                       "docker-entrypoint.sh"   9 minutes ago       Up 9 minutes                0.0.0.0:8080->80/tcp                                     happy_curran
2e2c569115b9        mysql:5.7                       "docker-entrypoint.sh"   10 minutes ago      Up 10 minutes               0.0.0.0:3306->3306/tcp                                   mysql
56341072b515        redis                           "docker-entrypoint.sh"   18 minutes ago      Up 10 minutes               0.0.0.0:1234->6379/tcp                                   furious_tesla
e1a00c5934a7        ubuntu:16.04                    "/bin/bash"              32 minutes ago      Exited (0) 32 minutes ago                                                            berserk_visvesvaray
```

## 3.2. stop

컨테이너를 중지하는 명령어로 특별할 건 없다. 중지하려고하는 컨테이너의 ID만 추가로 입력하면 된다.

> 도커의 ID는 길이가 64자인데, 겹치지만 않는다면 앞의 2자리정도만 입력해도 자동으로 매칭된다.
> 

```
docker stop [OPTIONS] CONTAINER [CONTAINER...]
docker ps # get container ID
docker stop ${TENSORFLOW_CONTAINER_ID}
docker ps -a # show all containers
```

## 3.3. rm

컨테이너를 제거하는 명령어이다. 중지와 마찬가지로 크게 중요한 옵션은 없다.

```
docker rm [OPTIONS] CONTAINER [CONTAINER...]
docker ps -a # get container ID
docker rm ${UBUNTU_CONTAINER_ID} ${TENSORFLOW_CONTAINER_ID}
docker ps -a # check exist
```

> 중지된 컨테이너를 일일이 삭제 하는 건 귀찮은 일입니다. docker rm -v $(docker ps -a -q -f status=exited) 명령어를 입력하면 중지된 컨테이너 ID를 가져와서 한번에 삭제합니다.
> 

## 3.4. images

이미지 목록을 확인하는 명령어이다. 이미지의 다양한 정보들이 나온다. 틈틈히 필요없는 이미지는 삭제해주는 것이 관리에 편할 것 같다.

```
docker images [OPTIONS] [REPOSITORY[:TAG]]
docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
wordpress                   latest              b1fe82b15de9        43 hours ago        400.2 MB
redis                       latest              45c3ea2cecac        44 hours ago        182.9 MB
mysql                       5.7                 f3694c67abdb        46 hours ago        400.1 MB
ubuntu                      16.04               104bec311bcd        4 weeks ago         129 MB
teamlab/pydata-tensorflow   0.1                 7bdf5d7e0191        6 months ago        3.081 GB
```

## 3.5. pull

이미지를 다운로드하는 명령어이다. 어차피 run 명령어에 없으면 자동으로 받는 기능이 포함되어 있으니 안 쓸 것 같지만, pull 명령어로 최신버전을 받을 때 사용한다.

```
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
docker pull ubuntu:14.04
```

## 3.6. rmi

이미지를 삭제하는 명령어이다. images 명령어에서 볼 수 있는 ID 중 입력하면 삭제된다. 해당 이미지에 대해 컨테이너가 실행 중이라면, 컨테이너를 종료해야지 삭제가 가능하다.

```
docker rmi [OPTIONS] IMAGE [IMAGE...]
docker images # get image ID
docker rmi ${TENSORFLOW_IMAGE_ID}
```

## 3.7. logs

컨테이너의 로그를 보는 명령어이다. 해당 명령어를 통해 컨테이너가 정상 동작하는지 확인 가능하다. 다음과 같은 옵션을 통해 로그 정보를 어떻게 출력할지 정할 수 있다.

- --tail : --tail 10 과 같이 입력하면, 마지막 10줄의 로그만 보여준다.
- -f : 해당 옵션 사용 시, 실시간으로 로그를 출력한다.

도커의 로그파일은 표준 스트림 우 stdout, stderr를 수집하여 보여줌. 컨테이너의 로그 파일은 json으로 어딘가에 저장되며, 플러그인들을 사용하면 json이 아닌 다른 로그 서비스에 전달 가능하다.

```
docker logs [OPTIONS] CONTAINER
docker ps
docker logs ${WORDPRESS_CONTAINER_ID}
```

## 3.8. exec

컨테이너 관리 및 실행 중인 컨테이너에 들어가거나 파일을 실행하려면 exec 명령어를 사용하면 된다. run 명령어와 형태는 비슷하지만, run은 컨테이너를 만들어서 실행하고 exec는 실행 중인 컨테이너에 명령을 하는 차이가 있다.

```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
#### 쉘을 통해서 접근하
docker exec -it mysql /bin/bash
# MySQL test
$ mysql -uroot
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| wp                 |
+--------------------+
5 rows in set (0.00 sec)
mysql> quit
exit
### 직접적으로 mysql 명령어 실행
docker exec -it mysql mysql -uroot
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| wp                 |
+--------------------+
5 rows in set (0.00 sec)
mysql> quit
```

‌

# 4. 컨테이너 업데이트

## 4.1. 데이터 볼륨 연결

도커에서 컨테이너 업데이트는‌

- 새 버전 이미지 다운로드 (pull)
- 기존 컨테이너 삭제 (stop, rm)
- 새 이미지 기반으로 컨테이너 실행 (run)

으로 이루어지며, 주의해야 할 점은 컨테이너를 삭제하면 그 안에서 생성/업로드 등을 진행했던 것이 모두 사라진다. 그래서 AWS S3같은 클라우드 서비스나 외부 디스크를 컨테이너에 연결하여서 외부에 데이터를 저장하는 것이 바람직하다.

‌

run 명령어 옵션 중 -v 옵션을 사용하면

```
# before
docker run -d -p 3306:3306 \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --name mysql \
  mysql:5.7
# after
docker run -d -p 3306:3306 \
  -e MYSQL_ALLOW_EMPTY_PASSWORD=true \
  --name mysql \
  -v /my/own/datadir:/var/lib/mysql \ # <- volume mount
  mysql:5.7‌
```

외부의 데이터 볼륨 /my/own/datadir 를 컨테이너의 /var/lib/mysql 에 연결하여 저 경로에 저장된 데이터는 컨테이너가 제거되도 사라지지 않게 된다.

‌

## 4.2. Docker Compose

지금까지 도커를 열심히 커맨드라인에서 명령어를 통해 작업하였지만, 실사용에서는 더 복잡한 작업들이 많기 때문에 여러가지 설정을 추가하기 어려워 진다. 도커는 YAML 방식의 설정파일을 이용해서 컨테이너의 세부 설정들을 손쉽게 관리할 수 있게 해준다.

리눅스만 별도로 docker compose를 다음과 같이 설치해주어야 한다.

```
curl -L "https://github.com/docker/compose/releases/download/1.9.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
# test
docker-compose version
```

‌

이전 예제의 wordpress를 같은 옵션으로 yaml 파일을 통해 만들게 되면 다음과 같다.

```
version: '2'
services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: wordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     volumes:
       - wp_data:/var/www/html
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
    db_data:
    wp_data:
```

yaml 파일 생성 후, docker-compose up 명령어를 사용하면 손쉽게 워드프레스가 실행된다.


# 5. 도커 이미지 만들기

도커는 이미지를 만들 때, 컨테이너의 상태를 그대로 저장하는 방법을 사용함.

예를 들면, 우분투 이미지에 원하는 프로그램을 추가로 설치하여 하나의 애플리케이션을 동작시킬 수 있는 환경을 구성하려면 우분투 이미지를 다운받고 그 안에서 쉘 명령어들로 필요한 것들을 전부 설치해야 한다. (기존 방법들과 크게 다를건 없다.) 하지만, 이런걸 잘 몰라도 기존에 많은 사람들이 많은 조합들을 만들어두었기 때문에 왠만한건 다 있으니 갖다 쓰면 된다.

‌

## 5.1. Sinatra 웹 애플리케이션 만들기

Sinatra를 사용하기 위한 Gemfile과 app.rb라는 파일을 만들어야 함. vim이든 뭐든 사용하여 원하는 장소에 만든다.

‌

**Gemfile**

```
source 'https://rubygems.org'
gem 'sinatra'
```

**app.rb**

```
require 'sinatra'
require 'socket'
get '/' do
  Socket.gethostname
end
```

루비로 작성되어있기 때문에 실행을 하려면 다음과 같은 코드를 입력해야 한다.

```
bundle install            # install package
bundle exec ruby app.rb   # Run sinatra
```

하지만, 루비가 설치되어 있지 않으면 안되기 때문에 도커를 이용해서 간단히 실행을 해볼 수 있다.

‌

**Run ruby**

```
docker run --rm \
-p 4567:4567 \
-v $PWD:/usr/src/app \
-w /usr/src/app \
ruby \
bash -c "bundle install && bundle exec ruby app.rb -o 0.0.0.0"
```

실행이 완료되고 [http://localhost:4567](http://localhost:4567/) 로 접속하면 아래와 같이 도커 컨테이너의 호스트 명이 보인다.

![https://gblobscdn.gitbook.com/assets%2F-M-9oD7tMq66BLgIHr7l%2F-M-xf6bo_WdgRPHp6hN0%2F-M00BsK--Ht37vsIN1Ez%2FScreenshot%20from%202020-02-14%2009-57-34.png?alt=media&token=ad4155cf-c953-461f-a671-0ef9b6e68ab1](https://gblobscdn.gitbook.com/assets%2F-M-9oD7tMq66BLgIHr7l%2F-M-xf6bo_WdgRPHp6hN0%2F-M00BsK--Ht37vsIN1Ez%2FScreenshot%20from%202020-02-14%2009-57-34.png?alt=media&token=ad4155cf-c953-461f-a671-0ef9b6e68ab1)

‌

## 5.2. Dockerfile 사용하기

도커는 이미지를 생성할 때 Dockerfile 이라는 이미지 빌드용 DSL(Domain Specific Language)를 사용한다. 도커 이미지를 생성하는 명령어(?)들을 텍스트 파일에 옮겨놓은 것으로 일반적으로 소스코드 + Dockerfile로 구성되어 배포한다. 아래의 순서대로 우선 쉘스크립로 표현해 보면

[Copy of Untitled](https://www.notion.so/d21e6deee69048f19085f76fb152ddb7)

‌

**Shell Script 표현**

아래 명령어를 순서대로 ubuntu 컨테이너에서 실행해도 sinatra 서버를 안정적으로 실행이 가능하다.

```
# 1. ubuntu 설치 (패키지 업데이트)
apt-get update
# 2. ruby 설치
apt-get install ruby
gem install bundler
# 3. 소스 복사
mkdir -p /usr/src/app
scp Gemfile app.rb root@ubuntu:/usr/src/app  # From host
# 4. Gem 패키지 설치
bundle install
# 5. Sinatra 서버 실행
bundle exec ruby app.rb
```

‌

**Dockerfile 표현**

쉘 스크립트의 내용을 거의 그대로 옮겼으며, 도커 빌드 중에는 키보드 입력이 안되기 때문에 미리 -y 옵션을 넣어줌.

```
# 1. ubuntu 설치 (패키지 업데이트 + 만든사람 표시)
FROM       ubuntu:16.04
MAINTAINER subicura@subicura.com
RUN        apt-get -y update
# 2. ruby 설치
RUN apt-get -y install ruby
RUN gem install bundler
# 3. 소스 복사
COPY . /usr/src/app
# 4. Gem 패키지 설치 (실행 디렉토리 설정)
WORKDIR /usr/src/app
RUN     bundle install
# 5. Sinatra 서버 실행 (Listen 포트 정의)
EXPOSE 4567
CMD    bundle exec ruby app.rb -o 0.0.0.0
```

‌

**OPTIONS**

- **FROM** : 베이스 이미지 지정. TAG로 지정하는 버전은 lastest보다는 구체적인 버전을 적어주는 것이 좋음.
- **MAINTAINER** : Dockerfile을 관리하는 사람의 정보를 입력. (이름, 메일주소 등)
- **COPY** : 파일이나 디렉토리를 이미지로 복사. 일밙거으로 소스를 복사할 때 사용함.
- **ADD** : COPY와 유사하나 src 입력에 파일 대신 URL입력 하거나 압축 파일을 지정 시 자동으로 압축을 해제하여 파일을 전송함.
- **RUN** : 명령어를 그대로 실행. 내부적으로 '/bin/sh -c' 를 실행 후 명령어가 실행 되는 로직임.
- **CMD** : 도커 컨테이너가 실행 되었을 때, 실행되는 명령어를 정의. 빌드할 때는 실행되지 않고 여러 개의 CMD가 있으면 가장 마지막 CMD만 실행된다. 한 번에 여러 프로그램을 실행 시킬 경우, run.sh나 supervisord, forege와 같은 프로그램을 사용해야 한다.
- **WORKDIR** : RUN, CMD, ADD, COPY 등이 이루어질 기본 디렉토리 설정. 각 명령어의 현재 디렉토리는 매번 초기화되기 때문에 같은 디렉토리에서의 작업을 위해 지정해줄 필요가 있음.
- **EXPOSE** : 도커 컨테이너가 실행되었을 때, 요청을 기다리는 포트 지정(Listen). 여러 포토를 지정 가능함.
- **VOLUME** : 컨테이너 외부 파일시스템을 마운트 할 때 사용. 기본으로 지정하는 편이 좋음.
- **ENV** : 컨테이너에서 사용할 환경변수 지정. 컨테이너 실행 시, -e 옵션을 사용하면 기존 값을 오버라이딩하여 사용함.

‌

만들어진 Dockerfile는 다음과 같이 빌드해볼 수 있다. 빌드한 이미지의 이름을 지어줄 -t 옵션을 사용하여 빌드하면 끗.

```
docker build [OPTIONS] PATH | URL | -
docker build -t app .
```

‌

## 5.3. Dockerfile 최적화‌

이전 예제의 Dockerfile에서 사실 우분투 베이스 이미지 말고 ruby 베이스 이미지를 사용하면 더 간편해진다.

```
# before
FROM ubuntu:16.04
MAINTAINER subicura@subicura.com
RUN apt-get -y update
RUN apt-get -y install ruby
RUN gem install bundler
# after
FROM ruby:2.3
MAINTAINER subicura@subicura.com
```

도커 이미지 빌드 시, Dockerfile의 명령어가 수정되거나 추가하는 파일이 변경 될 때 캐시가 깨져서 새로운 이미지를 재생성하기 때문에 시간을 많이 잡아먹게 됨. 그래서 캐시가 깨지는 것을 최대한 방지하여 빌드 시간을 줄여야 함.

‌

**예제 소스 캐시 최적화**

```
COPY . /usr/src/app    # <- 소스파일이 변경되면 캐시가 깨짐
WORKDIR /usr/src/app
RUN bundle install     # 패키지를 추가하지 않았는데 또 인스톨하게 됨 ㅠㅠ
# Gemfile을 미리 복사하여 ruby gem 패키지 설치를 무시
COPY Gemfile* /usr/src/app/ # Gemfile을 먼저 복사함
WORKDIR /usr/src/app
RUN bundle install          # 패키지 인스톨
COPY . /usr/src/app         # <- 소스가 바꼈을 때 캐시가 깨지는 시점 ^0^
```

‌

**명령어 최적화**

- -qq 옵션을 넣어주어 패키지 설치 시 나타나는 로그를 보이지 않게 해줌.
- --no-doc와 --no-ri 옵션으로 필요 없는 문서 생성을 방지.
- 명령어가 비슷한 것은 묶어서 레이어 생성을 줄이는 것이 좋음.

```
# 로그 최적화
# before
RUN apt-get -y update
# after
RUN apt-get -y -qq update
# 문서 작업 최적
# before
RUN bundle install
# after
RUN bundle install --no-rdoc --no-ri
# 레이어 생성 최적화
# before
RUN apt-get -y -qq update
RUN apt-get -y -qq install ruby
# after
RUN apt-get -y -qq update && \
    apt-get -y -qq install ruby
```

‌

# 도커 이미지 배포‌

도커는 빌드한 이미지를 git처럼 도커 레지스트리라고 불리는 이미지 저장소에 push & pull 하여 사용함. 도커 레지스트리는 오픈소스이며 무료로 설치 가능. 설치형이 아닌 것을 사용하려면 도커 허브를 사용하면 됨.

# 6. Docker Hub

도커 허브를 사용하려면 docker login 명령어로 로그인을 해야한다. (그 전에 미리 웹사이트에서 회원가입 필요) 도커 이미지 이름은 다음과 같이 구성된다.

```
[Registry URL]/[사용자 ID]/[이미지명]:[tag]
# tag 명령어는 기존에 만들어진 이미지의 이름을 지어줄 수 있음.
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
# Docker Hub에 Push 하기
docker push subicura/sinatra-app:1
```
‌

- **Registry URL** : 기본적으로 docker 허브를 바라보고 있어서 docker.io가 자동 입력. 특정 URL 지정도 가능.(Private 저장소 같이)
- **사용자 ID** : 지정하지 않으면 기본 ID인 library로 지정됨.
- **이미지 명** : 이전 예제들처럼 Name:Version 의 형식으로 지정하면 됨.

‌
도커의 배포 방식은 빌드한 이미지를 업로드하고 다운로드 받을 줄 만 알면 끗. 업데이트 과정도 복잡하지 않다. 그냥 기존에 잘 작동 중인 컨테이너를 끄고, 새 컨테이너로 덮어 쓴 후 다시 실행하면 된다. 물론 이 과정에서 잠깐 공백이 생기지만 이를 보완하는 무중단 배 방법들도 있다.

# References‌

- [https://subicura.com/2017/02/10/docker-guide-for-beginners-create-image-and-deploy.html](https://subicura.com/2017/02/10/docker-guide-for-beginners-create-image-and-deploy.html)
- [https://www.leafcats.com/153](https://www.leafcats.com/153)
- [https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html](https://subicura.com/2017/01/19/docker-guide-for-beginners-2.html)