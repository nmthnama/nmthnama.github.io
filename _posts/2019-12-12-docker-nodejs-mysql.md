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
### 2) Nodejs
> Docker toolbox 에서 virutal box 에 자동으로 설치해주지만 로컬에서도 nodejs project 를 생성하고 테스트해봐야 하기 때문에 설치해줍니다.

## 2. NodeJS 프로젝트 생성
> 이미 생성한 NodeJS 프로젝트가 있다면 Skip 가능합니다. 
### 1) 빈폴더(tutorial) 생성
### 2) 빈폴더에 들어가서 shift+마우스 우클릭 후 여기에서 명령 창 실행 클릭
![](/assets/img/blog/2019-12-12-docker-nodejs-mysql/2019-12-12-17-46-48.png)
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
        // res.render('index', { title: 'Express', printtext: rows[0].printtext });
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
> --save 옵션은 packege.json 에 필요한 패키지를 등록해서 