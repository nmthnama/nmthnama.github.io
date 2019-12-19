---
layout: post
title:  "Deploy nodejs and mysql through docker toolbox on windows"
date:   2019-12-12
desc: "Using docker to install nodejs and mysql"
keywords: "docker,node,mysql,windows"
categories: [HTML]
tags: [docker]
icon: icon-html
---


## 1. Environment
### 1) Docker toolbox 
> Windows 10 home 버전 이하에서는 Docker Desktop을 설치하지 못하기 때문에 설치해야 대용으로 설치하는 프로그램
> ([document](https://docs.docker.com/toolbox/toolbox_install_windows/)), ([download](https://github.com/docker/toolbox/releases)))
### 2) NodeJS
> Docker toolbox 에서 virutal box 에 자동으로 설치해주지만 로컬에서도 nodejs project 를 생성하고 테스트해봐야 하기 때문에 설치해줍니다.
### 3) HeidiSQL
> DB 에 접근하기 위한 MySQL 프론트엔드. HeidiSQL 외에 MySQL Workbench 등을 사용해도 됨<br>
> [top 5 mysql gui tools for windows](https://www.eversql.com/top-5-mysql-gui-tools-for-windows/)
## 2. NodeJS 프로젝트 생성
> 이미 생성한 NodeJS 프로젝트가 있다면 Skip 가능합니다. 
### 1) 빈폴더(tutorial) 생성
### 2) 빈폴더에 들어가서 shift+마우스 우클릭 후 여기에서 명령 창 실행 클릭
![open cmd](/assets/img/blog/2019-12-12-docker-nodejs-mysql/2019-12-12-17-46-48.png)
### 3) nodejs 프로젝트 초기화
> npm init 명령어를 실행하면 nodejs 프로젝트 초기 설정 입력 대기를 하는데 모두 Enter 를 눌러서 진행
```console
C:\dockers\tutorial>npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (tutorial)
version: (1.0.0)
description:
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
About to write to C:\dockers\tutorial\package.json:

{
  "name": "tutorial",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}


Is this ok? (yes)
```
> 설정이 끝나면 package.json 파일이 생성된다.
### 4) Entry point (index.js) 코드 작성
> tutorial 폴더 경로에 index.js 를 생성하고 아래 코드를 copy&paste
```javascript
var mysql = require('mysql');

var connection = mysql.createConnection({
    host: 'app_db',
    port : '3306',
    user : 'root',
    password : 'testtest82',
    database : 'example01'
})

const port = 3000;


var express = require('express');
var app = express();

var server = app.listen(port, function () {
    console.log("Listening port : " + server.address().port);
});

app.get('/', function (req, res) {
    res.send('Hello Nodejs');
});


app.get("/dbcall", function (req, res) {
    connection.query('select * from test01', function(err, rows, fields){
        if (err) 
        {
            console.log(err);
            res.send("Query Error");
            return;
        }
        console.log(rows[0].printtext);
        res.send(rows[0].printtext);
        
      });
});
```
> require() 호출 인자를 보면 현재 코드에서 필요한 패키지가 무엇인지 확인 가능<br>
```javascript
var mysql = require('mysql');

var express = require('express');
```
> 위 2개의 라인을 보면 mysql 과 express 가 필요하다는 뜻

### 5) 필요 패키지 설치
```console
C:\dockers\tutorial>npm install --save mysql express
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN tutorial@1.0.0 No description
npm WARN tutorial@1.0.0 No repository field.

+ mysql@2.17.1
+ express@4.17.1
removed 16 packages and updated 2 packages in 8.183s
```
> --save 옵션은 packege.json 에 필요한 패키지를 등록
```json
{
  "name": "tutorial",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.17.1",
    "mysql": "^2.17.1"
  }
}
```
> package.json 의 dependencies 항목에 express 와 mysql 패키지가 save 된 것을 확인할 수 있다<br>
> ※ package.json 파일이 있는 폴더에서 npm install 명령어를 수행하면 dependencies 에 등록된 패키지가 자동으로 설치된다. 즉, 어떤 패키지가 필요한 지 일일히 찾아보지 않고도 npm install 명령어 하나로 모두 설치 가능

### 6) NodeJS 실행
> NodeJS 프로젝트 준비가 완료되었다. 정상적으로 설치 되었는지 확인해보자
> node index.js 명령어를 수행하여 node 를 실행한다.
```console
C:\dockers\tutorial>node index.js
Listening port : 3000
```
> 인터넷 브라우저를 이용하여 127.0.0.1:3000 에 접속
![request to node](/assets/img/blog/2019-12-12-docker-nodejs-mysql/2019-12-12-18-10-33.png)

## 3. NodeJS, MySQL 이미지 생성 및 컨테이너 실행
### 1) NodeJS 의 Dockerfile 생성
> Dockerfile 이란, 도커 이미지를 자동으로 빌드해주기 위한 명령어들을 기재한 파일이다.
> 각 명령어(ex : FROM, LABEL, RUN, . . .)에 대한 자세한 설명은 공식홈페이지를 참고 ([Link](https://docs.docker.com/engine/reference/builder/))
```Dockerfile
FROM node:carbon
LABEL maintainer "testtest@gmail.com"

#app 폴더 만들기 - NodeJS 어플리케이션 폴더
RUN mkdir -p /app
 
WORKDIR /app
 
# 도커 이미지로 파일 복사 (참고로 ./ 는 현재 경로를 뜻하는데, 이 Dockerfile 이 위치하고 있는 경로 기준)
# Syntax : ADD <src>... <dest>
ADD ./ /app
 
#패키지 파일 받기
RUN apt-get update
RUN npm install
 
# express.js 에서 참조하는 배포 환경 변수
ENV NODE_ENV=production
 
# 컨테이너가 실행될 때 ENTRYPOINT 의 파라미터로 전달
CMD node index.js
```
> 한 가지 주의 깊게 봐야할 부분은 CMD 부분인데, CMD 의 인자들은 ENTRYPOINT 의 파라미터로 전달되는 값이다.<br>
> 위 Dockerfile 은 ENTRYPOINT 를 지정하지 않았으므로 CMD 자체가 명령어가 될 수 있다.<br>
> 그러므로 CMD node index.js 는 아래와 같이 변형할 수 있다.<br>
```Dockerfile
#CMD node index.js
ENTRYPOINT ["node"]
CMD ["index.js"]
```
> ENTRYPOINT 와 CMD 의 combination 과 관련된 정보는 공식 가이드에 자세히 설명 되어 있으니 참고([Link](https://docs.docker.com/engine/reference/builder/))
### 2) MySQL 의 Dockerfile 생성
> mysql 폴더를 생성하고 그 아래에 Dockerfile 를 생성

### 3) compose file 생성
> compose 파일이란, 각 컨테이너가 실행될 때 수행되는 파라미터를 정의한 파일이다.<br>
> 즉, 위에서 생성한 dockerfile 들을 참조하여 컨테이너를 실행할 때마다 매번 옵션을 지정해주지 않고, 파일에 미리 옵션을 정의해서 간편하게 실행시키는 것을 가능하게 하는 파일이다.<br>
> Nodejs 와 mysql 의 dockerfile 을 참조하여 컨테이너를 생성하고 실행하게 해주는 compose 파일은 아래와 같다

```yml
# docker-compose의 버전을 명시. 버전별로 명령어등의 약간의 차이가 있다.
version: "2"

services:
    app:
        container_name: app
        build: .

        # 환경변수를 지정
        environment:
            NODE_ENV: localhost
        ports:
            - 3000:3000
        # 다른 컨테이너와 연결
            # Syntax : { SERVICE:ALIAS }
            # SERVICE 는 다른 컨테이너, ALIAS 는 내 컨테이너에서 SERVICE 를 이용할 때 사용할 별칭
        links:
            - db:app_db
        
    db:
        container_name: mysql
        build: ./mysql
        ports:
            - 3306:3306
        
        # 데이터 볼륨 지정 : 
            # 컨테이너는 시작할 때마다 IMAGE 상태로 기동되기 때문에 매번 데이터가 초기화 된다. 
            # 즉, 컨테이너가 정지되면, 컨테이너 안에서 작업했던 내용이 모두 사라진다.
            # Syntax : { HOST : CONTAINER }
        volumes: 
            - /data/mysql:/var/lib/mysql
            
```

## 4. DB 에 테이블 생성하고 nodejs 를 통해 접근하기
