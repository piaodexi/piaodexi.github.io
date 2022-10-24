---
title: 코코아팟 사용하기
author: dexi
date: 2022-10-24 00:00:01 +0800
categories: [Blog, Xcode]
tags: [Xcode]
math: true
mermaid: true
futrue: true
published: true
---

1. 터미널에서 cocoapods을 추가할 프로젝트의 루트디렉토리로 이동

2. sudo gem install cocoapods << 명령어로 cocoaPods 설치 

3. pod setup  명령어로 pod 설치 

4.pod init 명령어로 초기화 
(경우에 따라서 can't modify frozen String (FrozenError) 에러가 뜰 수 있음.
이때는 
project Document의 project format의 xcode 버전을 변경해줌. (버전 하나정도 내리면 될수도 있음. (ex 14 ->13, 13->12 ))

5.정상적으로 pod init이 되면 podfile이 생성됨. 해당파일을 열어서 추가할 라이브러리를 입력함. 

6. pod update 명령어로 업데이트를 해줌. 

7. 프로젝트에 xcworkspace로 프로젝트를 오픈함. 

