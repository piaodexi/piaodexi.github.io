---
title: git blog 페이지 오류 
author: dexi
date: 2022-12-20 00:00:01 +0800
categories: [Blog, gitBlog]
tags: [gitBlog]
math: true
mermaid: true
futrue: true
published: true
---

때때로 깃블로그에 글을 쓰고 깃에 push를 하고 나면 ..

빌드가 정상적으로 됬음에도 

메인 페이지에서   
--- layout: home # Index page ---   
위와 같이 뜨거나 404에러가 뜨는데   

이때 브라우저 캐시에 문제가 있을 수도 있으므로...

크롬같은 경우 시크릿 모드에서 열어보거나... 아니면 모바일로 유알엘 치고 들어가보거나 ...   
했을때 잘되면 잘되는거다...   

쓸데없이 삽질하지말자...   

----------------------------
10시간 후 

잘 안된다.. 

이번엔 이미지 url의 문제인듯 싶다..
문제가 시작된 커밋부분을 싹다 주석치고 푸시하니 잘된다..
