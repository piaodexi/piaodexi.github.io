---
title: RemoteMonster lib 생각정리..  
author: dexi
date: 2023-01-17 00:00:01 +0800
categories: [Blog, Swift]
tags: [Swift]
math: true
mermaid: true
futrue: true
published: true
---
RemoteMonster라는 lib에서 방송 송출 및 시청을 위한 클래스인 RemonCast에 대해 짚고 넘어간다.   
 
RemonCast 라는 클래스에는 방송에 접속,생성,목록등을 가져오는 함수와 공통 클래스인 RemonClient의 remoteView 라는 UIView 객체가 있다.   
 
RemonCast 사용해 방송송출 기능을 구현할려면 시청자가 방송을 보기 위해서 실제 비디오가 그려지는 View를 정하고 연결해야 한다.   

일단 내 경우엔 실제 비디오가 그려지는 View는...viewWithTag로 가져오는 스크롤뷰의 두번째 화면이다..   
 
```
//방송을 보여줄 뷰(subView1)
let subView1 = view.viewWithTag(VIEW_TAG.SUB_VIEW_TAG_BASE + position - 1 )! as UIView
```

여기서 viewWithTag는 ...뷰에 접근하는 방법들중 하나인데 ..    
뷰 컨트롤러에서 뷰에 접근하려면 보통  **@IBOutlet 을 통해 참조를 얻어서 접근**을 하는데    
이 외에 또 다른 방법이 있는데, 바로  **View Tagging**  이다    
각각의 뷰에 태그 값을 부여해서, UIView 의 인스턴스 메소드인  **viewWithTag(_:)**  에 태그 값을 전달하여 사용한다.   

이렇게 가져온 스크롤뷰의 두번째 화면에 시청자에게 송출자가 보이도록 RemoteView을 연결해줘야한다.   
```
//실제 비디오가 그려지는 view(remoteView1)을 시청자에게 방송을 보여줄 뷰에 넣어준다.
subView1.addSubview(remoteView1)
//시청자에게 송출자가 보이도록 remoteView을 연결해준다. 
self.remonCast.remoteView = remoteView1
```






