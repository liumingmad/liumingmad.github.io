title: docker+flask+mysql
author: ming
tags: []
categories:
  - docker
date: 2018-09-16 12:12:00
---
[阮一峰docker](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)

[阮一峰docker-compose](http://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html)

0. 每个container相当于一个虚拟机，有自己的ip地址

1. Dockerfile
```
FROM python:3
EXPOSE 5001
WORKDIR /what
COPY requirements.txt ./
RUN pip install -r requirements.txt
COPY . /what
CMD ["python", "main.py"]
```

2. docker-compose.yml 
```
version: '3'
services:
  db:
    image: mysql:5
    container_name: mydb
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=mingtest
    ports:
      - "3307:3306"
    volumes:
      - ~/Data/mysql:/var/lib/mysql
  web:
    build: .
    container_name: myweb
    ports:
      - "5001:5001"
    links:
      - db
```
3. requirements.txt
```
Flask
pymysql
flask-restful
```
4. main.py
```
  from flask import Flask
  from flask_restful import Api, Resource
  app = Flask(__name__)
  api = Api(app)
  class HelloWorld(Resource):
    def get(self):
      return {'hello': 'world'}

  api.add_resource(HelloWorld, '/')
  if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5001, debug=True)
```

5. docker-compose ps
```
Name        Command      State                Ports          
mydb    docker-entrypoint.sh mysqld   Up      0.0.0.0:3307->3306/tcp, 33060/tcp
myweb   python main.py                Up      0.0.0.0:5001->5001/tcp 
```

6. 连接msql
```
mysql -h127.0.0.1 -P3307 -uroot -proot
```

7. shell进正在运行的docker
```
docker container exec -it 43a38e5c1965 bash
```