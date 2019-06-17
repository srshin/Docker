## DOCKER 사용환경
* HOST OS : windows 10 home
* Docker : dockertoolbox
* volume 설정 사용: default는 c:/Users만 공유되므로 다른 폴더를 공유하기 위해서는 virtualbox에서 공유폴더 설정한뒤 docker-machine restart. 
* docker toolbox 사용시 192.168.99.100으로 사용해야 함. 
### docker 기본 명령
* https://github.com/srshin/blog/blob/master/docker/docker.md

## folder별 설명
### ubuntu
* ubuntu image를 pull하여 my_ubuntu 생성 
1. Dockerfile 
2. 이미지 생성  
$ docker build . -t my_ubuntu
3. 컨테이너 만들기    
* port 설정 / volume mount (volumn은 반드시 c:/Users아래 있어야 함.)    
```
$ docker run -dit --name my_ubuntu1  -p 3000:3000 -v /c/Users/src:/src my_ubuntu    
$ docker container ps    
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
1bec59f746c5        my_ubuntu           "/bin/bash"         16 seconds ago      Up 10 seconds       0.0.0.0:3000->3000/tcp   my_ubuntu1
```
*  접속및 mounting 확인   
```
$ docker exec -it my_ubuntu1 bash
root@216beee81140:/# cd /src
root@216beee81140:/src# ls
Dockerfile
```
### python
* python image를 가져와서 myapp생성 (flask framework)
1. 이미지 생성  
$ docker build . -t myapp  
2. 컨테이너 생성 및 실행  
$ docker run -it --name myapp2 -p 4000:80 myapp  
3. 브라우저에서 실행  
http://192.168.99.100:4000/  

### django
* python image로 장고 framework에서 돌아가는 이미지 생성 
1. 이미지 생성  
$ docker build . -t django  
2. 컨테이너 생성 및 실행  
$ docker run -it --name mydjango1 -p 8000:8000 django  
3. 브라우저에서 실행  
http://192.168.99.100:8000/  
4. docker-machin ip 확인
```
$ docker-machine ip  
192.168.99.100
```

### django_compose
* docker-compose를 이용하여 컨테이너 실행
1. 이미지 생성 & 컨테이너 실행  
$docker-compose up   
2. 브라우저에서 실행  
http://192.168.99.100:8000/


### django_mysql
* docker-compose를 이용하여 컨테이너 실행 (django만, mysql은 doker run으로 연결) 
* mysql image를 가져와서 local 의 django frame을 실행하여 mysql에 django db를 반영. 
* mysql 컨테이너가 살아있는 상태에서  python을 기반으로 장고 이미지를 만든 후 실행 
* image : mysql:5.7, django_mysql   
1. windows환경변수에 path에 추가   
C:\Program Files\MySQL\MySQL Server 8.0\bin   
2. mysql 이미지 pull  
$ docker pull mysql:5.7   
3. mysql 실행   
$ docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=djangodocker_db mysql:5.7  
화면이 더이상 움지이지 않으면 Ctrl + C로 멈춘다. 컨테이너가 떠있는 지 docker ps로 확인한다. 
4. mysql 접속/databse생성    
$ mysql -h127.0.0.1 -proot -uroot  
mysql> show DATABASES;  
5. local에서 docker database update  (local database service는 꺼야 함.)  
$ python manage.py migrate
6. local에서 동작 확인  
$ python manage.py runserver  
http://192.168.99.100:8000/   
7. mysql이 떠 있는 상태인지 먼저 확인
$ docker ps   
* TIP: mysql 에 data를 update하였으므로 종료된 상태인 경우에 start해야 함.   
* run하면 컨테이너가 새로 생성되므로 이전 저장한 데이타 사용 불가   
$ docker start [container name]   
8. 이미지 생성 & 컨테이너 실행    
$ docker-compose up  
9. 브라우저에서 실행  
http://192.168.99.100:8000/


### django_mysql_compose
* docker-compose를 이용하여 mysql과 django를 실행
1. 이미지 build   
$ docker-compose build
2. 컨테이너 실행  
$ docker-compose up
* database보다 django가 먼저 실행되는 경우 중간에 에러가 날 수 있음. 이때 docker-compose up을 한번 더 하면 됨.  
http://192.168.99.100:8000/  

### django_postgress_compose
* docker-compose를 이용하여 django와 postgress를 실행
1. 이미지 build   
$ docker-compose build
2. 컨테이너 실행   
$ docker-compose up
* database보다 django가 먼저 실행되는 경우 중간에 에러가 날 수 있음. 이때 docker-compose up을 한번 더 하면 됨.  
http://192.168.99.100:8000/  

### django_mysql_compose_volume
* docker-compose와 volume을 이용하여 mysql과 django 실행
* 데이타베이스와 코드가 모두 volume에 의하여 초기화되어있으므로 compose-up만 하면됨. 코드/database 수정시 실시간 반영됨. 
1. 프로젝트 생성 
```
$ docker-compose run web django-admin startproject composeexample .
```
2. db update
```
$ docker-compose run web python manage.py migrate
$ docker-compose run web python manage.py createsuperuser
```
3. docker-compose 수정  
이미지와 데이터베이스가 각각 volume으로 마운트되도록 수정 
```
  db:
    volumes:
      - /d/program/git/docker/django_mysql_compose_volume/mysql:/var/lib/mysql
  web:
    volumes:
      - /d/program/git/docker/django_mysql_compose_volume:/code
```
4. 이미지 build   
$ docker-compose build
5. 컨테이너 실행   
$ docker-compose up    
http://192.168.99.100:8000/    
6. 컨테이너 삭제  
$ docker-compose down 

### django_nginx
* docker-compose와 volume을 이용하여 django를 nginx server에 배포
1. polls 라는 app을 생성  
```
python manage.py startapp polls
```
2. setting.py 수정 - 배포용 
* DEBUG=False 
* ALLOWED_HOSTS 변경
* STATIC_ROOT 추가 
3. static 모으기, DB만들기  
```
$ python manage.py collectstatic
$ docker-compose run app python manage.py migrate
```
4. docker-compose 수정  
nginx, database, app 이미지와 데이터베이스가 각각 volume으로 마운트되도록 수정 
5. 이미지 build   
$ docker-compose build
6. 컨테이너 실행   
$ docker-compose up    
http://192.168.99.100:3000/polls    
7 . 컨테이너 삭제
$ docker-compose down 

### python_compose
* docker-compose를 이용하여 python 개발환경 구축
1. 이미지 build   
$ docker-compose build
2. interactive하게 container실행
```
$  docker-compose run app bash
root@6dbfed152eb0:/code# python --version
Python 3.6.8
```


