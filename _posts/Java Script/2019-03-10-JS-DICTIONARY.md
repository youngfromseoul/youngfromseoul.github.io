---
title: "[Java Script] 객체란?" 
excerpt: "자바스크립트의 객체"
categories: 
  - Java Script
last_modified_at: 2019-03-10T22:39:00+09:00
tags: 
    - Java Script
    - Programing
    - Develope
author_profile: true
read_time: true
toc_label: "Dictionary" 
toc_icon: "cog" 
toc: true
toc_sticky: true
---

##Dictionary?
  배열은 아이템에 대한 식별자로 숫자를 사용했다. 데이터가 추가되면 배열 전체에서 중복되지 않는 인덱스가 자동으로 만들어져서 추가된 데이터에 대한 식별자가 된다. 이 인덱스를 이용해서 데이터를 가져오게 되는 것이다.  
  만약 인덱스로 문자를 사용하고 싶다면 객체(dictionary)를 사용해야 한다. 다른 언어에서는 연관배열(associative array) 또는 맵( map), 딕셔너리(Dictionary)라는 데이터 타입이 객체에 해당한다.  


```html
<!DOCTYPE html>
<html>
    <head>
        <title>

        </title>
    </head>
    <body>
        <script type="text/javascript">

        //객체 생성
        var grades = {"hyoyoungkim":1234,"songyipark":4332,"guccipark":3543};
        //앞의 값이 key, 뒤의 값이 valu능

        //아래와 같은 방법으로도 생성 가능
        var grades = {};
        grades["hyoyungkim"] = 1234;
        grades["songyipark"] = 4332;
        grades["guccipark"] = 3543;

        //객체 불러오기
        alert(grades.hyoyoungkim);
        grades["hyoyoungkim"]

        </script>
    </body>
</html>
```
